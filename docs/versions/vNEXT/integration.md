# Bevy Integration

This guide covers the details of integrating Fusabi scripts with Bevy's Entity Component System and asset pipeline.

## Plugin Architecture

### FusabiPlugin

The core plugin that registers the asset type and loader:

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

App::new()
    .add_plugins(DefaultPlugins)
    .add_plugins(FusabiPlugin)  // Registers FusabiScript asset and loader
    .run();
```

The `FusabiPlugin` provides:
- `FusabiScript` asset type registration
- `FusabiLoader` for loading `.fsx` and `.fzb` files
- Automatic compilation of source files
- Deserialization of bytecode files

### RunnerPlugin

Optional plugin for automatic script execution:

```rust
App::new()
    .add_plugins(DefaultPlugins)
    .add_plugins(FusabiPlugin)
    .add_plugins(RunnerPlugin)  // Enables automatic execution
    .run();
```

## Asset Loading System

### FusabiScript Asset

The `FusabiScript` asset represents a compiled script:

```rust
pub struct FusabiScript {
    pub name: String,
    pub bytecode: Vec<u8>,
}
```

Key features:
- Thread-safe (Send + Sync)
- Stores serialized bytecode
- Compatible with Bevy's asset system
- Lazy deserialization for performance

### Loading Process

1. **Load Request**: Asset server receives load request
2. **File Read**: Loader reads the file from disk
3. **Compilation** (for .fsx): Source is compiled to bytecode
4. **Serialization**: Chunk is serialized to Vec<u8>
5. **Asset Creation**: FusabiScript asset is created
6. **Registration**: Asset is registered with Bevy

### Custom Loading

For advanced use cases:

```rust
#[derive(Resource)]
struct ScriptLibrary {
    gameplay: Handle<FusabiScript>,
    ui: Handle<FusabiScript>,
    ai: Handle<FusabiScript>,
}

fn load_scripts(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.insert_resource(ScriptLibrary {
        gameplay: asset_server.load("scripts/gameplay.fsx"),
        ui: asset_server.load("scripts/ui.fsx"),
        ai: asset_server.load("scripts/ai.fsx"),
    });
}
```

## ECS Integration

### RunScript Component

The standard component for script execution:

```rust
#[derive(Component)]
pub struct RunScript {
    pub handle: Handle<FusabiScript>,
    pub executed: bool,
}
```

Example usage:

```rust
fn spawn_scripted_entity(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
) {
    commands.spawn((
        Name::new("Scripted Entity"),
        RunScript {
            handle: asset_server.load("scripts/behavior.fsx"),
            executed: false,
        },
    ));
}
```

### Script Execution System

The runner system executes scripts:

```rust
fn run_scripts(
    mut query: Query<&mut RunScript>,
    scripts: Res<Assets<FusabiScript>>,
) {
    for mut runner in query.iter_mut() {
        if runner.executed {
            continue;
        }

        if let Some(script) = scripts.get(&runner.handle) {
            // Execute script
            match script.to_chunk() {
                Ok(chunk) => {
                    let mut vm = Vm::new();
                    match vm.execute(chunk) {
                        Ok(value) => {
                            println!("Result: {:?}", value);
                            runner.executed = true;
                        }
                        Err(e) => println!("Error: {:?}", e),
                    }
                }
                Err(e) => println!("Failed to load chunk: {}", e),
            }
        }
    }
}
```

### Custom Execution Patterns

#### Periodic Execution

Run scripts on a timer:

```rust
#[derive(Component)]
struct PeriodicScript {
    handle: Handle<FusabiScript>,
    timer: Timer,
}

fn run_periodic_scripts(
    mut query: Query<&mut PeriodicScript>,
    scripts: Res<Assets<FusabiScript>>,
    time: Res<Time>,
) {
    for mut periodic in query.iter_mut() {
        periodic.timer.tick(time.delta());

        if periodic.timer.just_finished() {
            if let Some(script) = scripts.get(&periodic.handle) {
                // Execute script
                execute_script(script);
            }
        }
    }
}
```

#### Event-Driven Execution

Execute scripts in response to events:

```rust
#[derive(Event)]
struct TriggerScript(Handle<FusabiScript>);

