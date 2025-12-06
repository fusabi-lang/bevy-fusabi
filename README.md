# Bevy Fusabi

[![Crates.io](https://img.shields.io/crates/v/bevy-fusabi.svg)](https://crates.io/crates/bevy-fusabi)
[![Docs.rs](https://docs.rs/bevy-fusabi/badge.svg)](https://docs.rs/bevy-fusabi)
[![CI](https://github.com/fusabi-lang/bevy-fusabi/workflows/CI/badge.svg)](https://github.com/fusabi-lang/bevy-fusabi/actions)
[![License](https://img.shields.io/crates/l/bevy-fusabi.svg)](https://github.com/fusabi-lang/bevy-fusabi#license)

Bevy plugin for Fusabi scripting language - bringing hot-reloadable scripting to your Bevy games.

## Features
- .fsx and .fzb asset loading
- Hot reloading support
- ECS integration
- Automatic script compilation
- Plugin runtime integration (upcoming)

## Quick Start

Add to your `Cargo.toml`:

```toml
[dependencies]
bevy = "0.15"
bevy-fusabi = "0.1.4"
```

Basic usage:

```rust
use bevy::prelude::*;
use bevy_fusabi::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(FusabiPlugin)
        .add_plugins(RunnerPlugin)
        .run();
}
```

## Documentation

- **[Getting Started Guide](docs/versions/vNEXT/getting-started.md)** - Installation and first steps
- **[Integration Guide](docs/versions/vNEXT/integration.md)** - Bevy ECS integration patterns
- **[Hot Reload Guide](docs/versions/vNEXT/hot-reload.md)** - Development workflow
- **[Examples](docs/versions/vNEXT/examples.md)** - Code examples and patterns
- **[Compatibility Matrix](docs/versions/vNEXT/compatibility.md)** - Version compatibility

See [docs/STRUCTURE.md](docs/STRUCTURE.md) for documentation organization.

## Contributing

See [docs/RELEASE.md](docs/RELEASE.md) for release process and contribution guidelines.

## License

Licensed under either of Apache License, Version 2.0 or MIT license at your option.

