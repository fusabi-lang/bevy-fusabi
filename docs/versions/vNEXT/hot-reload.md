# Hot Reload Guide

This guide covers hot reload capabilities in Bevy Fusabi, enabling rapid iteration on your scripts without restarting your application.

## Overview

Hot reload allows you to modify scripts while your application is running and see the changes immediately. This is particularly useful during development for:

- Testing gameplay tweaks
- Adjusting AI behavior
- Fine-tuning UI scripts
- Rapid prototyping

## Basic Hot Reload

### Setup

Hot reload is enabled by default when using the Bevy asset server with file watching:

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(AssetPlugin {
            // File watching is enabled by default in debug builds
            watch_for_changes_override: Some(true),
            ..default()
        }))
        .add_plugins(FusabiPlugin)
        .add_plugins(RunnerPlugin)
        .add_systems(Startup, setup)
        .add_systems(Update, handle_script_reload)
        .run();
}
```

### Handling Reload Events

React to script changes:

```rust
fn handle_script_reload(
    mut events: EventReader<AssetEvent<FusabiScript>>,
    mut query: Query<&mut RunScript>,
) {
    for event in events.read() {
        match event {
            AssetEvent::Modified { id } => {
                println!("Script modified: {:?}", id);

                // Reset execution flag to allow re-run
                for mut runner in query.iter_mut() {
                    if runner.handle.id() == *id {
                        runner.executed = false;
                        println!("Marked script for re-execution");
                    }
                }
            }
            _ => {}
        }
    }
}
```

## Development Workflow

### 1. Initial Setup

Create your script file:

```fusabi
// assets/scripts/gameplay.fsx
fn on_player_jump() {
    print("Player jumped!");
}

on_player_jump();
```

Load it in your app:

```rust
fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn(RunScript {
        handle: asset_server.load("scripts/gameplay.fsx"),
        executed: false,
    });
}
```

### 2. Iterate

While your app is running:

1. Edit the script file
2. Save the changes
3. The asset server detects the change
4. Script is recompiled automatically
5. Modified event is fired
6. Your handler resets execution state
7. Script runs with new code

### 3. Test

See your changes immediately without restarting the application!

## Advanced Hot Reload Patterns

### State Preservation

Preserve state across reloads:

```rust
#[derive(Resource)]
struct ScriptState {
    data: HashMap<String, Value>,
}

fn save_state_before_reload(
    events: EventReader<AssetEvent<FusabiScript>>,
    mut state: ResMut<ScriptState>,
) {
    // Save important state before reload
}

fn restore_state_after_reload(
    events: EventReader<AssetEvent<FusabiScript>>,
    state: Res<ScriptState>,
) {
    // Restore state after reload
}
```

### Conditional Reload

Only reload specific scripts:

```rust
#[derive(Component)]
struct ReloadableScript {
    hot_reload: bool,
}

fn selective_reload(
    mut events: EventReader<AssetEvent<FusabiScript>>,
    mut query: Query<(&mut RunScript, &ReloadableScript)>,
) {
    for event in events.read() {
        if let AssetEvent::Modified { id } = event {
            for (mut runner, reloadable) in query.iter_mut() {
                if reloadable.hot_reload && runner.handle.id() == *id {
                    runner.executed = false;
                }
            }
        }
    }
}
```

### Reload with Validation

Validate scripts before applying changes:

```rust
fn validated_reload(
    mut events: EventReader<AssetEvent<FusabiScript>>,
    scripts: Res<Assets<FusabiScript>>,
    mut query: Query<&mut RunScript>,
) {
    for event in events.read() {
        if let AssetEvent::Modified { id } = event {
            // Validate the script first
            if let Some(script) = scripts.get(*id) {
                match script.to_chunk() {
                    Ok(_) => {
                        println!("✓ Script validated successfully");
                        // Reset for re-execution
                        for mut runner in query.iter_mut() {
                            if runner.handle.id() == *id {
                                runner.executed = false;
                            }
                        }
                    }
                    Err(e) => {
                        error!("✗ Script validation failed: {}", e);
                        // Don't reload invalid scripts
                    }
                }
            }
        }
    }
}
```

## Production Considerations

### Disabling Hot Reload

Disable file watching in production:

```rust
App::new()
    .add_plugins(DefaultPlugins.set(AssetPlugin {
        watch_for_changes_override: Some(cfg!(debug_assertions)),
        ..default()
    }))
    .add_plugins(FusabiPlugin)
    .run();
