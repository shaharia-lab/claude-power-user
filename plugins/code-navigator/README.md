# Code Navigator

Efficient code navigation plugin for Claude Code CLI using the `codenav` tool. Save up to 92% on token usage by navigating codebases intelligently instead of blindly reading files.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Overview

Code Navigator teaches Claude Code to use the `codenav` CLI tool for efficient codebase exploration. Instead of reading dozens of files blindly, codenav builds indexed call graphs that allow:

- **Finding functions** and symbols instantly
- **Tracing dependencies** (what does this call?)
- **Finding callers** (who calls this?)
- **Understanding call paths** between functions
- **Analyzing architecture** and code relationships

**Result**: 87-92% token savings and faster code understanding.

## Installation

### From Claude Code Plugins Marketplace

```bash
# Add the marketplace (if not already added)
/plugin marketplace add https://github.com/shaharia-lab/claude-code-plugins.git

# Install the plugin
/plugin install code-navigator@shaharia-lab
```

### Manual Installation

```bash
# Clone into Claude plugins directory
git clone https://github.com/shaharia-lab/claude-code-plugins.git ~/.claude/plugins/code-navigator

# Or clone anywhere and symlink
git clone https://github.com/shaharia-lab/claude-code-plugins.git ~/Projects/code-navigator
ln -s ~/Projects/code-navigator/plugins/code-navigator ~/.claude/plugins/code-navigator
```

Restart Claude Code CLI or restart your terminal.

## Prerequisites

### 1. Codenav CLI Tool

This plugin requires the `codenav` CLI tool to be installed and available in your PATH.

```bash
# Check if codenav is installed
codenav --version

# If not installed, install codenav
# (Installation instructions depend on your codenav distribution)
```

### 2. Indexed Codebase

Codenav works with pre-indexed codebases:

```bash
# Index your codebase
codenav index /path/to/your/project --output /tmp/project.bin

# Verify the index
codenav query --graph /tmp/project.bin --count
```

**Tip**: Add indexing to your project setup or CI pipeline to keep it updated.

## Usage

### Automatic (Recommended)

Claude automatically uses this skill when you ask navigation-related questions:

```
"How does authentication work in this codebase?"
"Find where the user data comes from"
"Trace the error handling flow"
"Understand the API endpoint structure"
```

Claude will use codenav to navigate efficiently instead of reading files blindly.

### Manual Invocation

```bash
/codenav-navigation functionName
```

## What's Included

### Skill: codenav-navigation

**Purpose**: Teach Claude to navigate code efficiently using codenav.

**Features**:
- Token efficiency rules (codenav → grep → read)
- Complete codenav command reference
- Common navigation patterns
- Real-world examples
- Integration strategies

**Files**:
- `SKILL.md` - Main skill instructions
- `reference.md` - Complete command reference
- `examples.md` - Real-world usage examples

## How It Works

### Traditional Approach (Token Inefficient)
```bash
# ❌ Wastes ~40,000 tokens
grep -r "pattern" . | head -100
read file1.ts    # 5000 tokens
read file2.ts    # 5000 tokens
read file3.ts    # 5000 tokens
# ... repeat 10+ times
```

### Code Navigator Approach (Token Efficient)
```bash
# ✅ Uses ~3,000 tokens
codenav query --name "*pattern*"           # 100 tokens
codenav callers functionName               # 100 tokens
codenav trace --from "function" --depth 2  # 200 tokens
read identified_file.ts                    # 5000 tokens
# Result: 92% token savings!
```

## Core Commands

### Find Functions
```bash
codenav query --graph /tmp/project.bin --name "*auth*" --output table
```

### Find Callers (Who calls this?)
```bash
codenav callers --graph /tmp/project.bin authenticate --show-lines
```

### Trace Dependencies (What does this call?)
```bash
codenav trace --graph /tmp/project.bin --from "handleRequest" --depth 2
```

### Find Call Paths
```bash
codenav path --graph /tmp/project.bin --from "login" --to "database"
```

### Analyze Code
```bash
codenav analyze --graph /tmp/project.bin complexity --limit 20
codenav analyze --graph /tmp/project.bin circular
codenav analyze --graph /tmp/project.bin hotspots --limit 10
```

## Real-World Examples

### Example 1: Finding Data Source

**Without Code Navigator** (~40,000 tokens):
- Read 20+ files blindly
- Search through multiple directories
- Manual tracing of dependencies

