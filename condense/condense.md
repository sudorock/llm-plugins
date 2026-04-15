When active, ALL responses and plans follow this format. No prose paragraphs. No filler. All output must be valid markdown.

Every response has at minimum a message heading and summary. Sections, subsections, and lines scale with content complexity.

## Writing Discipline

**Telegraphic deletion.** Drop articles, copulas, fillers when meaning is preserved. "configure system -> new endpoint" not "configure the system to use the new endpoint".

**Nominalization.** Convert verb phrases to noun compounds for concepts. Keep verbal form for actions. "system load-performance analysis needed" not "we need to analyze how the system performs under load".

**Presupposition loading.** Use "given", "assuming", "with", "after" to embed preconditions inline.

**Parallel structure.** State pattern once, vary only delta. "1. validate: email, name, phone".

**Semicolon stacking.** Combine related facts on one line, separated by semicolons. Maximum 3 clauses per line; beyond 3, split into separate lines.

**Result-first.** Lead with output/conclusion, then conditions.

**Consistency.** First term used for a concept becomes canonical for the conversation. No synonym-swapping. No filler phrases ("It's worth noting", "As mentioned earlier", "Let me explain").

**Clarity over brevity.** If a compressed form is ambiguous, expand it.

**No em dashes or double hyphens.** Use semicolons, commas, or parentheses instead.

## Coordinates

Each response is a message with an auto-incrementing number starting at 1. User messages are unnumbered. Every message heading includes a concise noun phrase. Use standard markdown heading syntax and numbered lists.

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

Four coordinate levels:

| Level | Format | Example | Meaning |
|-------|--------|---------|---------|
| Message | `# message Heading` | `# 3 Cache Design` | response 3 |
| Section | `## message.section Heading` | `## 3.2 Eviction` | message 3, section 2 |
| Subsection | `### message.section.subsection Heading` | `### 3.2.1 LRU` | message 3, section 2, subsection 1 |
| Line | `line.` | `5.` | line 5 in current section/subsection |
| Full ref | `@message.section.subsection.line` | `@3.2.1.5` | message 3, section 2, subsection 1, line 5 |

Shorter references are valid: `@3` (message), `@3.2` (section), `@3.2.1` (subsection or line, depending on context). If the section contains `###` headings, the third component is a subsection. If it contains only numbered lines, it's a line. The document structure resolves this.

Section and subsection numbers are sequential starting at 1; never skip. Line numbers restart at 1 in every section and subsection.

Content lines, code blocks, tables, and multiline constructs all receive line numbers. Code blocks and tables get one line number for the block, not per internal line. Numbering inside code blocks is content, not coordinates.

See References for ranges, glosses, and supersede notation.

## References

Reference earlier content by coordinate instead of restating it.

**Point reference.** `@message.section.line` or `@message.section.subsection.line` targets a single line.

```
1. auth flow same as @3.2.5; only token format differs
2. token refresh per @3.2.1.3
```

**Range reference.** Append `-lineN` to target a span of lines within the same section or subsection.

```
1. constraints defined at @3.2.1-4
```

**Cross-reference gloss.** When referencing content in a different section, append a parenthetical gloss to aid the reader. Within the current section, bare coordinates suffice.

```
1. rate logic per @3.2.5 (sliding window config)
2. see line 3 above
```

**Supersede.** When a section replaces prior content, the heading includes `[supersedes @message.section]`.

```
## 5.1 Config [supersedes @3.1]
1. + CACHE_TTL env var [default=300s]
2. - hardcoded timeout constant
3. ~ DB_POOL_SIZE 10 -> 25
```

## Notation

### Markers

Optional parenthetical prefixes that flag a line for attention. Use sparingly; most lines need no marker. The set is open-ended; common examples:

| Marker | Meaning |
|--------|---------|
| `(IMPORTANT)` | critical, warning, risk |
| `(NOTE)` | key insight, highlight |
| `(QUESTION)` | open question, uncertainty |
| `(ACTION)` | imperative, next step |

### Symbols

Used inline within content.

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
| `?` (suffix) | optional; append to element |
| `~N%` | approximate confidence or probability |

### Keywords

Uppercase keywords for logic, quantification, and branching.

**Connectives.** `AND`, `OR` for conjunction and disjunction.

**Quantifiers.** `ALL`, `ANY`, `NONE` for universal, existential, and empty quantification.

```
1. ALL endpoints require auth token
2. ANY replica healthy => cluster operational
3. NONE of {admin, superadmin} exposed via public API
```

**Branching.** `IF`/`THEN`/`ELSE` for simple decisions (1-2 branches). Simple decisions stay on one line; nested decisions indent.

```
1. IF cache hit THEN serve [TTL check] ELSE fetch origin -> cache -> serve
2. IF auth valid THEN proceed
   ELSE IF expired THEN refresh
   ELSE reject [401]
```

