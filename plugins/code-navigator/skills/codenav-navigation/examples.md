# Codenav Real-World Examples

## Example 1: Finding Data Source

**Task**: Find where a specific data field "userProfile" comes from - is it from API, database, or cached?

### Step-by-Step Solution

```bash
# Step 1: Find profile-related functions
$ codenav query --graph /tmp/codebase.bin --name "*profile*" --output table

Name                                     Type            Package                        Line
-----------------------------------------------------------------------------------------------
fetchUserProfile                         Function        api                            25
cacheUserProfile                         Function        cache                          18
transformProfile                         Function        transformers                   10

# Step 2: Get details about the transformer
$ codenav query --graph /tmp/codebase.bin --name "transformProfile" --output json

[{
  "id": "./src/api/profile.ts:transformProfile:10",
  "name": "transformProfile",
  "type": "function",
  "file_path": "./src/api/profile.ts",
  "line": 10,
  "end_line": 44,
  "package": "transformers",
  "parameters": [{"name": "rawProfile", "param_type": "RawProfile"}]
}]

# Step 3: Find who calls this transformer
$ codenav callers --graph /tmp/codebase.bin transformProfile --show-lines

Callers of transformProfile
├─ fetchUserProfile (line 57)
→ 1 callers found

# Step 4: Continue tracing backwards
$ codenav callers --graph /tmp/codebase.bin fetchUserProfile --show-lines

Callers of fetchUserProfile
├─ useUserProfile (line 15)
├─ ProfilePage (line 33)
├─ UserDashboard (line 22)
→ 3 callers found

# Step 5: Trace the full dependency tree
$ codenav trace --graph /tmp/codebase.bin --from "fetchUserProfile" --depth 2 --show-lines

Dependencies of fetchUserProfile
├─ fetch (line 28)
│  └─ getApiUrl (line 5)
├─ transformProfile (line 35)
│  ├─ formatDate (line 15)
│  └─ sanitizeData (line 20)
└─ cacheUserProfile (line 40)

# Step 6: NOW use grep to find the API endpoint
$ grep -r "/api/users" /path/to/src --files-with-matches

src/api/profile.ts
src/config/endpoints.ts

# Step 7: Read only the identified files
$ read src/api/profile.ts

# Found: const response = await fetch(`${baseUrl}/api/users/${userId}/profile`)
```

**Conclusion**: The "userProfile" data comes from **API endpoint** `/api/users/{userId}/profile`, transformed by `transformProfile`, and cached by `cacheUserProfile`.

**Token Savings**: Used codenav to navigate the call chain (6 commands) instead of blindly reading 15+ files. Saved ~12,000 tokens.

---

## Example 2: Understanding Authentication Flow

**Task**: Understand how user authentication works in the app.

```bash
# Step 1: Find auth-related functions
$ codenav query --graph /tmp/codebase.bin --name "*auth*" --type function --limit 20

Name                                     Type            Package
-------------------------------------------------------------------------
authenticate                             Function        auth
validateAuth                             Function        auth
useAuth                                  Function        hooks

# Step 2: Find who uses the auth hook
$ codenav callers --graph /tmp/codebase.bin useAuth --show-lines

Callers of useAuth
├─ LoginPage (line 12)
├─ ProtectedRoute (line 8)
├─ UserProfile (line 15)
└─ NavBar (line 22)
→ 4 callers found

# Step 3: Trace what authenticate does
$ codenav trace --graph /tmp/codebase.bin --from "authenticate" --depth 3 --show-lines

Dependencies of authenticate
├─ validateCredentials (line 45)
│  ├─ hashPassword (line 12)
│  └─ compareHashes (line 18)
├─ createSession (line 50)
│  ├─ generateToken (line 8)
│  └─ storeSession (line 15)
└─ fetchUserData (line 55)
   └─ fetch (line 22)

# Step 4: Find the path from login to session storage
$ codenav path --graph /tmp/codebase.bin --from "LoginPage" --to "storeSession"

Path from LoginPage to storeSession
LoginPage → useAuth → authenticate → createSession → storeSession

# Step 5: Read only the key files identified
$ read src/auth/authenticate.ts
$ read src/hooks/useAuth.ts
```

