---
name: condense
description: Structured, information-dense output format with coordinate addressing, symbolic notation, and parenthetical markers. Use when the user wants structured output, numbered responses, addressable messages, dense/concise formatting, coordinate-based referencing, compressed communication, or anything resembling a log, plan, spec, or structured knowledge format. Trigger on "dense mode".
---

# Accurate, Concise, Indexed

Every response is structured output: indexed by coordinates, compressed via telegraphic style, referenceable by address. No prose paragraphs. No filler. Valid markdown only.

Every response has at minimum a message heading and summary. Sections, subsections, and lines scale with content complexity.

## Writing Discipline

**Telegraphic deletion.** Drop articles, copulas, fillers when meaning is preserved. "configure system -> new endpoint" not "configure the system to use the new endpoint".

**Nominalization.** Verb phrases -> noun compounds for concepts; verbal form for actions. "system load-performance analysis needed" not "we need to analyze how the system performs under load".

**Presupposition loading.** "given", "assuming", "with", "after" to embed preconditions inline.

**Parallel structure.** State pattern once, vary only delta. "validate: email, name, phone".

**Semicolon stacking.** Related facts on one line, separated by semicolons. Max 3 clauses per line; beyond 3, split.

**Result-first.** Lead with output/conclusion, then conditions.

**Consistency.** First term for a concept = canonical. No synonym-swapping. No filler phrases.

**Clarity over brevity.** If compressed form is ambiguous, expand.

**No em dashes or double hyphens.** Use semicolons, commas, or parentheses.

## Coordinates

Each response is a message with an auto-incrementing number starting at 1. User messages are unnumbered. Every message heading includes a concise noun phrase.

```
# 3 Cache Design
Redis-backed LRU cache; 300s TTL; invalidation on write

## 3.1 Storage Backend
1. Redis cluster; 3 shards; consistent hashing
2. fallback to local LRU on connection failure

## 3.2 Eviction Strategy
1. LRU with TTL floor; no entry lives past 300s

### 3.2.1 TTL Rules
1. read refreshes TTL; write resets to max
2. stale entries evicted lazily on next access

### 3.2.2 Memory Pressure
1. IF used > 80% THEN aggressive eviction [LFU]
2. IF used > 95% THEN reject new writes [503]
```

| Level | Format | Example | Meaning |
|-------|--------|---------|---------|
| Message | `# N Heading` | `# 3 Cache Design` | response 3 |
| Section | `## N.S Heading` | `## 3.2 Eviction` | message 3, section 2 |
| Subsection | `### N.S.Sub Heading` | `### 3.2.1 LRU` | message 3, section 2, subsection 1 |
| Line | `L.` | `5.` | line 5 in current section/subsection |

Full reference: `@N.S.Sub.L` (e.g., `@3.2.1.5`). Shorter forms valid: `@3` (message), `@3.2` (section), `@3.2.1` (subsection or line per context). Document structure resolves ambiguity.

Section and subsection numbers sequential from 1; never skip. Line numbers restart at 1 per section/subsection.

Content lines, code blocks, tables, and multiline constructs all get line numbers. Code blocks and tables get one number for the block. Numbering inside code blocks is content, not coordinates.

## Structure

### Summaries

Summaries are content, not meta-commentary. Single line. Heading names topic; summary states key point. No redundancy between heading and summary.

**Message summary (mandatory).** Line after message heading. For small messages, this IS the content.

Bad: "This message covers the analysis of the caching layer and proposed solutions"
Good: "cache miss rate 40% <- stale TTL config; fix: bump TTL, add invalidation hook"

**Section/subsection summary (recommended).** First line summarizes content. A reader of only summaries gets the essential structure.

### Scaling

Depth grows with complexity. Minimum response:

```
# 5 Cache Fix
cache invalidation bug; TTL not respected on writes
```

Sections appear when content has distinct aspects; subsections when a section needs breakdown; lines when multiple points exist. Single-point sections need only their summary.

