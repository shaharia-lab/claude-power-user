---
name: codenav-navigation
description: Use codenav CLI tool for efficient code navigation to explore function dependencies, call graphs, and code relationships. CRITICAL for token efficiency - always use codenav BEFORE grep/read when navigating codebases. Use when tracing function calls, finding dependencies, understanding architecture, or locating code patterns.
allowed-tools: Bash, Grep, Read
argument-hint: [function-name] or [pattern]
user-invocable: true
---

# Efficient Code Navigation with Codenav

**CRITICAL: Always use codenav for navigation BEFORE using grep or read to save thousands of tokens.**

This skill teaches efficient codebase exploration using the `codenav` CLI tool. Codenav builds indexed call graphs that let you navigate code relationships without reading files.

## Token Efficiency Rules

### ✅ ALWAYS Use Codenav First For:
- Finding functions/symbols in the codebase
- Tracing function dependencies (what does this call?)
- Finding callers (who calls this function?)
- Understanding call paths between functions
- Exploring code architecture and relationships
- Locating functions by package or file pattern

### ⚠️ Use Grep ONLY For:
- Finding files containing specific text/strings
- Locating hardcoded values or constants
- Finding configuration values
- Searching for comments or documentation

### ⚠️ Use Read ONLY For:
- Reading specific files identified by codenav or grep
- Understanding implementation details AFTER finding the right file
- Viewing actual source code content

## Core Codenav Commands

### 1. Query Functions/Symbols
```bash
# Find functions by name pattern
codenav query --graph /tmp/codebase.bin --name "*pattern*" --output table

# Filter by type
codenav query --graph /tmp/codebase.bin --name "*auth*" --type function --output table --limit 20

# Filter by package
codenav query --graph /tmp/codebase.bin --package "api" --output table --limit 30

# Get function details in JSON
codenav query --graph /tmp/codebase.bin --name "functionName" --output json

# Count total nodes
codenav query --graph /tmp/codebase.bin --count
```

### 2. Find Callers (Who Calls This?)
```bash
# Find all functions that call this one
codenav callers --graph /tmp/codebase.bin functionName --show-lines --output tree

# Get count only
codenav callers --graph /tmp/codebase.bin functionName --count
```

### 3. Trace Dependencies (What Does This Call?)
```bash
# Trace what a function calls
codenav trace --graph /tmp/codebase.bin --from "functionName" --depth 2 --show-lines

# Deeper trace
codenav trace --graph /tmp/codebase.bin --from "functionName" --depth 3 --output tree
```

### 4. Find Call Paths
```bash
# Find shortest path between two functions
codenav path --graph /tmp/codebase.bin --from "startFunc" --to "endFunc"

# Find multiple paths
codenav path --graph /tmp/codebase.bin --from "startFunc" --to "endFunc" --limit 5
```

### 5. Analyze Code
```bash
# Find most complex functions
codenav analyze --graph /tmp/codebase.bin complexity --limit 20

# Find circular dependencies
codenav analyze --graph /tmp/codebase.bin circular

# Find hotspots (most called functions)
codenav analyze --graph /tmp/codebase.bin hotspots --limit 10
```

## Efficient Workflow Pattern

When investigating code (e.g., finding where data comes from):

### Phase 1: Navigate with Codenav (REQUIRED FIRST STEP)
```bash
# Step 1: Find the starting function
codenav query --graph INDEX --name "*dataHandler*" --output table

# Step 2: Find who calls it (trace backwards)
codenav callers --graph INDEX dataHandler --show-lines

# Step 3: Continue tracing up the call chain
codenav callers --graph INDEX parentFunction --show-lines

# Step 4: Get the full dependency tree
codenav trace --graph INDEX --from "entryPoint" --depth 3
```

### Phase 2: Identify Specific Content (When Needed)
```bash
# Only NOW use grep to find files with specific text
Grep pattern: "specific.text.pattern"
output_mode: files_with_matches
```

### Phase 3: Read Identified Files (Final Step)
```bash
# Read only the specific files identified by codenav/grep
Read file_path: /path/identified/by/codenav/file.ts
```

## Real Example: Finding Data Source

**Task**: Find where a specific data field originates.

**WRONG Approach (Wastes Tokens)**:
```bash
# ❌ Don't do this - wastes tokens
grep -r "dataField" . | head -100
read file1.ts
read file2.ts
read file3.ts
# ... reading many files blindly
```