**Result**: Complete understanding of auth flow in 5 codenav commands + 2 reads instead of 15+ blind reads.

---

## Example 3: Finding API Endpoint

**Task**: Find which API endpoint is called when loading product data.

```bash
# Step 1: Find product-related functions
$ codenav query --graph /tmp/codebase.bin --name "*product*" --type function --limit 20

Name                                     Type            Package
-------------------------------------------------------------------------
useProducts                              Function        hooks
fetchProductData                         Function        api
transformProduct                         Function        transformers

# Step 2: Trace from UI hook to API
$ codenav trace --graph /tmp/codebase.bin --from "useProducts" --depth 3

Dependencies of useProducts
├─ useQuery (line 8)
├─ fetchProductData (line 12)
│  ├─ fetch (line 15)
│  │  └─ getApiConfig (line 18)
│  └─ transformResponse (line 20)
└─ transformProduct (line 14)

# Step 3: Get the fetch function details
$ codenav query --graph /tmp/codebase.bin --name "fetchProductData" --output json

# Shows file path: ./src/api/products.ts

# Step 4: Read only that file to see the endpoint
$ read ./src/api/products.ts

# Found: fetch(`/api/v1/products`, ...)
```

**Result**: Found API endpoint in 4 commands instead of searching through dozens of files.

---

## Example 4: Finding Unused Code

**Task**: Find functions that are never called (potential dead code).

```bash
# Step 1: Query all functions in a module
$ codenav query --graph /tmp/codebase.bin --package "utils" --output json > /tmp/utils-funcs.json

# Step 2: Check callers for each function
$ for func in $(jq -r '.[].name' /tmp/utils-funcs.json); do
  count=$(codenav callers --graph /tmp/codebase.bin "$func" --count)
  if [ "$count" = "0" ]; then
    echo "UNUSED: $func"
  fi
done

UNUSED: formatLegacyDate
UNUSED: deprecatedHelper
UNUSED: oldValidator

# Step 3: Read these files to confirm they can be removed
$ read src/utils/formatLegacyDate.ts
```

**Result**: Identified dead code without manual inspection of every file.

---

## Example 5: Understanding Error Propagation

**Task**: Understand how errors are caught and displayed to users.

```bash
# Step 1: Find error handling functions
$ codenav query --graph /tmp/codebase.bin --name "*error*" --type function

Name                                     Type            Package
-------------------------------------------------------------------------
handleError                              Function        errorHandler
logError                                 Function        logger
showErrorToast                           Function        ui

# Step 2: Find all functions that call error handler
$ codenav callers --graph /tmp/codebase.bin handleError --show-lines

Callers of handleError
├─ apiClient (line 88)
├─ asyncHandler (line 12)
├─ formSubmit (line 45)
└─ globalErrorBoundary (line 22)
→ 4 callers found

# Step 3: Trace what happens when error is handled
$ codenav trace --graph /tmp/codebase.bin --from "handleError" --depth 2

Dependencies of handleError
├─ logError (line 15)
│  ├─ console.error (line 8)
│  └─ sendToMonitoring (line 12)
├─ showErrorToast (line 20)
│  └─ toast (line 5)
└─ cleanupResources (line 25)

# Step 4: Find path from API call to UI error display
$ codenav path --graph /tmp/codebase.bin --from "fetchData" --to "showErrorToast" --limit 3

Path 1: fetchData → handleError → showErrorToast
Path 2: fetchData → asyncHandler → handleError → showErrorToast
Path 3: fetchData → apiClient → handleError → showErrorToast
```

**Result**: Complete error flow understanding with visual paths.

---

## Example 6: Refactoring Impact Analysis

**Task**: Before refactoring `calculatePrice`, understand its impact.

