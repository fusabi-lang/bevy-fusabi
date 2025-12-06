# Compatibility Matrix

This document provides version compatibility information for bevy-fusabi and its dependencies.

## Version Compatibility

### Bevy Fusabi Versions

| bevy-fusabi | Bevy | fusabi-vm | fusabi-frontend | Rust | Status |
|-------------|------|-----------|-----------------|------|--------|
| 0.1.4 | 0.15 | 0.17.0 | 0.17.0 | 1.75+ | Current |
| 0.1.3 | 0.15 | 0.17.0 | 0.17.0 | 1.75+ | Stable |
| 0.1.2 | 0.15 | 0.17.0 | 0.17.0 | 1.75+ | Stable |
| 0.1.1 | 0.15 | 0.17.0 | 0.17.0 | 1.75+ | Stable |
| 0.1.0 | 0.15 | 0.17.0 | 0.17.0 | 1.75+ | Stable |

### Upcoming Version (vNEXT)

| Component | Version | Notes |
|-----------|---------|-------|
| bevy-fusabi | 0.2.0 | Next minor release |
| Bevy | 0.15 | Same as current |
| fusabi-vm | 0.17.0+ | Compatible with 0.17.x |
| fusabi-frontend | 0.17.0+ | Compatible with 0.17.x |
| fusabi-plugin-runtime | 0.1.0+ | New integration |
| Rust | 1.75+ | MSRV maintained |

## Platform Support

### Tier 1 Platforms

Fully tested and supported:

- **Linux**: x86_64-unknown-linux-gnu
- **macOS**: x86_64-apple-darwin, aarch64-apple-darwin (Apple Silicon)
- **Windows**: x86_64-pc-windows-msvc

### Tier 2 Platforms

Should work but less frequently tested:

- **Windows**: i686-pc-windows-msvc
- **Linux**: aarch64-unknown-linux-gnu

## Feature Compatibility

### Core Features

All platforms support:
- ✅ Asset loading (.fsx, .fzb)
- ✅ Script compilation
- ✅ Script execution
- ✅ Basic hot reload

### Platform-Specific Features

| Feature | Linux | macOS | Windows | Notes |
|---------|-------|-------|---------|-------|
| File watching | ✅ | ✅ | ✅ | Via notify crate |
| Hot reload | ✅ | ✅ | ✅ | Full support |
| Multi-threading | ✅ | ✅ | ✅ | Bevy's task system |

## Bevy Version Migration

### From Bevy 0.14 to 0.15

Not applicable - bevy-fusabi starts with Bevy 0.15 support.

### Future Bevy Versions

We aim to support new Bevy versions within 2-4 weeks of release. Major Bevy updates may require breaking changes in bevy-fusabi.

## Fusabi Version Compatibility

### fusabi-vm and fusabi-frontend

bevy-fusabi tracks the fusabi-vm and fusabi-frontend versions closely:

- **Patch versions** (0.17.x): Generally compatible
- **Minor versions** (0.x.0): May require updates
- **Major versions** (x.0.0): Will require breaking changes

### Bytecode Compatibility

Fusabi bytecode format (.fzb) compatibility:

| Source Version | Runtime Version | Compatible |
|----------------|-----------------|------------|
| 0.17.x | 0.17.x | ✅ Yes |
| 0.16.x | 0.17.x | ⚠️ Maybe* |
| 0.17.x | 0.16.x | ❌ No |

*Backwards compatibility depends on bytecode format changes. Always recompile scripts when upgrading fusabi-vm.

## Plugin Runtime Compatibility (Upcoming)

### fusabi-plugin-runtime Integration

Expected compatibility for future releases:

| bevy-fusabi | fusabi-plugin-runtime | Features |
|-------------|----------------------|----------|
| 0.2.0+ | 0.1.0+ | Full integration |
| 0.1.x | - | Not available |

### Planned Features

