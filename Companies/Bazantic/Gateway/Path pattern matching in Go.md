# Path pattern matching in Go

Using standard library `http.ServeMux` *just* for routing is a solid, pragmatic fallback, but it carries a decent amount of allocation overhead because it's designed to manage the full lifecycle of HTTP handling, concurrency, and request mutation.

If you want a lightweight, ultra-fast alternative without dragging in a massive OpenAPI ecosystem or an entire web framework (like Gin or Echo), a **Radix Tree** (or Trie) is the gold standard for URL path matching.

The most popular, battle-tested library for this in the Go ecosystem is **`julienschmidt/httprouter`**. It uses a highly optimized Radix tree, allocates almost zero memory during lookups, and is incredibly fast.

Here is how you can use a pure Radix tree matcher for *just* matching patterns, without actually spinning up a server or wrapping your logic in standard HTTP handlers.

---

### The Minimalist Solution: Using `httprouter` for Lookups

First, grab the lightweight router:

```bash
go get github.com/julienschmidt/httprouter

```

Here is a clean, minimal implementation showing how to build the tree once and perform blazing-fast lookups on incoming request paths.

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/julienschmidt/httprouter"
)

// PathMatcher wraps the radix tree for clean path lookups.
type PathMatcher struct {
	router *httprouter.Router
}

// NewPathMatcher initializes the matcher with a list of Go-style patterns.
func NewPathMatcher(patterns []string) *PathMatcher {
	r := httprouter.New()
	
	// httprouter requires a Handle for routes, so we register a dummy one.
	// We only care if a match happens, not what the handler does.
	dummyHandle := func(w http.ResponseWriter, r *http.Request, p httprouter.Params) {}

	for _, pattern := range patterns {
		// httprouter uses `:param` instead of Go 1.22's `{param}`.
		// If your inputs use `{param}`, you can string-replace them here.
		r.GET(pattern, dummyHandle)
	}

	return &PathMatcher{router: r}
}

// Match checks if a path matches a registered pattern.
// It returns the matched pattern and any extracted parameters.
func (pm *PathMatcher) Match(path string) (string, httprouter.Params, bool) {
	// httprouter's Lookup method is extremely fast and avoids HTTP allocations
	handle, params, _ := pm.router.Lookup("GET", path)
	
	if handle != nil {
		// To get the original pattern string back, we can reconstruct or map it.
		// For a pure boolean check or param extraction, `handle != nil` is enough.
		return path, params, true
	}
	
	return "", nil, false
}

func main() {
	// Define your Go-style URL path patterns
	patterns := []string{
		"/api/v1/users",
		"/api/v1/users/:id",       // standard radix format
		"/api/v1/posts/:id/comments",
	}

	matcher := NewPathMatcher(patterns)

	// Test a couple of incoming request paths
	requests := []string{
		"/api/v1/users/42",
		"/api/v1/posts/101/comments",
		"/api/v1/unknown-route",
	}

	for _, reqPath := range requests {
		_, params, matched := matcher.Match(reqPath)
		if matched {
			fmt.Printf("✓ Match Found for: %s\n", reqPath)
			if len(params) > 0 {
				fmt.Printf("  Params: %+v\n", params)
			}
		} else {
			fmt.Printf("✗ No Match for:    %s\n", reqPath)
		}
	}
}

```

---

### Why this beats `http.ServeMux` for your use case:

* **Zero Allocation Lookups:** `httprouter.Lookup` travels down a tree structure by scanning the string slice directly. It doesn't create `http.Request` objects or build heavy context maps.
* **No Spec Baggage:** Unlike `kin-openapi`, it doesn't care about payloads, query parameters, headers, or validation components. It strictly evaluates the string path.
* **Explicit Parameter Extraction:** If your patterns contain variables (like `/users/:id`), the Radix tree parses out the values into a tiny slice of key-value pairs instantly, which `ServeMux` historically makes you jump through extra hoops to fetch cleanly.

> **Note on Syntax:** If your incoming patterns strictly use Go 1.22's wildcard syntax (e.g., `/users/{id}`), you can easily sanitize the string slice using `strings.Replacer` to convert `{` and `}` to `:` before feeding them into the router.

Since your patterns are stored in the database using the `{param}` syntax (Go 1.22 style), you can seamlessly normalize them right before registering them into the Radix tree.

A `strings.Replacer` is perfect for this—it's highly efficient and avoids heavy regex overhead.

Here is the updated, clean implementation that pulls the bracket-syntax strings, sanitizes them on the fly, and sets up your high-performance matcher.

Go

```
package main

