# System Requirements Document (SRD): Embedded Full-Text Search Engine

## 1. Executive Summary

This document defines the functional requirements, constraints, and architecture for a localized, high-performance full-text search capability built natively within our Go microservice ecosystem.

The objective is to index a small, distinct catalog of documents—each characterized by a unique identifier, title/name, descriptive narrative, and collection of keyword tags. Searching must yield highly relevant, deterministic results without bleeding into partial or unrelated records. Given the bounded scale of the corpus, the architecture prioritizes speed, resource efficiency, dynamic field prioritization (boosting), and low infrastructure complexity by employing an embedded, in-memory, and localized file-system cache architecture via the Bleve library.

## 2. Requirements

### 2.1 Functional Requirements

- **FR-1: Field-Level Relevance & Weights:** The engine must search across three distinct document fields (`Name`, `Tags`, and `Description`) simultaneously. Relevancy calculation must prioritize `Name` and `Tags` significantly higher than matches found within the general `Description`.
    
- **FR-2: Dynamic Slice / Array Indexing:** The engine must natively index slices of strings for the `Tags` attribute via struct reflection, treating individual array items as distinct tokens searchable under a single index field identity.
    
- **FR-3: Zero-Result Enforcement for Weak Hits:** The engine must suppress low-relevance results. Searches that yield no direct or meaningful matches must return an empty result payload rather than falling back to weak, incidental keyword matches.
    
- **FR-4: Full-Catalog Refresh Capabilities:** The application must support periodic, comprehensive catalog refreshes. New indices must be generated to ensure that documents manually deleted from the source of truth do not persist in the active search runtime.
    
- **FR-5: Querying During Rebuilds (Zero Downtime):** Search execution must remain available and non-blocking to consumers during background catalog ingestion or index rebuild tasks.
    

### 2.2 Non-Functional Requirements

- **NFR-1: Zero External Dependencies:** The implementation must run completely embedded within the Go application process space. It must not rely on external search infrastructure or network-isolated daemons (e.g., Elasticsearch, OpenSearch).
    
- **NFR-2: Low Latency Bounds:** Query execution times must remain within low single-digit milliseconds given the small corpus size.
    
- **NFR-3: Resilient Ephemeral Storage:** The index must leverage the host operating system's temporary/cache directory. It must seamlessly handle reboots or cache-clearing events by regenerating data on demand, while leveraging an existing on-disk cache on immediate application launch.
    

## 3. Implementation Notes

### 3.1 Core Architecture & Library Choice

The system utilizes **Bleve** (`github.com/blevesearch/bleve/v2`) due to its long-term stability, robust maintenance lifecycle, native support for in-memory mapping (`bleve.NewMemOnly`), and seamless reflection-based struct indexing.

### 3.2 Document Mapping Design

Documents are mapped directly to Bleve using struct structures or standard map bindings. Bleve's reflection mapping engine automatically extracts slices of strings into multi-valued token structures under the matching field identifier:

Go

```
// Document Representation
type Item struct {
	ID          string   `json:"id"`
	Name        string   `json:"name"`
	Description string   `json:"description"`
	Tags        []string `json:"tags"`
}

// Ingestion is a simple single-line execution passing the struct directly
err := index.Index(item.ID, item)
```

### 3.3 Atomic Swap Strategy (A/B Partitioning)

To fulfill **FR-4** (stale document eviction) and **FR-5** (zero downtime during index generation) over a disk cache directory, an **A/B Blue-Green Directory Partitioning** pattern must be applied.

1. Maintain two subdirectories inside the temporary storage path: `index_a` and `index_b`.
    
2. The runtime engine tracks which index path is currently bound to the active search index instance via a pointer guarded by a mutex or atomic reference.
    
3. When a refresh cycle triggers, the background process identifies the _inactive_ path, completely removes any old data (`os.RemoveAll`), opens a fresh index via `bleve.New(path, mapping)`, and streams the full catalog into it.
    
4. Once completed, the process safely performs a synchronized pointer swap to route new incoming traffic to the fresh index instance, and gracefully invokes `.Close()` on the stale index context.
    

### 3.4 Multi-Tier Search Precision Control

To strictly address **FR-3** (preventing low-quality matches from polluting the result set), a two-phase screening gate must be applied during query construction and output iteration.

#### Tier 1: Clause Matching Floor (`NewDisjunctionQueryMin`)

Queries must break apart multi-word user strings and create multi-field `MatchQuery` tokens combined under a disjunction block. Enforce precise match semantics by specifying the minimum required matching queries directly at instantiation:

Go

```
// Require that at least 'N' distinct field sub-clauses are satisfied to qualify
disjunctionQuery := bleve.NewDisjunctionQueryMin([]query.Query{nameQuery, tagsQuery, descQuery}, 2)
```

#### Tier 2: Relevance Score Ceiling Gate

Bleve structures result slices (`searchResult.Hits`) in descending order of their calculated relevance score (strongest match first). To enforce strict precision:

1. Define an acceptable application score threshold floor (e.g., `minimumScoreFloor = 0.5`). This value should be confirmed via target integration profiling.
    
2. During iteration over the `Hits` slice, evaluate `hit.Score`.
    
3. The moment a result drops below the threshold value, execute an early termination (`break`) on the loop. Because of Bleve's descending sort order guarantee, discarding a score instantly guarantees all subsequent results fail validation, reducing unnecessary resource spend.
    

Go

```
searchResult, _ := index.Search(searchRequest)

for _, hit := range searchResult.Hits {
    if hit.Score < minimumScoreFloor {
        break // Stop processing safely; all subsequent records are weaker matches
    }
    // Process matching document ...
}
```