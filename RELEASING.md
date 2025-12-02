# Release Process

This document describes the process for releasing a new version of `bevy-fusabi` to crates.io.

## Prerequisites

1. Ensure you have proper permissions to publish to crates.io
2. The `CARGO_TOKEN` secret must be configured in GitHub repository settings
3. All CI checks must be passing on the main branch

## Release Steps

### 1. Update Version

Update the version number in `Cargo.toml`:

```toml
[package]
version = "X.Y.Z"
```

Follow [Semantic Versioning](https://semver.org/):
- **MAJOR** (X): Breaking changes
- **MINOR** (Y): New features, backward compatible
- **PATCH** (Z): Bug fixes, backward compatible

### 2. Update CHANGELOG.md

Update `CHANGELOG.md` with all changes for this release:

1. Move items from `[Unreleased]` to a new version section
2. Add the release date
3. Update the comparison links at the bottom
4. Create a new empty `[Unreleased]` section

Example:
```markdown
## [Unreleased]

## [0.2.0] - 2025-12-01

### Added
- New feature X
- New feature Y

### Changed
- Updated dependency Z

[Unreleased]: https://github.com/fusabi-lang/bevy-fusabi/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/fusabi-lang/bevy-fusabi/compare/v0.1.0...v0.2.0
```

### 3. Commit Changes

```bash
git add Cargo.toml CHANGELOG.md
git commit -m "Release v0.X.Y"
git push origin main
```

### 4. Create and Push Tag

```bash
git tag v0.X.Y
git push origin v0.X.Y
```

This will trigger three GitHub Actions workflows:
- **CI**: Runs tests and checks
- **Release**: Creates a GitHub release with changelog
- **Publish**: Publishes the crate to crates.io

### 5. Verify Publication

1. Check GitHub Actions to ensure all workflows completed successfully
2. Verify the package appears on [crates.io](https://crates.io/crates/bevy-fusabi)
3. Check the [GitHub release](https://github.com/fusabi-lang/bevy-fusabi/releases) was created

### 6. Post-Release

1. Announce the release on relevant channels
2. Update documentation if needed
3. Close any resolved issues/PRs

## Dry Run (Testing)

To test the release process without publishing:

1. Go to GitHub Actions
2. Select "Publish to crates.io" workflow
3. Click "Run workflow"
4. Check the "Perform a dry run" option
5. Run the workflow

This will run all checks including `cargo publish --dry-run` without actually publishing to crates.io.

## Troubleshooting

### Version Mismatch Error

If you see "Tag version does not match Cargo.toml version":
- Ensure `Cargo.toml` version matches the git tag (without the 'v' prefix)
- Tag: `v0.2.0` should match Cargo.toml: `version = "0.2.0"`

### Failed to Publish

If publishing fails:
1. Check that `CARGO_TOKEN` secret is set correctly
2. Ensure the version doesn't already exist on crates.io
3. Verify all dependencies are available on crates.io
4. Review the error logs in GitHub Actions

### CI Failures

If CI fails before publishing:
- Fix the issues locally
- Push the fixes
- Delete and recreate the tag:
  ```bash
  git tag -d v0.X.Y
  git push origin :refs/tags/v0.X.Y
  git tag v0.X.Y
  git push origin v0.X.Y
  ```

## Yanking a Release

If a release needs to be yanked:

```bash
cargo yank --vers 0.X.Y
```

To un-yank:

```bash
cargo yank --vers 0.X.Y --undo
```

Note: Yanking doesn't delete the version, it just prevents new projects from using it.