import (
	"fmt"
	"net/http"
	"strings"

	"github.com/julienschmidt/httprouter"
)

type DBPathMatcher struct {
	router   *httprouter.Router
	replacer *strings.Replacer
}

func NewDBPathMatcher(dbPatterns []string) *DBPathMatcher {
	r := httprouter.New()
	
	// Pre-configure a replacer to convert "{param}" into ":param"
	// It strips the trailing '}' and converts the leading '{' to ':'
	replacer := strings.NewReplacer("{", ":", "}", "")
	
	dummyHandle := func(w http.ResponseWriter, req *http.Request, p httprouter.Params) {}

	for _, dbPattern := range dbPatterns {
		// Convert "/api/v1/users/{id}" -> "/api/v1/users/:id"
		normalizedPattern := replacer.Replace(dbPattern)
		
		r.GET(normalizedPattern, dummyHandle)
	}

	return &DBPathMatcher{
		router:   r,
		replacer: replacer,
	}
}

// Match performs a zero-allocation Radix tree lookup
func (m *DBPathMatcher) Match(path string) (httprouter.Params, bool) {
	handle, params, _ := m.router.Lookup("GET", path)
	return params, handle != nil
}

func main() {
	// Simulated data coming straight from your PostgreSQL rows
	patternsFromDB := []string{
		"/api/v1/users",
		"/api/v1/users/{id}",
		"/api/v1/posts/{post_id}/comments/{comment_id}",
	}

	matcher := NewDBPathMatcher(patternsFromDB)

	// Test case: Live incoming request path
	incomingPath := "/api/v1/posts/101/comments/555"

	params, matched := matcher.Match(incomingPath)
	if matched {
		fmt.Printf("✓ Success: Matched path safely!\n")
		for _, p := range params {
			fmt.Printf("  Extracted -> %s: %s\n", p.Key, p.Value)
		}
	} else {
		fmt.Printf("✗ No match found.\n")
	}
}
```

### Why this works perfectly with your DB:

- **One-Time Cost:** The `strings.Replacer` normalization only happens once during your application startup (or when you refresh your DB cache).
    
- **Lookup Performance:** The actual incoming request routing (`matcher.Match`) remains entirely untouched and lightning-fast, keeping your HTTP hot-path free of memory allocations.

The reason `httprouter` uses that `:param` syntax comes down to a classic engineering trade-off: **extreme performance optimization vs. developer ergonomics.**

To understand why, we have to look at when it was written and how a Radix tree works under the hood.

### 1. It's a Radix Tree, Not a Regex Engine

Most modern, flexible routers (like Go's new 1.22 `ServeMux`) parse patterns into complex token arrays or use regular expressions behind the scenes to handle the `{param}` brackets.

`httprouter`, however, is a strict **Radix Tree** (a space-optimized trie). It literally walks the incoming URL string character-by-character.

When building the tree:

- A static path segment is just a string of characters (e.g., `/users/`).
    
- The moment it hits a `:` character, the internal logic instantly knows, _"Ah, everything from this byte until the next `/` is a wildcard variable name."_
    

Using single-character markers like `:` or `*` allows the internal tree-building loop to be incredibly simple and fast, requiring almost zero string splitting or parsing logic during a request lookup.

### 2. Historical Context (The Sinatra Legacy)

When `httprouter` was created over a decade ago (around 2013), Go's standard library router was incredibly primitive—it only supported exact matches or trailing slashes.

At that time, the reigning king of web framework design was Ruby's **Sinatra** (and later Node's **Express**). Both of those massively influential frameworks used the `:param` syntax for variables. The author of `httprouter` adopted it because it was the industry standard for web developers at the time.

Go's `{param}` syntax didn't become standard library convention until Go 1.22 was released in early 2024.

### Summary: The Performance Payoff

By keeping the syntax rigid and "arcane," `httprouter` achieves its legendary status:

- **Zero dynamic memory allocations** (`0 allocs/op`) during routing lookups.
    
- **Nanosecond-level speeds** that beat almost every other router in the ecosystem.
    

It forces you to do a tiny bit of string manipulation upfront so that your live server can handle millions of requests a second without breaking a sweat.