```
# 5 Cache Fix
cache invalidation bug; TTL not respected on writes

## 5.1 Root Cause
1. TTL set on read path but skipped on write path
2. write handler bypasses cache layer entirely

## 5.2 Fix
1. add TTL propagation to write path
2. invalidate stale entries on write
```

### Blank-Line Chunking

Blank lines within a section separate logical groups. Line numbers continue across them.

## References

Reference earlier content by coordinate; do not restate.

```
1. auth flow same as @3.2.5; only token format differs
2. token refresh per @3.2.1.3
3. rate logic per @3.2.5 (sliding window config)
4. retry budget inherited from @4.1 (circuit breaker)
```

**Point.** `@N.S.L` or deeper targets a single line.

**Range.** Append `-L` for a span within same section/subsection: `@3.2.1-4`.

**Cross-reference gloss.** Different-section references get a parenthetical gloss; same-section uses bare coordinates.

**Supersede.** Heading includes `[supersedes @N.S]` when replacing prior content.

```
## 5.1 Config [supersedes @3.1]
1. + CACHE_TTL env var [default=300s]
2. - hardcoded timeout constant
3. ~ DB_POOL_SIZE 10 -> 25
```

## Expand

When a user asks to expand a coordinate, drill into that coordinate in-place. The message counter does not increment.

**Maximum 3 visible levels.** Expanded coordinate as `#`, direct children as `##`, grandchildren as `###`. Deeper content is not rendered; expandable via further requests.

**Coordinate depth grows with each expand.** Expanding `@1.4` produces `# 1.4`, `## 1.4.1`, `## 1.4.2`. Expanding `@1.4.1` produces `# 1.4.1`, `## 1.4.1.1`, `## 1.4.1.2`. Coordinates extend indefinitely.

Example; given `## 3.2 Eviction Strategy` from the Coordinates example above.

User: `expand @3.2`

```
# 3.2 Eviction Strategy
LRU with TTL floor; no entry lives past 300s

## 3.2.1 TTL Rules
read refreshes TTL; write resets to max

1. default TTL = 300s; configurable per cache region
2. stale entries evicted lazily on next access
3. bulk invalidation via pub/sub channel on write-through

## 3.2.2 Memory Pressure
graduated response based on heap usage

1. IF used > 80% THEN aggressive eviction [LFU]
2. IF used > 95% THEN reject new writes [503]

## 3.2.3 Monitoring
1. eviction rate tracked per shard; alert if > 500/s
2. #perf cache hit ratio target > 95%
```

User: `expand @3.2.1`

```
# 3.2.1 TTL Rules
read refreshes TTL; write resets to max

## 3.2.1.1 Default Behavior
1. GET refreshes TTL from current value; no server round-trip
2. PUT/DELETE resets TTL to region max

## 3.2.1.2 Region Overrides
1. session cache: TTL = 1800s; no refresh on read
2. api-response cache: TTL = 60s; refresh on read

## 3.2.1.3 Bulk Invalidation
1. write-through publishes to invalidation channel
2. all replicas evict matching keys within 50ms [confirmed]
```

## Notation

### Markers

Optional parenthetical prefixes; use sparingly. Set is open-ended.

| Marker | Meaning |
|--------|---------|
| `(IMPORTANT)` | critical, warning, risk |
| `(NOTE)` | key insight, highlight |
| `(QUESTION)` | open question, uncertainty |
| `(ACTION)` | imperative, next step |

### Symbols

| Symbol | Use |
|--------|-----|
| `->` | flow, transformation |
| `<-` | derivation, caused by |
| `<->` | bidirectional |
| `=>` | implies |
| `<=>` | iff |
| `in` | member of |
| `not-in` | not member of |
| `=` `!=` `~=` | equals, not equal, approximately |
| `>` `<` `>=` `<=` | comparison |
| `△` | delta, change |
| `[...]` | metadata, constraints, annotations |
| `{a, b, c}` | unordered set |
| `(a, b, c)` | ordered sequence |
| `a \| b \| c` | alternatives |
| `?` (suffix) | optional |
| `~N%` | approximate confidence/probability |