**With Code Navigator** (~3,000 tokens):
1. Query for data-related functions
2. Trace backwards to find source
3. Read only identified files

**Result**: 92% token savings

### Example 2: Understanding Authentication

**Task**: How does login work?

```bash
# Find auth functions
codenav query --graph INDEX --name "*auth*"

# Find who uses it
codenav callers authenticate --show-lines

# Trace the flow
codenav trace --from "authenticate" --depth 2

# Find path from UI to database
codenav path --from "LoginPage" --to "database"
```

**Result**: Complete understanding in 4 commands instead of reading 15+ files.

See [examples.md](./skills/codenav-navigation/examples.md) for 8 complete real-world examples.

## Token Efficiency

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Files Read | 15-20 | 2-3 | 85% fewer |
| Token Usage | 40,000+ | 3,000-5,000 | 87-92% savings |
| Time to Understanding | Slow | Fast | Much faster |
| Accuracy | Hit-or-miss | Precise | Higher quality |

## When to Use Each Tool

| Tool | Use For | Token Cost |
|------|---------|------------|
| **codenav** | Navigation, call graphs, dependencies | Low (~100) |
| **grep** | Finding text, strings, constants | Medium (~500-1000) |
| **read** | Reading specific file content | High (~5000-10000) |

**Golden Rule**: Navigate with codenav first, search with grep second, read last.

## Configuration

### Default Index Location

The skill uses `/tmp/codebase.bin` by default. You can:

1. **Set per-project**: Create a `.codenav` file in your project root:
   ```
   /path/to/your/project.bin
   ```

2. **Use environment variable**:
   ```bash
   export CODENAV_INDEX=/path/to/index.bin
   ```

3. **Specify in commands**: Always provide `--graph` parameter

### Index Update Strategy

Keep your index updated:

```bash
# Manual update
codenav index /path/to/project --output /tmp/project.bin

# Or add to git hooks (.git/hooks/post-merge)
#!/bin/bash
codenav index . --output /tmp/project.bin

# Or add to CI/CD pipeline
```

## Limitations

Codenav **cannot** determine:
- GraphQL query documents
- API endpoint URLs
- CMS queries (Contentful, Strapi, etc.)
- Hardcoded constants
- Configuration values
- Comments and documentation

**For these cases**: Use grep to find files, then read to inspect.

## Best Practices

1. ✅ **Always start with codenav** for navigation
2. ✅ **Use callers and trace** to understand relationships
3. ✅ **Limit depth** on trace to avoid overwhelming output
4. ✅ **Use --show-lines** to get precise locations
5. ✅ **Export JSON** for programmatic processing
6. ✅ **Read files last** after identifying exactly what you need
7. ❌ **Don't skip codenav** and jump straight to grep/read
8. ❌ **Don't trace too deep** without limits
9. ❌ **Don't query all functions** without filtering

## Troubleshooting

### "Codenav not found"
```bash
# Check if codenav is installed
which codenav

# Install codenav if needed
# (Installation method depends on your distribution)
```

### "No results found"
```bash
# Check if index exists
ls -lh /tmp/codebase.bin

# Try broader pattern
codenav query --graph /tmp/codebase.bin --name "*pattern*"

# Verify case sensitivity
```

### "Index not found"
```bash
# Re-index your codebase
codenav index /path/to/project --output /tmp/project.bin
```

## Language Support

Codenav typically supports:
- JavaScript/TypeScript
- Python
- Go
- Java
- Ruby
- PHP
- C/C++
- Rust

Check your codenav documentation for specific language support.

## Tech Stack

- **Tool**: codenav CLI
- **Claude Code CLI**: v1.0.33+
- **License**: MIT

## Contributing

Contributions welcome! Please:
1. Add new examples to `examples.md`
2. Document new codenav features in `reference.md`
3. Update workflows in `SKILL.md`

## Support

- **Issues**: [GitHub Issues](https://github.com/shaharia-lab/claude-code-plugins/issues)
- **Documentation**: See skill files in `skills/codenav-navigation/`

## License

MIT License - see [LICENSE](../../LICENSE)

## Repository

https://github.com/shaharia-lab/claude-code-plugins

---

**Remember**: Navigate with codenav, search with grep, read last! 🚀

**Token Savings**: Up to 92% reduction in token usage for code exploration tasks.