The plugin runtime integration will add:
- Advanced hot-reload with state preservation
- Plugin lifecycle management
- Capability-based security
- Metrics and monitoring

## Known Issues

### Current Version (0.1.4)

No known critical issues.

### Known Limitations

1. **VM State**: VM state is not preserved across reloads
2. **Concurrency**: VM is not Send/Sync, limiting parallel execution
3. **FFI**: No foreign function interface yet
4. **Debugging**: Limited debugging support

### Platform-Specific Issues

#### Windows
- No known issues

#### macOS
- No known issues

#### Linux
- No known issues

## Dependency Versions

### Core Dependencies

```toml
[dependencies]
bevy = { version = "0.15", default-features = false, features = ["bevy_asset"] }
fusabi-vm = { version = "0.17.0", features = ["serde"] }
fusabi-frontend = "0.17.0"
thiserror = "1.0"
anyhow = "1.0"
serde = { version = "1.0", features = ["derive"] }
tracing = "0.1"
```

### Development Dependencies

```toml
[dev-dependencies]
bevy = { version = "0.15", default-features = false, features = ["bevy_asset", "multi_threaded"] }
```

## Minimum Supported Rust Version (MSRV)

**Current MSRV: 1.75**

We follow Bevy's MSRV policy:
- Test against stable Rust
- Support at least 2 versions back
- Update MSRV only when necessary
- Announce MSRV changes in release notes

### MSRV Testing

Test with specific Rust version:

```bash
rustup install 1.75
cargo +1.75 build
cargo +1.75 test
```

## Upgrade Guides

### Upgrading bevy-fusabi

#### Patch Updates (0.1.x → 0.1.y)

Patch updates are backwards compatible:

```bash
cargo update -p bevy-fusabi
```

#### Minor Updates (0.x.0 → 0.y.0)

Review the [CHANGELOG](../../../CHANGELOG.md) for breaking changes:

1. Update Cargo.toml
2. Review deprecation warnings
3. Update code for any breaking changes
4. Test thoroughly

#### Major Updates (x.0.0 → y.0.0)

Follow the migration guide (will be provided with major releases).

### Upgrading Bevy

When upgrading Bevy, check the compatibility matrix above and:

1. Ensure bevy-fusabi supports the new Bevy version
2. Update both dependencies simultaneously
3. Review Bevy's migration guide
4. Test asset loading and script execution

## Version Support Policy

### Supported Versions

- **Latest minor version**: Full support
- **Previous minor version**: Security fixes only
- **Older versions**: Not supported

### Security Updates

Security issues are backported to:
- Current minor version
- Previous minor version (if still recent)

Report security issues to: security@fusabi-lang.org (or via GitHub Security Advisory)

## Testing

### Version Testing

We test against:
- Latest stable Rust
- MSRV (1.75)
- Latest Bevy (0.15)
- All supported platforms

### CI Matrix

See `.github/workflows/ci.yml` for the full test matrix.

## Feature Flags

Current feature flags:

```toml
[features]
default = []
# No optional features currently
```

Future features may include:
- `plugin-runtime`: Integration with fusabi-plugin-runtime
- `debug`: Enhanced debugging support
- `metrics`: Performance metrics

## Getting Help

### Version-Specific Issues

When reporting issues, include:
- bevy-fusabi version
- Bevy version
- Rust version
- Platform/OS
- Relevant dependency versions

### Resources

- [GitHub Issues](https://github.com/fusabi-lang/bevy-fusabi/issues)
- [Discussions](https://github.com/fusabi-lang/bevy-fusabi/discussions)
- [Fusabi Documentation](https://github.com/fusabi-lang/fusabi)

## Changelog

See [CHANGELOG.md](../../../CHANGELOG.md) for detailed version history.

## Next Steps

- Review [Getting Started](getting-started.md) for version-specific setup
- Check [Examples](examples.md) for version-compatible code samples
- Read [Integration](integration.md) for version-specific API details
