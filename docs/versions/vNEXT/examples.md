# Examples

This guide provides detailed walkthroughs of the examples included with bevy-fusabi and demonstrates common integration patterns.

## Included Examples

The repository includes the following examples:

1. **load_script** - Basic script loading and asset management
2. **execute_script** - Script execution with RunnerPlugin

## Example 1: Basic Script Loading

Location: `examples/load_script.rs`

### Overview

This example demonstrates:
- Loading Fusabi scripts as Bevy assets
- Monitoring asset loading events
- Accessing loaded script data
- Deserializing bytecode

### Complete Code

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

fn main() {
    App::new()
        .add_plugins(MinimalPlugins)
        .add_plugins(AssetPlugin::default())
        .add_plugins(FusabiPlugin)
        .add_systems(Startup, setup)
        .add_systems(Update, check_asset)
        .run();
}

#[derive(Resource)]
struct ScriptHandle(Handle<FusabiScript>);

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    let handle = asset_server.load("hello.fsx");
    commands.insert_resource(ScriptHandle(handle));
    println!("Loading script...");
}

fn check_asset(
    mut events: EventReader<AssetEvent<FusabiScript>>,
    scripts: Res<Assets<FusabiScript>>,
) {
    for event in events.read() {
        match event {
            AssetEvent::LoadedWithDependencies { id } => {
                let script = scripts.get(*id).unwrap();
                println!("Script loaded successfully: {}", script.name);
                println!("Bytecode size: {} bytes", script.bytecode.len());

                // Verify we can deserialize it
                match script.to_chunk() {
                    Ok(chunk) => println!(
                        "Deserialized chunk successfully. Opcode count: {}",
                        chunk.instructions.len()
                    ),
                    Err(e) => println!("Failed to deserialize chunk: {}", e),
                }
            }
            _ => {}
        }
    }
}
```

### Script File

Create `assets/hello.fsx`:

```fusabi
fn greet(name) {
    print("Hello, " + name + "!");
}

greet("World");
```

### Running

```bash
cargo run --example load_script
```

### Expected Output

```
Loading script...
Script loaded successfully: hello
Bytecode size: 42 bytes
Deserialized chunk successfully. Opcode count: 15
```

### Key Concepts

1. **Asset Loading**: Scripts are loaded through Bevy's asset server
2. **Event Handling**: Use `AssetEvent` to monitor loading progress
3. **Bytecode Access**: Access compiled bytecode via `script.bytecode`
4. **Deserialization**: Convert bytecode to executable chunks

## Example 2: Script Execution

Location: `examples/execute_script.rs`

### Overview

This example demonstrates:
- Automatic script execution via RunnerPlugin
- Using the RunScript component
- Script lifecycle management

### Complete Code

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

fn main() {
    App::new()
        .add_plugins(MinimalPlugins)
        .add_plugins(AssetPlugin::default())
        .add_plugins(FusabiPlugin)
        .add_plugins(RunnerPlugin)
        .add_systems(Startup, setup)
        .run();
}

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    let handle = asset_server.load("hello.fsx");

    // Spawn an entity that wants to run this script
    commands.spawn(RunScript {
        handle,
        executed: false,
    });
}
```

### Running

```bash
cargo run --example execute_script
```

### Expected Output

```
Executing script: hello
Hello, World!
Script execution result: Nil
```

### Key Concepts

1. **RunnerPlugin**: Automatically executes scripts
2. **RunScript Component**: Marks entities for script execution
3. **Execution State**: `executed` flag prevents re-execution
4. **Return Values**: Scripts can return values to Rust

## Common Patterns

### Pattern 1: Script Library

Load multiple scripts at startup:

```rust
#[derive(Resource)]
struct ScriptLibrary {
    init: Handle<FusabiScript>,
    gameplay: Handle<FusabiScript>,
    ui: Handle<FusabiScript>,
}

fn setup_scripts(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.insert_resource(ScriptLibrary {
        init: asset_server.load("scripts/init.fsx"),
        gameplay: asset_server.load("scripts/gameplay.fsx"),
        ui: asset_server.load("scripts/ui.fsx"),
    });
}

fn execute_init_script(
    mut commands: Commands,
    library: Res<ScriptLibrary>,
    scripts: Res<Assets<FusabiScript>>,
) {
    if scripts.get(&library.init).is_some() {
        commands.spawn(RunScript {
            handle: library.init.clone(),
            executed: false,
        });
    }
}
```

### Pattern 2: Conditional Execution

Execute scripts based on game state:

```rust
#[derive(Resource)]
enum GameState {
    Menu,
    Playing,
    Paused,
}

fn conditional_execution(
    mut commands: Commands,
    state: Res<GameState>,
    asset_server: Res<AssetServer>,
) {
    match *state {
        GameState::Menu => {
            commands.spawn(RunScript {
                handle: asset_server.load("scripts/menu.fsx"),
                executed: false,
            });
        }
        GameState::Playing => {
            commands.spawn(RunScript {
                handle: asset_server.load("scripts/gameplay.fsx"),
                executed: false,
            });
        }
        _ => {}
    }
}
```

### Pattern 3: Error Handling

Handle script errors gracefully:

```rust
fn execute_with_error_handling(
    mut query: Query<&mut RunScript>,
    scripts: Res<Assets<FusabiScript>>,
) {
    for mut runner in query.iter_mut() {
        if runner.executed {
            continue;
        }

        if let Some(script) = scripts.get(&runner.handle) {
            match script.to_chunk() {
                Ok(chunk) => {
                    let mut vm = Vm::new();
                    match vm.execute(chunk) {
                        Ok(value) => {
                            info!("Script completed: {:?}", value);
                            runner.executed = true;
                        }
                        Err(e) => {
                            error!("Script error: {:?}", e);
                            // Optionally mark as executed to prevent retry
                            runner.executed = true;
                        }
                    }
                }
                Err(e) => {
                    error!("Failed to deserialize script: {}", e);
                    runner.executed = true;
                }
            }
        }
    }
}
```

### Pattern 4: Hot Reload Support

Enable hot reload during development:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(AssetPlugin {
            watch_for_changes_override: Some(true),
            ..default()
        }))
        .add_plugins(FusabiPlugin)
        .add_plugins(RunnerPlugin)
        .add_systems(Startup, setup)
        .add_systems(Update, handle_reload)
        .run();
}

#[derive(Component)]
struct HotReloadable;

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn((
        RunScript {
            handle: asset_server.load("scripts/game.fsx"),
            executed: false,
        },
        HotReloadable,
    ));
}

fn handle_reload(
    mut events: EventReader<AssetEvent<FusabiScript>>,
    mut query: Query<&mut RunScript, With<HotReloadable>>,
) {
    for event in events.read() {
        if let AssetEvent::Modified { id } = event {
            for mut runner in query.iter_mut() {
                if runner.handle.id() == *id {
                    runner.executed = false;
                    println!("Script reloaded, will re-execute");
                }
            }
        }
    }
}
```

### Pattern 5: Multiple VM Instances

Use separate VMs for isolation:

```rust
#[derive(Component)]
struct ScriptedEntity {
    script: Handle<FusabiScript>,
    vm: Vm,
    executed: bool,
}

fn execute_isolated_scripts(
    mut query: Query<&mut ScriptedEntity>,
    scripts: Res<Assets<FusabiScript>>,
) {
    for mut entity in query.iter_mut() {
        if entity.executed {
            continue;
        }

        if let Some(script) = scripts.get(&entity.script) {
            if let Ok(chunk) = script.to_chunk() {
                match entity.vm.execute(chunk) {
                    Ok(value) => {
                        println!("Entity script result: {:?}", value);
                        entity.executed = true;
                    }
                    Err(e) => {
                        error!("Entity script error: {:?}", e);
                    }
                }
            }
        }
    }
}
```

## Advanced Examples

### Example: Plugin Runtime Integration (Future)

Planned example for plugin runtime integration:

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;
use fusabi_plugin_runtime::PluginRuntime;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(FusabiPlugin)
        .add_systems(Startup, setup_plugin_runtime)
        .add_systems(Update, update_plugins)
        .run();
}

fn setup_plugin_runtime(mut commands: Commands) {
    let mut runtime = PluginRuntime::new();

    // Load plugin with hot reload
    runtime.load_plugin("assets/plugins/gameplay.fsx", true)
        .expect("Failed to load plugin");

    commands.insert_resource(runtime);
}

fn update_plugins(
    mut runtime: ResMut<PluginRuntime>,
    time: Res<Time>,
) {
    // Call update hook on all plugins
    runtime.update(time.delta_seconds());

    // Check for hot reload
    if let Some(reloaded) = runtime.check_reloads() {
        info!("Plugins reloaded: {:?}", reloaded);
    }
}
```

### Example: Networked Scripts (Future)

Planned example for networked script synchronization:

```rust
// Server loads authoritative scripts
fn server_setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn((
        RunScript {
            handle: asset_server.load("scripts/server_logic.fsx"),
            executed: false,
        },
        ServerScript,
    ));
}

// Client receives script results
fn client_update(
    messages: Res<NetworkMessages>,
    mut query: Query<&mut GameState>,
) {
    for msg in messages.iter() {
        if let NetworkMessage::ScriptResult(result) = msg {
            // Update client state from server script
        }
    }
}
```

## Testing Examples

Run all examples:

```bash
# List all examples
cargo run --example

# Run specific example
cargo run --example load_script

# Run with logging
RUST_LOG=debug cargo run --example execute_script
```

## Creating Your Own Examples

### Template

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(FusabiPlugin)
        // Add your systems
        .run();
}

// Your systems here
```

### Best Practices

1. **Keep it simple**: Focus on one concept per example
2. **Add comments**: Explain what each part does
3. **Error handling**: Show how to handle errors
4. **Documentation**: Add inline docs for complex parts
5. **Test data**: Include sample scripts in assets/

## Troubleshooting Examples

### Example Won't Compile

1. Check Rust version: `rustc --version` (needs 1.75+)
2. Update dependencies: `cargo update`
3. Clean build: `cargo clean && cargo build`

### Script Not Found

1. Verify assets directory exists
2. Check script path matches load call
3. Ensure working directory is project root

### No Output

1. Enable logging: `RUST_LOG=info cargo run`
2. Check script syntax is valid
3. Verify plugins are added in correct order

## Next Steps

- Review [Integration](integration.md) for API details
- Check [Hot Reload](hot-reload.md) for development workflow
- See [Compatibility](compatibility.md) for version requirements
- Read [Getting Started](getting-started.md) for setup instructions
