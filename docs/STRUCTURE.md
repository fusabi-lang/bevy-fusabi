# Documentation Structure

This document describes the required structure and sections for bevy-fusabi documentation.

## Directory Organization

```
docs/
├── STRUCTURE.md          # This file - describes doc organization
├── RELEASE.md           # Release process and guidelines
├── versions/            # Versioned documentation
│   ├── vNEXT/          # Upcoming release docs
│   │   ├── README.md
│   │   ├── getting-started.md
│   │   ├── integration.md
│   │   ├── hot-reload.md
│   │   ├── compatibility.md
│   │   └── examples.md
│   ├── v0.2.0/         # Stable version docs
│   └── v0.1.0/         # Previous version docs
```

## Required Sections

Each versioned documentation directory MUST contain the following sections:

### 1. README.md
- Overview of bevy-fusabi for this version
- Key features
- Quick start guide
- Links to other documentation sections

### 2. getting-started.md
- Installation instructions
- Basic setup
- Your first Fusabi script in Bevy
- Common patterns

### 3. integration.md
- Bevy plugin integration details
- Asset loading system
- Script execution
- ECS integration patterns
- Plugin runtime integration (if available)

### 4. hot-reload.md
- Hot reload capabilities
- Setup and configuration
- Development workflow
- Best practices

### 5. compatibility.md
- Bevy version compatibility matrix
- Fusabi version compatibility
- Rust version requirements
- Known issues and workarounds

### 6. examples.md
- Detailed walkthrough of provided examples
- Common use cases
- Integration patterns
- Sample projects

## Documentation Standards

### Code Samples
- All code samples must be tested and working
- Include complete examples when possible
- Use proper syntax highlighting
- Show both source and compiled forms where relevant

### Version-Specific Content
- Clearly mark deprecated features
- Document migration paths between versions
- Include changelog references

### Links
- Use relative links within versioned docs
- External links should be version-appropriate
- Keep broken link checking enabled in CI

## CI Requirements

The CI pipeline MUST check:
1. All required sections exist in vNEXT
2. No broken internal links
3. Code samples compile (if possible)
4. Markdown formatting is valid

## Release Process Integration

When cutting a release:
1. Copy `versions/vNEXT/` to `versions/vX.Y.Z/`
2. Create new empty `versions/vNEXT/` for next release
3. Update root README.md to point to latest stable version
4. Ensure all documentation is up to date with code changes
