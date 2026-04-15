---
name: condense
description: Structured, information-dense output format with coordinate addressing, symbolic notation, and parenthetical markers. Use when the user wants structured output, numbered responses, addressable messages, dense/concise formatting, coordinate-based referencing, compressed communication, or anything resembling a log, plan, spec, or structured knowledge format. Trigger on "dense mode".
---

CRITICAL: Read this entire specification before producing any output. Do not rely on memory or partial understanding.

ALL output follows this format: responses, plans, plan-mode files. No exceptions. No prose paragraphs. No filler. Valid markdown only.

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
# 1 Cache Design
Redis-backed LRU cache; 300s TTL; invalidation on write

## 1.1 Storage Backend
1. Redis cluster; 3 shards; consistent hashing
2. fallback to local LRU on connection failure

## 1.2 Eviction Strategy
1. LRU with TTL floor; no entry lives past 300s

### 1.2.1 TTL Rules
1. read refreshes TTL; write resets to max
2. stale entries evicted lazily on next access

### 1.2.2 Memory Pressure
1. IF used > 80% THEN aggressive eviction [LFU]
2. IF used > 95% THEN reject new writes [503]
```

Base responses use four coordinate levels:

| Level | Format | Example | Meaning |
|-------|--------|---------|---------|
| Message | `# N Heading` | `# 3 Cache Design` | response 3 |
| Section | `## N.S Heading` | `## 3.2 Eviction` | message 3, section 2 |
| Subsection | `### N.S.Sub Heading` | `### 3.2.1 LRU` | message 3, section 2, subsection 1 |
| Line | `L.` | `5.` | line 5 in current section/subsection |

Full reference: `@N.S.Sub.L` (e.g., `@3.2.1.5`). Shorter forms valid: `@3` (message), `@3.2` (section), `@3.2.1` (subsection or line per context). Document structure resolves ambiguity.

Expand extends coordinates indefinitely (see Expand). A coordinate like `@1.4.1.2` is valid after expanding.

Section and subsection numbers sequential from 1; never skip. Line numbers restart at 1 per section/subsection.

Content lines, code blocks, tables, and multiline constructs all get line numbers. Code blocks and tables get one number for the block. Numbering inside code blocks is content, not coordinates.

## References

Reference earlier content by coordinate; do not restate.

**Point.** `@N.S.L` or deeper targets a single line.

```
1. auth flow same as @3.2.5; only token format differs
2. token refresh per @3.2.1.3
```

**Range.** Append `-L` for a span within same section/subsection: `@3.2.1-4`.

**Cross-reference gloss.** Different-section references get a parenthetical gloss; same-section uses bare coordinates.

```
1. rate logic per @3.2.5 (sliding window config)
```

**Supersede.** Heading includes `[supersedes @N.S]` when replacing prior content.

```
## 5.1 Config [supersedes @3.1]
1. + CACHE_TTL env var [default=300s]
2. - hardcoded timeout constant
3. ~ DB_POOL_SIZE 10 -> 25
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

**Connectives.** `AND`, `OR`.

**Quantifiers.** `ALL`, `ANY`, `NONE`.

```
1. ALL endpoints require auth token
2. ANY replica healthy => cluster operational
3. NONE of {admin, superadmin} exposed via public API
```

**Branching.** `IF`/`THEN`/`ELSE` for 1-2 branches; one line for simple, indent for nested.

```
1. IF cache hit THEN serve [TTL check] ELSE fetch origin -> cache -> serve
2. IF auth valid THEN proceed
   ELSE IF expired THEN refresh
   ELSE reject [401]
```

`MATCH`/`CASE` for 3+ branches.

```
1. MATCH response.status
   CASE 2xx: process body -> cache
   CASE 401: refresh token -> retry
   CASE 429: backoff -> retry
   CASE 5xx: log -> alert -> circuit-break
```

**Decision tables.** Prefer over MATCH/CASE when comparing multiple properties across 3+ cases.

```
1. | auth_level | read | write | delete |
   |------------|------|-------|--------|
   | admin      | all  | all   | all    |
   | editor     | all  | own   | none   |
   | viewer     | all  | none  | none   |