**CORRECT Approach (Saves Tokens)**:
```bash
# ✅ Step 1: Use codenav to find related functions
codenav query --graph /tmp/codebase.bin --name "*data*" --output table
# Output: dataHandler, dataTransformer, dataFetcher

# ✅ Step 2: Trace backwards to find data source
codenav callers --graph /tmp/codebase.bin dataHandler --show-lines
# Output: Called by processData at line 57

# ✅ Step 3: Continue tracing
codenav callers --graph /tmp/codebase.bin processData --show-lines
# Output: Called by apiController in 3 locations

# ✅ Step 4: Get function details
codenav query --graph /tmp/codebase.bin --name "dataHandler" --output json
# Output: Shows file path, line numbers, parameters

# ✅ Step 5: NOW use grep to find the specific text (if needed)
Grep pattern: "dataField"
output_mode: files_with_matches
# Output: 3 files found

# ✅ Step 6: Read only the identified files
Read file_path: /path/to/constants.ts
Read file_path: /path/to/dataHandler.ts
```

**Token Savings**: ~80% reduction by using codenav to navigate before reading.

## Common Patterns

### Pattern 1: Understanding a Feature
```bash
# Find all related functions
codenav query --graph INDEX --name "*feature*" --output table

# Get dependency tree
codenav trace --graph INDEX --from "mainFeatureFunction" --depth 2

# Find usage
codenav callers --graph INDEX mainFeatureFunction
```

### Pattern 2: Finding API Data Source
```bash
# Start from UI component displaying data
codenav query --graph INDEX --name "*Component*" --type function

# Trace back through hooks/services
codenav callers --graph INDEX useDataHook

# Find API call
codenav trace --graph INDEX --from "useDataHook" --depth 3

# Read only the API endpoint file
Read file_path: /identified/api/endpoint.ts
```

### Pattern 3: Understanding Error Flow
```bash
# Find error handler
codenav query --graph INDEX --name "*error*" --type function

# Find all callers
codenav callers --graph INDEX errorHandler --show-lines

# Trace error propagation
codenav path --graph INDEX --from "errorSource" --to "errorHandler"
```

## Codenav Limitations (When to Use Grep/Read)

Codenav **CANNOT** determine:
- GraphQL query documents (use grep to find `.gql` or `gql\``)
- API endpoint URLs (use grep for `fetch(` or `axios(`)
- CMS queries (use grep for `contentful`, `strapi`, etc.)
- Hardcoded constants (use grep for specific values)
- Configuration values (use grep in config files)
- Comments and documentation (use grep)

**For these cases**: Use grep to find files, then read to inspect.

## Advanced Tips

### List Available Packages
```bash
codenav query --graph INDEX --output json --limit 50 | jq -r '.[].package' | sort -u
```

### Find Functions in Specific File
```bash
codenav query --graph INDEX --file "*targetFile.ts" --output table
```

### Extract Subgraph
```bash
codenav extract --graph INDEX --from "rootFunction" --depth 2 --output /tmp/subgraph.bin
```

### Export for Visualization
```bash
codenav export --graph INDEX --output graph.dot --format dot
```

## Quick Reference Table

| Task | Tool | Command |
|------|------|---------|
| Find function | codenav | `query --name "pattern"` |
| Who calls this? | codenav | `callers functionName` |
| What does this call? | codenav | `trace --from "func"` |
| Call path A→B | codenav | `path --from A --to B` |
| Find text in files | grep | `pattern: "text"` |
| Read file content | read | `file_path: /path` |
| Function details | codenav | `query --name "func" --output json` |
| Package functions | codenav | `query --package "pkg"` |

## Workflow Summary for $ARGUMENTS

If investigating "$ARGUMENTS":

1. **Start**: `codenav query --graph INDEX --name "*$ARGUMENTS*" --output table`
2. **Trace Back**: `codenav callers --graph INDEX relevantFunction --show-lines`
3. **Trace Forward**: `codenav trace --graph INDEX --from "relevantFunction" --depth 2`
4. **Find Text** (if needed): `Grep pattern: "$ARGUMENTS"`
5. **Read Files** (last resort): Only read files identified by steps 1-4

## Remember

🎯 **Codenav = Navigation (call graphs, dependencies, architecture)**
🔍 **Grep = Finding (text, strings, constants)**
📖 **Read = Content (implementation details)**

**Always navigate with codenav first, search with grep second, read last.**

This order saves thousands of tokens and provides better context faster.
