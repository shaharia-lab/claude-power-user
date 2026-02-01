# Codenav Complete Reference

## Installation & Setup

Codenav requires an indexed codebase. Index files are typically generated separately and provided via the `--graph` parameter.

```bash
# Index a codebase (if needed)
codenav index /path/to/codebase --output /tmp/codebase.bin

# Use the index
codenav query --graph /tmp/codebase.bin --name "*pattern*"
```

## Complete Command Reference

### Index Command
```bash
codenav index [OPTIONS] <PATH>

Options:
  --output <OUTPUT>      Output file path [default: codenav.bin]
  --exclude <PATTERN>    Exclude paths matching pattern
  --language <LANG>      Target language (auto-detected if not specified)
  -v, --verbose          Verbose output
```

### Query Command
```bash
codenav query [OPTIONS]

Options:
  -g, --graph <GRAPH>      Graph file [default: codenav.bin]
  -o, --output <OUTPUT>    Output format: table, json, tree [default: table]
  -c, --count              Show count only
  --limit <LIMIT>          Limit results
  --name <NAME>            Filter by name (supports wildcards)
  --type <TYPE>            Filter by type: function, method, handler
  --package <PACKAGE>      Filter by package
  --file <FILE>            Filter by file path
  --tag <TAG>              Filter by tag
```

**Examples:**
```bash
# Find all functions
codenav query --graph /tmp/codebase.bin --type function --limit 100

# Find functions in a package
codenav query --graph /tmp/codebase.bin --package "utils" --output table

# Count total nodes
codenav query --graph /tmp/codebase.bin --count

# JSON output for parsing
codenav query --graph /tmp/codebase.bin --name "auth*" --output json | jq .
```

### Callers Command
```bash
codenav callers [OPTIONS] <FUNCTION>

Arguments:
  <FUNCTION>  Function or method name

Options:
  -g, --graph <GRAPH>    Graph file [default: codenav.bin]
  -c, --count            Show count only
  -o, --output <OUTPUT>  Output format: tree, json, table [default: tree]
  --show-lines           Show line numbers
```

**Examples:**
```bash
# Find all callers with line numbers
codenav callers --graph /tmp/codebase.bin myFunction --show-lines

# Get caller count
codenav callers --graph /tmp/codebase.bin myFunction --count

# JSON output
codenav callers --graph /tmp/codebase.bin myFunction --output json
```

### Trace Command
```bash
codenav trace [OPTIONS] --from <FROM>

Options:
  -g, --graph <GRAPH>    Graph file [default: codenav.bin]
  --from <FROM>          Function or method name to trace from
  -d, --depth <DEPTH>    Traversal depth (default: 1) [default: 1]
  -o, --output <OUTPUT>  Output format: tree, json, dot [default: tree]
  --show-lines           Show line numbers
  -f, --filter <FILTER>  Filter by pattern
```

**Examples:**
```bash
# Trace immediate dependencies
codenav trace --graph /tmp/codebase.bin --from "processData"

# Deep trace with line numbers
codenav trace --graph /tmp/codebase.bin --from "handleRequest" --depth 3 --show-lines

# Filter results
codenav trace --graph /tmp/codebase.bin --from "main" --depth 2 --filter "*service*"

# Output as DOT for visualization
codenav trace --graph /tmp/codebase.bin --from "main" --output dot > trace.dot
```

### Path Command
```bash
codenav path [OPTIONS] --from <FROM> --to <TO>

Options:
  -g, --graph <GRAPH>          Graph file [default: codenav.bin]
  --from <FROM>                Starting function
  --to <TO>                    Target function
  -l, --limit <LIMIT>          Find multiple paths (specify number)
  --all                        Find all possible paths (may be slow)
  --max-depth <MAX_DEPTH>      Maximum search depth [default: 10]
  -o, --output <OUTPUT>        Output format: tree, json [default: tree]
```

