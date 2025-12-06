# Bevy Fusabi Documentation

Welcome to the Bevy Fusabi documentation. This integration brings the Fusabi scripting language to the Bevy game engine, enabling hot-reloadable scripts within your ECS-based games.

## Overview

Bevy Fusabi is a plugin that integrates the Fusabi scripting language with the Bevy game engine. It provides:

- **Asset Loading**: Load `.fsx` (source) and `.fzb` (bytecode) files as Bevy assets
- **Hot Reloading**: Automatically reload scripts during development
- **ECS Integration**: Execute scripts within Bevy's Entity Component System
- **Plugin Runtime**: Advanced hot-reload capabilities via fusabi-plugin-runtime (upcoming)

## Quick Start

Add to your `Cargo.toml`:

```toml
[dependencies]
bevy = "0.15"
bevy-fusabi = "0.1.4"
```

Minimal example:

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
    let handle = asset_server.load("scripts/hello.fsx");
    commands.spawn(RunScript {
        handle,
        executed: false,
    });
}
```

## Documentation Sections

- **[Getting Started](getting-started.md)** - Installation and basic setup
- **[Integration](integration.md)** - Bevy plugin integration details
- **[Hot Reload](hot-reload.md)** - Hot reload capabilities and workflow
- **[Compatibility](compatibility.md)** - Version compatibility matrix
- **[Examples](examples.md)** - Detailed examples and use cases

## Features by Version

### Current (0.1.4)
- Asset loading for .fsx and .fzb files
- Basic hot reloading support
- ECS integration with RunScript component
- Script execution via fusabi-vm

### Upcoming (vNEXT)
- Enhanced plugin runtime integration
- Advanced hot-reload scripting
- Expanded examples
- Improved documentation
- Better error handling

## Community

- GitHub: [fusabi-lang/bevy-fusabi](https://github.com/fusabi-lang/bevy-fusabi)
- Issues: [Issue Tracker](https://github.com/fusabi-lang/bevy-fusabi/issues)
- Fusabi Language: [fusabi-lang](https://github.com/fusabi-lang/fusabi)

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](../../LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](../../LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.