```bash
# Step 1: Find all callers
$ codenav callers --graph /tmp/codebase.bin calculatePrice --show-lines --output json > /tmp/callers.json

# Step 2: Count total usage
$ jq 'length' /tmp/callers.json
15

# Step 3: See caller details
$ jq -r '.[] | "\(.name) at \(.file_path):\(.line)"' /tmp/callers.json

ShoppingCart at src/cart/ShoppingCart.tsx:45
Checkout at src/checkout/Checkout.tsx:88
OrderSummary at src/orders/OrderSummary.tsx:23
...

# Step 4: Analyze complexity impact
$ codenav analyze --graph /tmp/codebase.bin complexity --limit 20

# Step 5: Check if any callers are in critical paths
$ for caller in $(jq -r '.[].name' /tmp/callers.json); do
  echo "=== $caller ==="
  codenav callers --graph /tmp/codebase.bin "$caller" --count
done
```

**Result**: Know exactly which 15 files will be affected by the refactor.

---

## Example 7: Finding Configuration Source

**Task**: Find where `MAX_RETRY_ATTEMPTS` configuration is defined and used.

```bash
# Step 1: Codenav can't find constants, use grep
$ grep -r "MAX_RETRY_ATTEMPTS" /path/to/src --files-with-matches

src/config/constants.ts
src/api/retryHandler.ts
src/services/httpClient.ts

# Step 2: Read the config file
$ read src/config/constants.ts

# Found: export const MAX_RETRY_ATTEMPTS = 3;

# Step 3: Find functions that use retry logic with codenav
$ codenav query --graph /tmp/codebase.bin --name "*retry*" --type function

Name                                     Type            Package
-------------------------------------------------------------------------
retryHandler                             Function        api
withRetry                                Function        utils

# Step 4: Trace retry handler dependencies
$ codenav trace --graph /tmp/codebase.bin --from "retryHandler" --depth 2

Dependencies of retryHandler
├─ sleep (line 12)
├─ exponentialBackoff (line 15)
└─ shouldRetry (line 20)

# Step 5: Find all callers of retry logic
$ codenav callers --graph /tmp/codebase.bin retryHandler --show-lines
```

**Result**: Combined grep (for constants) and codenav (for logic flow) efficiently.

---

## Example 8: Understanding State Management Flow

**Task**: Trace how state updates propagate through the application.

```bash
# Step 1: Find state management hooks
$ codenav query --graph /tmp/codebase.bin --name "useState*" --type function

Name                                     Type            Package
-------------------------------------------------------------------------
useStateManager                          Function        hooks
useStateSync                             Function        hooks

# Step 2: Find what uses the state manager
$ codenav callers --graph /tmp/codebase.bin useStateManager --show-lines

Callers of useStateManager
├─ AppContext (line 10)
├─ Dashboard (line 25)
└─ Settings (line 18)
→ 3 callers found

# Step 3: Trace state update path
$ codenav path --graph /tmp/codebase.bin --from "updateSettings" --to "renderUI"

Path: updateSettings → setState → useStateManager → Dashboard → renderUI

# Step 4: Read the state management implementation
$ read src/hooks/useStateManager.ts
```

**Result**: Understand the complete state flow in 4 commands.

---

## Key Takeaways from Examples

1. **Start with codenav query** to find relevant functions
2. **Use callers** to trace backwards (where is this used?)
3. **Use trace** to go forwards (what does this do?)
4. **Use path** to understand connections between components
5. **Use grep** only for finding text/constants that codenav can't index
6. **Read files** only after identifying exactly what you need
7. **Combine tools** strategically: codenav for structure, grep for content, read for details

## Token Efficiency Comparison

| Approach | Commands | Files Read | Tokens Used |
|----------|----------|------------|-------------|
| **Blind Reading** | grep → read 20+ files | 20+ | ~40,000 |
| **Codenav First** | 5 codenav + 2 reads | 2 | ~3,000 |
| **Savings** | - | 90% fewer | **92% savings** |

Always navigate with codenav before reading files!
