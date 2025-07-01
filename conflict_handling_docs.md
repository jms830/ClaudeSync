# Conflict Handling Documentation

## Overview

ClaudeSync now includes comprehensive conflict detection and resolution capabilities to handle cases where files are modified both locally and remotely. This system ensures you never lose work due to sync conflicts.

## Features

### 🔍 **Conflict Detection**
- Automatic detection of files modified in both locations
- MD5 checksum comparison for accurate conflict identification
- Smart file scanning with performance optimization

### 🔧 **Resolution Strategies**
- **Interactive Resolution**: Manual conflict resolution with diff viewing
- **Auto-Resolution**: Configurable automatic conflict handling
- **External Editor Support**: Integration with VS Code, Sublime, Vim, etc.
- **Preview & Compare**: Side-by-side and unified diff views

### ⚙️ **Integration**
- Seamless integration with existing sync process
- Non-invasive design preserves existing functionality
- Configurable behavior for different workflows

## Command Reference

### Core Commands

```bash
# Detect conflicts in current project
claudesync conflict detect

# Resolve conflicts interactively  
claudesync conflict resolve

# Show conflict status overview
claudesync conflict status

# Show detailed diff for specific file
claudesync conflict diff <file_path>

# Configure auto-resolution strategy
claudesync conflict configure --strategy local-wins
```

### Quick Actions (Aliases)

```bash
# Quick scan for conflicts
claudesync conflict scan

# Quick interactive resolution
claudesync conflict fix
```

### Auto-Resolution Options

```bash
# Auto-resolve keeping local versions
claudesync conflict detect --auto-resolve local

# Auto-resolve keeping remote versions  
claudesync conflict detect --auto-resolve remote

# Skip conflicts (leave both versions)
claudesync conflict detect --auto-resolve skip
```

## Enhanced Push Command

The `push` command now includes conflict handling options:

```bash
# Standard push with conflict detection (default)
claudesync push

# Skip conflict detection (legacy behavior)
claudesync push --skip-conflicts

# Auto-resolve conflicts during push
claudesync push --auto-resolve local   # Keep local versions
claudesync push --auto-resolve remote  # Keep remote versions
```

## Resolution Strategies

### 1. Interactive Resolution

Most flexible option with full user control:

```bash
claudesync conflict resolve
```

**Features:**
- Unified and side-by-side diff viewing
- External editor integration for manual merging
- Per-file resolution decisions
- Progress tracking with summary

**Resolution Options:**
1. **Keep Local** - Overwrite remote with local version
2. **Keep Remote** - Overwrite local with remote version  
3. **Manual Merge** - Open external editor for custom merge
4. **Show Comparison** - View detailed side-by-side diff
5. **Skip File** - Leave both versions unchanged

### 2. Auto-Resolution Strategies

Configure automatic conflict handling:

```bash
# Set default strategy
claudesync conflict configure --strategy local-wins
claudesync conflict configure --strategy remote-wins
claudesync conflict configure --strategy auto-merge
```

**Available Strategies:**
- **`local-wins`**: Local files take precedence (default for most users)
- **`remote-wins`**: Remote files take precedence (good for shared workflows)
- **`auto-merge`**: Attempt intelligent merging (experimental)

### 3. Manual Merge with External Editors

When choosing manual merge, ClaudeSync will:

1. Create temporary files with local and remote versions
2. Open your preferred editor with the merge file
3. Wait for you to resolve conflicts and save
4. Apply the merged result

**Supported Editors:**
- VS Code (`code`)
- Sublime Text (`subl`) 
- Atom (`atom`)
- Vim (`vim`)
- Nano (`nano`)
- Notepad (Windows)

## Workflow Examples

### Development Workflow

```bash
# Daily development workflow
claudesync conflict configure --strategy local-wins  # Set once
claudesync push                                       # Auto-resolves keeping local
```

### Collaborative Workflow

```bash
# Before making changes
claudesync conflict status                           # Check for conflicts
claudesync conflict resolve                          # Resolve any conflicts
claudesync push                                      # Clean sync
```

### Review Workflow

```bash
# Check what conflicts exist
claudesync conflict detect

# Review specific files
claudesync conflict diff project-instructions.md

# Resolve selectively  
claudesync conflict resolve --file project-instructions.md
```

## Configuration

### Global Settings

Conflict resolution preferences are stored per-project:

```bash
# View current configuration
claudesync config get conflict_resolution_strategy

# Set auto-resolution strategy
claudesync config set conflict_resolution_strategy local-wins
```

### Available Settings

| Setting | Values | Description |
|---------|--------|-------------|
| `conflict_resolution_strategy` | `local-wins`, `remote-wins`, `auto-merge`, `null` | Default auto-resolution behavior |

## Integration with Existing Features

### File Watching

Conflict detection works with file watching:

```bash
# Start watching with conflict handling
claudesync watch start

# Skip conflicts during auto-sync (not recommended)
claudesync watch start --skip-conflicts
```

### Workspace Operations

Multi-project conflict handling:

```bash
# Check conflicts across all projects
claudesync workspace status

# Resolve conflicts project by project
claudesync workspace sync-all  # Will prompt for conflict resolution
```

