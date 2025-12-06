# Getting Started with Bevy Fusabi

This guide will walk you through setting up Bevy Fusabi in your project and creating your first Fusabi script.

## Installation

### Prerequisites

- Rust 1.75 or later
- Basic familiarity with Bevy
- Basic understanding of scripting languages

### Add Dependencies

Add bevy-fusabi to your `Cargo.toml`:

```toml
[dependencies]
bevy = "0.15"
bevy-fusabi = "0.1.4"
```

## Your First Script

### 1. Create a Script File

Create an `assets/scripts` directory in your project and add a file called `hello.fsx`:

```fusabi
// hello.fsx
fn greet(name) {
    print("Hello, " + name + "!");
}

greet("Bevy");
```

### 2. Set Up Your Bevy App

Create a basic Bevy application with the Fusabi plugin:

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(FusabiPlugin)
        .add_plugins(RunnerPlugin)
        .add_systems(Startup, setup)
        .run();
}

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    // Load and execute the script
    let handle = asset_server.load("scripts/hello.fsx");

    commands.spawn(RunScript {
        handle,
        executed: false,
    });
}
```

### 3. Run Your Application

```bash
cargo run
```

You should see "Hello, Bevy!" printed to the console when the script executes.

## Loading Scripts

### Source Files (.fsx)

Source files are compiled on load:

```rust
fn load_source_script(mut commands: Commands, asset_server: Res<AssetServer>) {
    let handle = asset_server.load("scripts/my_script.fsx");
    commands.insert_resource(MyScriptHandle(handle));
}
```

### Bytecode Files (.fzb)

Pre-compiled bytecode files load faster:

```rust
fn load_bytecode_script(mut commands: Commands, asset_server: Res<AssetServer>) {
    let handle = asset_server.load("scripts/my_script.fzb");
    commands.insert_resource(MyScriptHandle(handle));
}
```

## Accessing Scripts

### Checking Load Status

Monitor when scripts are loaded:

```rust
fn check_script_loaded(
    mut events: EventReader<AssetEvent<FusabiScript>>,
    scripts: Res<Assets<FusabiScript>>,
) {
    for event in events.read() {
        match event {
            AssetEvent::LoadedWithDependencies { id } => {
                if let Some(script) = scripts.get(*id) {
                    println!("Script loaded: {}", script.name);
                }
            }
            AssetEvent::Failed { id, error } => {
                println!("Script failed to load: {:?}", error);
            }
            _ => {}
        }
    }
}
```

### Executing Scripts

Use the `RunScript` component for automatic execution:

```rust
#[derive(Component)]
pub struct RunScript {
    pub handle: Handle<FusabiScript>,
    pub executed: bool,
}

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn(RunScript {
        handle: asset_server.load("scripts/init.fsx"),
        executed: false,
    });
}
```

The `RunnerPlugin` will automatically execute scripts when loaded.

## Common Patterns

### One-Time Execution

For scripts that should run once on startup:

```rust
fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn(RunScript {
        handle: asset_server.load("scripts/init.fsx"),
        executed: false,
    });
}
```

### Reloadable Scripts

For development with hot reload:

```rust
fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    // The asset server will automatically reload changed files
    let handle = asset_server.load("scripts/gameplay.fsx");
    commands.spawn((
        RunScript { handle: handle.clone(), executed: false },
        ReloadableScript,
    ));
}

#[derive(Component)]
struct ReloadableScript;

fn reset_on_reload(
    mut events: EventReader<AssetEvent<FusabiScript>>,
    mut query: Query<&mut RunScript, With<ReloadableScript>>,
) {
    for event in events.read() {
        if matches!(event, AssetEvent::Modified { .. }) {
            for mut runner in query.iter_mut() {
                runner.executed = false; // Allow re-execution
            }
        }
    }
}
```

## Next Steps

- Learn about [Integration](integration.md) with Bevy systems
- Explore [Hot Reload](hot-reload.md) capabilities
- Check out [Examples](examples.md) for more use cases
- Review [Compatibility](compatibility.md) information