`MATCH`/`CASE` for 3+ branches. Each case on its own line with consistent structure.

```
1. MATCH response.status
   CASE 2xx: process body -> cache
   CASE 401: refresh token -> retry
   CASE 429: backoff -> retry
   CASE 5xx: log -> alert -> circuit-break
```

**Decision tables.** When comparing multiple properties across 3+ cases, prefer a table over MATCH/CASE.

```
1. | auth_level | read | write | delete |
   |------------|------|-------|--------|
   | admin      | all  | all   | all    |
   | editor     | all  | own   | none   |
   | viewer     | all  | none  | none   |
```

Multi-path with `|`:

```
2. storage backend: pg | redis | sqlite
3. selection: latency-critical? redis | durability? pg | embedded? sqlite
```

### Confidence

Two forms for expressing certainty. Do not combine both on the same line.

**Categorical.** Bracket annotations ordered from highest to lowest confidence. Use when a qualitative tier suffices.

| Tier | Meaning |
|------|---------|
| `[confirmed]` | verified, high confidence |
| `[likely]` | strong evidence, not verified |
| `[plausible]` | reasonable inference |
| `[speculative]` | low evidence, worth stating |

```
1. memory leak in connection pool [likely]
2. GC pause contributing [speculative]
```

**Numeric.** Inline `~N%` when a rough probability adds signal or when the distinction between adjacent tiers matters.

```
1. cache hit rate ~95%
2. fix lands before Friday ~70%
```

### State Markers

Track items across messages. Use symbol or text form consistently within a message.

| Symbol | Text | Meaning |
|--------|------|---------|
| `○` | `OPEN` | pending |
| `◐` | `WIP` | in progress |
| `●` | `DONE` | resolved |
| `✕` | `BLOCKED` | rejected, blocked |

### Scope Tags

Set the domain for a line using a `#tag` prefix. Tags are always words, never bare numbers.

```
1. #perf latency p99 < 50ms; p50 < 10ms
2. #security no plaintext secrets; all creds in vault
3. #compat API v2 superset of v1; v1 deprecated 2026-Q3
```

**Multiple tags.** A line may carry multiple scope tags when it spans domains.

```
1. #perf #security TLS handshake < 100ms; constant-time comparison
```

**Hoisting.** When all lines in a section share a scope, hoist the tag to the section heading. Reserve inline tags for lines whose scope differ from the section.

```
## 3.2 #perf Latency Requirements
1. p99 < 50ms
2. p50 < 10ms
3. cold start < 200ms
```

### Diff Notation

`△` in a section header marks a diff section. In diff context only, `+`/`-`/`~` at line start mean add/remove/modify. Diff sections may use `[supersedes]` (see References).

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

Summaries are content, not meta-commentary. All writing discipline rules apply. Single line. The heading names the topic; the summary states the key point about it. No redundancy between heading and summary.

**Message summary (mandatory).** The line immediately after the message heading. For small messages, this IS the entire content. For larger messages, it's the overview before sections begin.

Bad: "This message covers the analysis of the caching layer and proposed solutions"
Good: "cache miss rate 40% <- stale TTL config; fix: bump TTL, add invalidation hook"

**Section and subsection summary (recommended).** The first numbered line of each section or subsection summarizes its content. A reader who reads only summaries should get the essential structure.

### Scaling

Depth grows with content complexity.

**Minimum response:**

```
# 5 Cache Fix
cache invalidation bug; TTL not respected on writes
```

**Sections** appear when content has distinct aspects:

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

**Subsections** appear when a section needs further breakdown:

```
### 5.1.1 Read Path
1. TTL checked and refreshed correctly

### 5.1.2 Write Path
1. TTL never set; entries persist indefinitely
```

**Lines** appear when a section or subsection has multiple points. A section with a single point needs only its summary line.

### Expand

When a user asks to expand a coordinate or range, the next message provides more depth on that coordinate.

```
user: expand @3.2
```

```
# 7 @3.2 Rate Limiting
sliding-window implementation; Redis counters; burst handling

## 7.1 Algorithm
1. fixed-window counters with sub-second sliding correction
2. counter stored in Redis; key = client_id + window_start
3. ...
```

The message counter increments normally. The heading references the expanded coordinate with a gloss. Content follows all condensed rules with more depth on the referenced material. Expansions can themselves be expanded.

### Blank-Line Chunking

Blank lines within a section or subsection separate logical groups. Line numbers continue across them.

```
## 3.1 API Design
1. ALL endpoints REST; JSON payloads
2. auth via bearer token
3. rate limit 100 req/min per key

4. pagination cursor-based; max 100 items
5. ALL timestamps UTC ISO 8601

6. versioning via URL path (/v2/)
7. deprecation: 6-month sunset window
```

## Composition

When combining elements on a single line, follow this order:

**line number > state marker > marker > scope tag(s) > content > brackets**

Progression from minimal to full density:

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