### Project Instructions

Project instructions participate in conflict detection:

```bash
# Conflicts in instructions are handled like any other file
claudesync project instructions status
claudesync conflict resolve --file project-instructions.md
```

## Advanced Features

### Diff Viewing

**Unified Diff:**
```
@@ -1,3 +1,4 @@
 # Project Title
+## New Section
 
 Some content
-Old line
+New line
```

**Side-by-Side Diff:**
```
LOCAL                                    | REMOTE
---------------------------------------- | ----------------------------------------
# Project Title                         | # Project Title  
                                        | ## New Section
Some content                            | Some content
Old line                                | New line
```

### Auto-Merge Intelligence

The auto-merge strategy uses simple heuristics:

1. **Subset Detection**: If one version contains all lines of the other, use the larger version
2. **Line-based Merging**: Combine non-conflicting changes when possible
3. **Fallback**: Default to local-wins if merge cannot be determined safely

### Performance Characteristics

- **Detection**: O(n) where n = number of files
- **Resolution**: Instant for auto-strategies, user-dependent for interactive
- **Memory Usage**: Minimal - only conflicted files loaded into memory
- **Network Impact**: None during detection (uses cached file content)

## Error Handling

### Common Scenarios

**File Read Errors:**
```bash
❌ Error reading local file example.md: Permission denied
```
*Solution: Check file permissions*

**External Editor Failures:**
```bash
⚠️ Could not launch external editor. Manual merge not available.
```
*Solution: Install a supported editor or use other resolution options*

**Merge Conflicts:**
```bash
❌ Manual merge failed: Invalid encoding
```
*Solution: Use "Keep Local" or "Keep Remote" resolution*

### Recovery Options

If conflict resolution fails:

1. **Check file status**: `claudesync conflict status`
2. **Manual file editing**: Edit files directly and re-sync
3. **Reset to known state**: Use `--auto-resolve` with known strategy
4. **Skip problematic files**: Use "Skip" resolution option

## Best Practices

### 1. Development Workflow

```bash
# Set up once per project
claudesync conflict configure --strategy local-wins

# Daily development
claudesync push  # Conflicts auto-resolved

# When collaboration is heavy
claudesync conflict status    # Check before starting work
claudesync conflict resolve   # Resolve conflicts manually
```

### 2. Team Collaboration

```bash
# Before making significant changes
claudesync conflict detect
claudesync conflict resolve

# After major remote updates
claudesync conflict status
claudesync push --auto-resolve remote  # Accept team changes
```

### 3. Safe Practices

- **Always review conflicts** before auto-resolving in production
- **Use manual merge** for important files like documentation
- **Set appropriate default strategy** based on your workflow
- **Regular conflict checks** prevent accumulation of issues

### 4. File-Specific Strategies

Different files may need different approaches:

- **Code files**: Often local-wins (you're actively developing)
- **Documentation**: Manual merge (preserve both perspectives) 
- **Configuration**: Remote-wins (team standards)
- **Generated files**: Skip conflicts (regenerate instead)

## Troubleshooting

### Performance Issues

If conflict detection is slow:

```bash
# Use specific categories to reduce file scope
claudesync conflict detect --category production_code

# Check for large files that might be slowing detection
claudesync conflict status
```

### Resolution Not Applied

If resolutions don't seem to take effect:

```bash
# Verify the resolution was applied
claudesync conflict status

# Manually sync after resolution
claudesync push
```

### External Editor Issues

If manual merge doesn't work:

1. **Install a supported editor**:
   ```bash
   # Ubuntu/Debian
   sudo apt install code  # VS Code
   
   # macOS  
   brew install --cask visual-studio-code
   
   # Windows
   winget install Microsoft.VisualStudioCode
   ```

2. **Use alternative resolution**:
   - Choose "Keep Local" or "Keep Remote"
   - Edit files manually and re-sync

## Migration from Previous Versions

Existing ClaudeSync users get conflict handling automatically:

- **Default behavior**: Conflicts detected and require manual resolution
- **Backward compatibility**: Use `--skip-conflicts` for legacy behavior  
- **Gradual adoption**: Start with `claudesync conflict status` to see what conflicts exist

### Recommended Migration Steps

1. **Check existing conflicts**:
   ```bash
   claudesync conflict status
   ```

2. **Resolve any conflicts**:
   ```bash
   claudesync conflict resolve
   ```

3. **Configure preferred strategy**:
   ```bash
   claudesync conflict configure --strategy local-wins
   ```

4. **Resume normal workflow**:
   ```bash
   claudesync push  # Now with conflict handling
   ```

---

## Summary

The conflict handling system provides:

✅ **Comprehensive Detection** - Never miss a conflict
✅ **Flexible Resolution** - Interactive and automatic options  
✅ **External Tool Integration** - Use your preferred editor
✅ **Performance Optimized** - Fast detection and resolution
✅ **Non-Invasive Design** - Existing workflows preserved
✅ **Configurable Behavior** - Adapt to your team's needs

**Result**: Worry-free synchronization with professional conflict management.
