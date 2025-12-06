# Release Process

This document describes the complete process for releasing a new version of `bevy-fusabi` to crates.io, including branch protection, code review, and documentation requirements.

## Branch Protection

The `main` branch is protected with the following rules:
- All changes must go through pull requests
- At least one CODEOWNERS approval required
- All CI checks must pass before merging
- Direct pushes to `main` are disabled
- Force pushes are disabled

## Prerequisites

Before starting a release:

1. Ensure you have proper permissions to:
   - Create releases on GitHub
   - Publish to crates.io
2. Verify `CARGO_TOKEN` secret is configured in GitHub repository settings
3. All CI checks must be passing on the main branch
4. All documentation is up to date
5. CHANGELOG.md is current with all changes

## Release Steps

### 1. Update Documentation

Before any code changes:

1. **Review vNEXT documentation**:
   - Ensure all required sections exist (see docs/STRUCTURE.md)
   - Verify code samples are working
   - Check all links are valid
   - Update compatibility matrix

2. **Create versioned documentation**:
   ```bash
   # Copy vNEXT to new version
   cp -r docs/versions/vNEXT docs/versions/v0.X.Y

   # Create fresh vNEXT for future work
   mkdir -p docs/versions/vNEXT
   # Seed vNEXT from the new stable version
   cp -r docs/versions/v0.X.Y/* docs/versions/vNEXT/
   ```

### 2. Update Version

1. Update the version number in `Cargo.toml`:
   ```toml
   [package]
   version = "X.Y.Z"
   ```

2. Follow [Semantic Versioning](https://semver.org/):
   - **MAJOR** (X): Breaking changes
   - **MINOR** (Y): New features, backward compatible
   - **PATCH** (Z): Bug fixes, backward compatible

### 3. Update CHANGELOG.md

Update `CHANGELOG.md` with all changes for this release:

1. Move items from `[Unreleased]` to a new version section
2. Add the release date
3. Group changes by category:
   - Added
   - Changed
   - Deprecated
   - Removed
   - Fixed
   - Security
4. Update the comparison links at the bottom
5. Create a new empty `[Unreleased]` section

Example:
```markdown
## [Unreleased]

## [0.2.0] - 2025-12-05

### Added
- Plugin runtime integration for hot-reload scripting
- Versioned documentation structure
- Compatibility matrix

### Changed
- Updated dependency fusabi-vm to 0.17.0

### Fixed
- Script reload edge cases

[Unreleased]: https://github.com/fusabi-lang/bevy-fusabi/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/fusabi-lang/bevy-fusabi/compare/v0.1.4...v0.2.0
```

### 4. Create Release Branch and PR

```bash
# Create release branch
git checkout -b release-v0.X.Y

# Add all changes
git add Cargo.toml CHANGELOG.md docs/

# Commit with clear message
git commit -m "Prepare release v0.X.Y

- Update version to 0.X.Y
- Update CHANGELOG
- Update versioned documentation
"

# Push branch
git push origin release-v0.X.Y

# Create pull request
gh pr create --title "Release v0.X.Y" --body "
## Release Checklist

- [ ] Version updated in Cargo.toml
- [ ] CHANGELOG.md updated
- [ ] Documentation versioned and updated
- [ ] All CI checks passing
- [ ] CODEOWNERS approval received

## Changes

See CHANGELOG.md for full list of changes.
"
```

### 5. Code Review and Approval

1. Wait for CODEOWNERS review and approval
2. Ensure all CI checks pass
3. Address any feedback from reviewers
4. Get final approval

### 6. Merge and Tag

Once approved:

```bash
# Merge PR (via GitHub UI or CLI)
gh pr merge --squash

# Pull latest main
git checkout main
git pull origin main

# Create and push tag
git tag v0.X.Y
git push origin v0.X.Y
```

This will trigger three GitHub Actions workflows:
- **CI**: Runs tests and checks
- **Release**: Creates a GitHub release with changelog
- **Publish**: Publishes the crate to crates.io

### 7. Verify Publication

1. Check GitHub Actions to ensure all workflows completed successfully
2. Verify the package appears on [crates.io](https://crates.io/crates/bevy-fusabi)
3. Check the [GitHub release](https://github.com/fusabi-lang/bevy-fusabi/releases) was created
4. Verify docs.rs build succeeded

### 8. Post-Release

1. Update documentation site (if applicable)
2. Announce the release:
   - GitHub Discussions
   - Discord/community channels
   - Social media (if appropriate)
3. Close any resolved issues
4. Update related projects/examples

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
5. Contact crates.io support if needed

### CI Failures

If CI fails before publishing:
- Fix the issues locally
- Push the fixes to the release branch
- Wait for CI to pass
- If needed after merge, delete and recreate the tag:
  ```bash
  git tag -d v0.X.Y
  git push origin :refs/tags/v0.X.Y
  git tag v0.X.Y
  git push origin v0.X.Y
  ```

### Documentation Build Failures

If docs.rs build fails:
1. Check the build log on docs.rs
2. Verify all dependencies build correctly
3. Test documentation locally: `cargo doc --all-features`
4. Fix any issues and release a patch version if needed

## Yanking a Release

If a release needs to be yanked due to critical bugs:

```bash
# Yank the problematic version
cargo yank --vers 0.X.Y

# Announce the yank and reason
# Prepare a patch release immediately
```

To un-yank (only if the issue was a false alarm):

```bash
cargo yank --vers 0.X.Y --undo
```

Note: Yanking doesn't delete the version, it just prevents new projects from using it. Existing projects with the version in Cargo.lock are unaffected.

## Release Cadence

- **Patch releases**: As needed for bug fixes
- **Minor releases**: Every 4-6 weeks for new features
- **Major releases**: When breaking changes are necessary

Consider aligning releases with Bevy releases when possible to maintain compatibility.

## CODEOWNERS

See `.github/CODEOWNERS` for the list of release approvers. At least one CODEOWNER must approve all release PRs.