### Keywords

Uppercase for logic, quantification, branching.

**Connectives.** `AND`, `OR`. **Quantifiers.** `ALL`, `ANY`, `NONE`.

**Branching.** `IF`/`THEN`/`ELSE` for 1-2 branches; one line for simple, indent for nested. `MATCH`/`CASE` for 3+ branches. Decision tables for comparing multiple properties across 3+ cases.

### Confidence

Two forms; do not combine on the same line.

**Categorical.** `[confirmed]` verified; `[likely]` strong evidence; `[plausible]` reasonable inference; `[speculative]` low evidence.

**Numeric.** Inline `~N%` when probability adds signal: "cache hit rate ~95%".

### State Markers

| Marker | Meaning |
|--------|---------|
| `OPEN` | pending |
| `WIP` | in progress |
| `DONE` | resolved |
| `BLOCKED` | rejected, blocked |

### Scope Tags

`#tag` prefix sets domain for a line. Tags are words, never bare numbers. Multiple tags allowed. Hoist to section heading when all lines share scope.

```
1. #perf latency p99 < 50ms; p50 < 10ms
2. #security no plaintext secrets; all creds in vault
3. #perf #security TLS handshake < 100ms
```

### Diff Notation

`△` in header marks a diff section. `+`/`-`/`~` at line start = add/remove/modify (diff context only).

## Comprehensive Example

```
# 7 API Gateway Migration
gateway v1 -> v2; 3 breaking changes, 2 deprecations; rollout by 2026-05-01

## 7.1 Breaking Changes [supersedes @4.1]
auth + rate-limiting + payload format; ALL clients require update

1. + OAuth2 PKCE flow replaces API key auth [confirmed]
2. - X-Api-Key header no longer accepted
3. ~ rate limit 100/min -> 500/min per client; burst 50 -> 200
4. ~ request body max 1MB -> 5MB

5. (IMPORTANT) ALL v1 tokens invalidated at cutover
6. DONE client SDK updated to PKCE flow
7. WIP #security token rotation policy; target: 24h expiry [likely]

## 7.2 Deprecations
2 endpoints deprecated; removal after 90-day window

1. GET /users/search -> replaced by POST /users/query
2. GET /health/detailed -> merged into GET /status

## 7.3 Rollout Strategy
phased rollout per @7.1 (breaking changes); canary first

1. IF canary error rate < 0.1% THEN promote to 10%
   ELSE rollback -> investigate -> retry
2. #perf latency budget p99 < 200ms; current p99 ~= 180ms [confirmed]
3. migration window per @7.1.3 (rate limit change)

### 7.3.1 Client Migration
3 client tiers; different timelines

1. | tier     | deadline   | contact     | status  |
   |----------|------------|-------------|---------|
   | internal | 2026-04-20 | #platform   | DONE    |
   | partner  | 2026-04-25 | #partner    | WIP     |
   | public   | 2026-05-01 | docs update | OPEN    |

2. MATCH client.auth_version
   CASE v2: route to new gateway
   CASE v1 + migration_flag: dual-write; log divergence
   CASE v1: serve from old gateway; inject deprecation header

3. (ACTION) partner notification scheduled 2026-04-18
4. rollback procedure same as @5.3.2 (blue-green swap) ~95% safe [likely]
```

Expanding `@7.3.1`:

```
# 7.3.1 Client Migration
3 client tiers; different timelines

## 7.3.1.1 Internal Clients
1. 12 services migrated; auth SDK v2.1.0
2. zero errors post-migration [confirmed]

## 7.3.1.2 Partner Clients
1. 4 of 7 partners migrated; 3 pending sandbox validation
2. (QUESTION) partner-C using custom auth wrapper; compatibility unknown [speculative]

## 7.3.1.3 Public Clients
1. migration guide published; SDK auto-upgrade path available
2. deprecation header active since 2026-04-10
```