fn on_trigger_script(
    mut events: EventReader<TriggerScript>,
    scripts: Res<Assets<FusabiScript>>,
) {
    for trigger in events.read() {
        if let Some(script) = scripts.get(&trigger.0) {
            execute_script(script);
        }
    }
}
```

## Plugin Runtime Integration (Upcoming)

Future versions will integrate with `fusabi-plugin-runtime` for advanced features:

### Features

- **Dynamic Plugin Loading**: Load scripts as plugins at runtime
- **Hot Reload**: Advanced hot-reload with state preservation
- **Capability System**: Fine-grained permission control
- **Lifecycle Management**: Init, update, shutdown hooks
- **Metrics**: Built-in performance tracking

### Planned API

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

fn setup_plugin_runtime(mut commands: Commands) {
    commands.spawn(PluginRuntimeConfig {
        watch_paths: vec!["assets/scripts"],
        hot_reload: true,
        capabilities: Capabilities::default(),
    });
}

#[derive(Component)]
struct ScriptedBehavior {
    runtime: PluginRuntime,
    plugin_id: PluginId,
}

fn update_scripted_behaviors(
    mut query: Query<&mut ScriptedBehavior>,
    time: Res<Time>,
) {
    for mut behavior in query.iter_mut() {
        // Call update hook on plugin
        behavior.runtime.call_hook(
            &behavior.plugin_id,
            "update",
            &[time.delta_seconds()],
        );
    }
}
```

## Error Handling

### Compilation Errors

Handle compilation errors gracefully:

```rust
fn check_script_errors(
    mut events: EventReader<AssetEvent<FusabiScript>>,
) {
    for event in events.read() {
        if let AssetEvent::Failed { id, error } = event {
            // Log detailed error information
            error!("Script compilation failed: {:?}", error);

            // Optionally notify the user
            // show_error_ui(&error);
        }
    }
}
```

### Runtime Errors

Handle execution errors:

```rust
fn execute_with_error_handling(script: &FusabiScript) {
    match script.to_chunk() {
        Ok(chunk) => {
            let mut vm = Vm::new();
            match vm.execute(chunk) {
                Ok(value) => println!("Success: {:?}", value),
                Err(e) => error!("Runtime error: {:?}", e),
            }
        }
        Err(e) => error!("Deserialization error: {}", e),
    }
}
```

## Performance Considerations

### Bytecode Caching

Pre-compile scripts to `.fzb` for faster loading:

```bash
fusabi compile script.fsx -o script.fzb
```

Then load the bytecode directly:

```rust
let handle = asset_server.load("scripts/script.fzb");
```

### VM Reuse

Reuse VM instances when possible:

```rust
#[derive(Resource)]
struct ScriptVm(Vm);

fn setup_vm(mut commands: Commands) {
    commands.insert_resource(ScriptVm(Vm::new()));
}

fn execute_with_shared_vm(
    script: &FusabiScript,
    mut vm: ResMut<ScriptVm>,
) {
    if let Ok(chunk) = script.to_chunk() {
        let _ = vm.0.execute(chunk);
    }
}
```

### Lazy Loading

Load scripts only when needed:

```rust
fn load_on_demand(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    trigger: Res<LoadScriptTrigger>,
) {
    if trigger.should_load {
        let handle = asset_server.load(&trigger.path);
        commands.spawn(RunScript {
            handle,
            executed: false,
        });
    }
}
```

## Best Practices

1. **Separate Concerns**: Use different scripts for different systems
2. **Error Handling**: Always handle compilation and runtime errors
3. **Resource Management**: Clean up unused script handles
4. **Development Workflow**: Use hot reload during development
5. **Production Builds**: Use pre-compiled `.fzb` files in production
6. **Testing**: Test scripts independently before integration
7. **Documentation**: Document script interfaces and expectations

## Next Steps

- Learn about [Hot Reload](hot-reload.md) workflows
- Explore [Examples](examples.md) for integration patterns
- Check [Compatibility](compatibility.md) for version requirements