**Examples:**
```bash
# Find shortest path
codenav path --graph /tmp/codebase.bin --from "main" --to "errorHandler"

# Find multiple paths
codenav path --graph /tmp/codebase.bin --from "api" --to "database" --limit 5

# Set max depth
codenav path --graph /tmp/codebase.bin --from "ui" --to "backend" --max-depth 15
```

### Analyze Command
```bash
codenav analyze [OPTIONS] <ANALYSIS_TYPE>

Arguments:
  <ANALYSIS_TYPE>  Analysis type: complexity, coupling, hotspots, circular

Options:
  -g, --graph <GRAPH>          Graph file [default: codenav.bin]
  --threshold <THRESHOLD>      Threshold for reporting
  --limit <LIMIT>              Limit results
  -o, --output <OUTPUT>        Output format: table, json [default: table]
```

**Examples:**
```bash
# Find most complex functions
codenav analyze --graph /tmp/codebase.bin complexity --limit 20

# Find circular dependencies
codenav analyze --graph /tmp/codebase.bin circular

# Find hotspots (most called functions)
codenav analyze --graph /tmp/codebase.bin hotspots --limit 10

# Find high coupling
codenav analyze --graph /tmp/codebase.bin coupling --threshold 5
```

### Extract Command
```bash
codenav extract [OPTIONS] --from <FROM> --output <OUTPUT>

Options:
  -g, --graph <GRAPH>    Graph file [default: codenav.bin]
  --from <FROM>          Starting node (function/method name)
  -d, --depth <DEPTH>    Traversal depth (default: 2) [default: 2]
  -o, --output <OUTPUT>  Output file
```

**Examples:**
```bash
# Extract subgraph around a function
codenav extract --graph /tmp/codebase.bin --from "authModule" --depth 3 --output /tmp/auth.bin

# Then query the subgraph
codenav query --graph /tmp/auth.bin --output table
```

### Export Command
```bash
codenav export [OPTIONS] --output <OUTPUT> --format <FORMAT>

Options:
  -g, --graph <GRAPH>    Graph file [default: codenav.bin]
  -o, --output <OUTPUT>  Output file
  -f, --format <FORMAT>  Format: graphml, dot, csv
  --filter <FILTER>      Filter nodes (format: package:NAME or type:TYPE)
  --exclude-tests        Exclude test files
```

**Examples:**
```bash
# Export as DOT for Graphviz
codenav export --graph /tmp/codebase.bin --output graph.dot --format dot

# Export as GraphML for analysis tools
codenav export --graph /tmp/codebase.bin --output graph.graphml --format graphml

# Export filtered graph
codenav export --graph /tmp/codebase.bin --output api.csv --format csv --filter package:api

# Exclude tests
codenav export --graph /tmp/codebase.bin --output prod.dot --format dot --exclude-tests
```

## Output Format Examples

### Table Output
```
Name                                     Type            Package                        Line
-----------------------------------------------------------------------------------------------
dataHandler                              Function        api                            10
dataTransformer                          Function        transformers                   7
```

### JSON Output
```json
[
  {
    "id": "./path/to/file.ts:functionName:10",
    "name": "functionName",
    "type": "function",
    "file_path": "./path/to/file.ts",
    "line": 10,
    "end_line": 44,
    "package": "api",
    "signature": "(",
    "parameters": [
      {
        "name": "param1",
        "param_type": "Type[]"
      }
    ],
    "returns": [],
    "tags": [],
    "metadata": {}
  }
]
```

### Tree Output
```
Dependencies of functionName

├─ childFunction1 (./path/to/file.ts:35)
│  ├─ grandchildFunction1 (./path/to/other.ts:12)
│  └─ grandchildFunction2 (./path/to/other.ts:45)
├─ childFunction2 (./path/to/file.ts:67)

→ 4 dependencies found
```

## Wildcard Patterns

Codenav supports wildcards in name and file filters:

- `*` - Match any characters
- `?` - Match single character
- `[abc]` - Match any character in brackets

**Examples:**
```bash
# Find all "use" hooks (React)
codenav query --graph INDEX --name "use*" --type function

# Find all test files
codenav query --graph INDEX --file "*.spec.ts"

# Find functions starting with get/set
codenav query --graph INDEX --name "[gs]et*"

# Find authentication related
codenav query --graph INDEX --name "*auth*"
```

## Performance Tips

1. **Use --count first** to gauge result size before full output
   ```bash
   codenav query --graph INDEX --name "use*" --count
   ```

2. **Limit results** when exploring
   ```bash
   codenav query --graph INDEX --name "*" --limit 20
   ```

3. **Use JSON output** for programmatic parsing
   ```bash
   codenav query --graph INDEX --name "auth*" --output json | jq '.[] | .file_path'
   ```

4. **Extract subgraphs** for focused analysis
   ```bash
   codenav extract --graph INDEX --from "module" --depth 2 --output /tmp/module.bin
   ```

5. **Filter early** with specific queries
   ```bash
   # Good: Specific query
   codenav query --graph INDEX --package "api" --name "*user*"

   # Bad: Too broad
   codenav query --graph INDEX --name "*"
   ```

## Integration with Grep and Read

### Pattern 1: Codenav → Read
```bash
# Find function location with codenav
codenav query --graph INDEX --name "authenticate" --output json

# Extract file path
FILE=$(codenav query --graph INDEX --name "authenticate" --output json | jq -r '.[0].file_path')

# Read only that file
read "$FILE"
```

### Pattern 2: Codenav → Grep → Read
```bash
# Find function with codenav
codenav query --graph INDEX --name "*config*"

# Use grep to find specific content in those files
grep -l "API_KEY" $(codenav query --graph INDEX --name "*config*" --output json | jq -r '.[].file_path')

# Read only matching files
read /path/to/matched/file.ts
```

### Pattern 3: Trace then Read
```bash
# Trace dependencies
codenav trace --graph INDEX --from "main" --depth 2 --output json

# Read each file in the trace
for file in $(codenav trace --graph INDEX --from "main" --output json | jq -r '.[] | .file_path' | sort -u); do
  echo "=== $file ==="
  read "$file"
done
```

## Troubleshooting

### No Results Found
```bash
# Check if index is valid
codenav query --graph /tmp/codebase.bin --count

# Verify function exists (case-sensitive)
codenav query --graph /tmp/codebase.bin --name "*pattern*"

# Try broader search
codenav query --graph /tmp/codebase.bin --name "*"
```

### Index Not Found
```bash
# Verify file exists
ls -lh /tmp/codebase.bin

# Check permissions
file /tmp/codebase.bin
```

### Slow Queries
```bash
# Use --count to check size first
codenav query --graph /tmp/codebase.bin --count

# Limit results
codenav query --graph /tmp/codebase.bin --limit 100

# Extract subgraph for focused work
codenav extract --graph /tmp/codebase.bin --from "module" --output /tmp/small.bin
```

## Best Practices

1. ✅ Always start with `codenav query` to find functions
2. ✅ Use `callers` and `trace` to understand relationships
3. ✅ Limit depth on `trace` to avoid overwhelming output
4. ✅ Use `--show-lines` to get precise locations
5. ✅ Export JSON for programmatic processing
6. ✅ Extract subgraphs for focused analysis
7. ❌ Don't trace too deep without limits
8. ❌ Don't query all functions without filtering
9. ❌ Don't skip codenav and jump to grep/read

## Summary

Codenav is your **navigation tool**. Use it to understand:
- 🗺️ Code structure and organization
- 🔗 Function relationships and dependencies
- 📞 Call graphs and paths
- 🎯 Where to focus your attention

Then use:
- 🔍 **Grep** to find specific text/strings
- 📖 **Read** to view actual implementation

This workflow saves tokens and provides better understanding faster.