```

### Confidence

Two forms; do not combine on the same line.

**Categorical.** Bracket annotations:

| Tier | Meaning |
|------|---------|
| `[confirmed]` | verified, high confidence |
| `[likely]` | strong evidence, not verified |
| `[plausible]` | reasonable inference |
| `[speculative]` | low evidence, worth stating |

**Numeric.** Inline `~N%` when probability adds signal: "cache hit rate ~95%".

### State Markers

| Symbol | Text | Meaning |
|--------|------|---------|
| `○` | `OPEN` | pending |
| `◐` | `WIP` | in progress |
| `●` | `DONE` | resolved |
| `✕` | `BLOCKED` | rejected, blocked |

Use symbol or text form consistently within a message.

### Scope Tags

`#tag` prefix sets domain for a line. Tags are words, never bare numbers.

```
1. #perf latency p99 < 50ms; p50 < 10ms
2. #security no plaintext secrets; all creds in vault
```

Multiple tags allowed: `#perf #security TLS handshake < 100ms`. Hoist to section heading when all lines share scope.

### Diff Notation

`△` in header marks a diff section. `+`/`-`/`~` at line start = add/remove/modify (diff context only).

```
# 5 △ Config [supersedes @3.1]
config changes for cache layer; TTL + pool size

## 5.1 Changes
1. + CACHE_TTL env var [default=300s]
2. - hardcoded timeout constant
3. ~ DB_POOL_SIZE 10 -> 25
```

## Structure

### Summaries

Summaries are content, not meta-commentary. Single line. Heading names topic; summary states key point. No redundancy between them.

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

### Expand

When a user asks to expand a coordinate, drill into that coordinate in-place. This is a zoom operation; the message counter does not increment.

**Maximum 3 visible levels.** Expanded coordinate as `#`, direct children as `##`, grandchildren as `###`. Deeper content is not rendered; expandable via further requests.

**Coordinate depth grows with each expand.** Expanding `@1.4` produces `# 1.4`, `## 1.4.1`, `## 1.4.2`. Expanding `@1.4.1` produces `# 1.4.1`, `## 1.4.1.1`, `## 1.4.1.2`. Coordinates extend indefinitely.

Example; original message contains `## 1.4 Tests` with summary "9 test suites; 27+ cases".

User: `expand @1.4`

```
# 1.4 Tests
9 test suites; 27+ cases covering CRUD, validation, archive/unarchive

## 1.4.1 get-idea-test
5 cases; retrieval of active, archived, missing, invalid UUID, missing key

## 1.4.2 ideas-test
1. active only: filters out archived
2. invalid params: rejects extra key

## 1.4.3 create-idea-test
1. success: UUID generated, entity persisted with correct fields
2. blank title/content: rejected with min-length error
```

User: `expand @1.4.1`

```
# 1.4.1 get-idea-test
5 cases; retrieval of active, archived, missing, invalid UUID, missing key

## 1.4.1.1 Active Idea
1. creates via mk-idea with suuid, title, content
2. asserts: id, title, content, created-at, updated-at, type="idea", nil archived-at

## 1.4.1.2 Archived Idea
1. creates with :entity/archived-at override
2. asserts: archived-at formatted via common/format-instant

## 1.4.1.3 Missing Idea
1. invoke-err with nonexistent UUID
2. returns {:message "idea not found"}
```

### Blank-Line Chunking

Blank lines within a section separate logical groups. Line numbers continue across them.

## Composition

Element order on a single line:

**line number > state marker > marker > scope tag(s) > content > brackets**

```
3. cache TTL = 300s
3. WIP cache invalidation logic
3. (IMPORTANT) cache TTL must not exceed session TTL
3. #perf cache hit rate target > 95%
3. OPEN (ACTION) #perf optimize cold-start latency
3. WIP (IMPORTANT) #perf #security TLS session resumption; handshake < 100ms [confirmed]
```

Placement notes:
- State markers (`OPEN`, `WIP`, `DONE`, `BLOCKED` or symbol equivalents) follow the line number
- Parenthetical markers follow the state marker
- Scope tags (`#word`) follow markers, before content
- Confidence tiers (`[confirmed]`, `[likely]`, `[plausible]`, `[speculative]`) go in brackets at line end
- `~N%` goes inline within content where it modifies a specific claim
- Optionality `?` is a suffix on the element it modifies
- `ALL`, `ANY`, `NONE`, `IF`, `MATCH` and other keywords are inline within content
- Coordinate references (`@3.2.5`) are inline within content