```

### Pre-compiled Assets

Use bytecode files in production for better performance:

```bash
# Compile all scripts to bytecode
fusabi compile assets/scripts/**/*.fsx
```

Then package `.fzb` files instead of `.fsx` files.

## Plugin Runtime Hot Reload (Upcoming)

Future integration with `fusabi-plugin-runtime` will provide advanced hot-reload features:

### Features

- **State Preservation**: Automatic state transfer between reloads
- **Lifecycle Hooks**: Clean shutdown and restart of plugins
- **Incremental Updates**: Only reload changed functions
- **Error Recovery**: Rollback to last working version on error

### Planned API

```rust
#[derive(Component)]
struct HotReloadablePlugin {
    runtime: PluginRuntime,
    watcher: FileWatcher,
}

fn setup_hot_reload(mut commands: Commands) {
    let mut runtime = PluginRuntime::new();
    let watcher = FileWatcher::new("assets/scripts")
        .on_change(|path| {
            println!("Reloading: {:?}", path);
        });

    commands.spawn(HotReloadablePlugin {
        runtime,
        watcher,
    });
}

fn handle_hot_reload(
    mut query: Query<&mut HotReloadablePlugin>,
) {
    for mut plugin in query.iter_mut() {
        if let Some(changes) = plugin.watcher.poll_changes() {
            for change in changes {
                // Save plugin state
                let state = plugin.runtime.save_state(&change.plugin_id);

                // Reload plugin
                plugin.runtime.reload_plugin(&change.plugin_id)?;

                // Restore state
                plugin.runtime.restore_state(&change.plugin_id, state);
            }
        }
    }
}
```

## Troubleshooting

### Changes Not Detected

1. **Check file watcher is enabled**:
   ```rust
   watch_for_changes_override: Some(true)
   ```

2. **Verify file permissions**: Ensure the file is writable
3. **Check file path**: Verify the asset path is correct
4. **IDE save settings**: Some IDEs use safe write, which may not trigger file events

### Compilation Errors After Reload

1. **Check syntax**: Validate your script syntax
2. **Review error messages**: Asset events include error details
3. **Test separately**: Compile the script standalone first
4. **Rollback changes**: Revert to last working version

### State Lost on Reload

1. **Implement state preservation**: See state preservation pattern above
2. **Use external storage**: Store critical state in Bevy resources
3. **Design for reload**: Structure scripts to be reload-friendly

## Best Practices

1. **Small Iterations**: Make small changes and test frequently
2. **Validation**: Always validate scripts before applying changes
3. **Error Handling**: Handle compilation and runtime errors gracefully
4. **State Management**: Plan for state preservation in stateful scripts
5. **Testing**: Test hot reload workflow during development
6. **Production**: Disable hot reload in production builds
7. **Documentation**: Document which scripts support hot reload

## Performance Tips

1. **Selective Watching**: Only watch scripts that need hot reload
2. **Debouncing**: Add debounce logic for rapid successive saves
3. **Lazy Recompilation**: Only recompile when actually needed
4. **Background Compilation**: Compile in background thread if possible

## Example: Complete Hot Reload Setup

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(AssetPlugin {
            watch_for_changes_override: Some(true),
            ..default()
        }))
        .add_plugins(FusabiPlugin)
        .add_plugins(RunnerPlugin)
        .init_resource::<ScriptState>()
        .add_systems(Startup, setup)
        .add_systems(Update, (
            handle_reload,
            log_script_changes,
        ))
        .run();
}

#[derive(Resource, Default)]
struct ScriptState {
    reload_count: u32,
}

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn((
        Name::new("Gameplay Script"),
        RunScript {
            handle: asset_server.load("scripts/gameplay.fsx"),
            executed: false,
        },
        HotReloadable,
    ));
}

#[derive(Component)]
struct HotReloadable;

fn handle_reload(
    mut events: EventReader<AssetEvent<FusabiScript>>,
    mut query: Query<&mut RunScript, With<HotReloadable>>,
    mut state: ResMut<ScriptState>,
) {
    for event in events.read() {
        if let AssetEvent::Modified { id } = event {
            state.reload_count += 1;
            println!("Reload #{}", state.reload_count);

            for mut runner in query.iter_mut() {
                if runner.handle.id() == *id {
                    runner.executed = false;
                }
            }
        }
    }
}

fn log_script_changes(
    mut events: EventReader<AssetEvent<FusabiScript>>,
) {
    for event in events.read() {
        match event {
            AssetEvent::Added { id } => println!("Script added: {:?}", id),
            AssetEvent::Modified { id } => println!("Script modified: {:?}", id),
            AssetEvent::Removed { id } => println!("Script removed: {:?}", id),
            _ => {}
        }
    }
}
```

## Next Steps

- Explore [Examples](examples.md) for more hot reload patterns
- Review [Integration](integration.md) for ECS patterns
- Check [Compatibility](compatibility.md) for version-specific features
