# Liora Findings — AI-Augmented Engineering Exercises (Python focus)

This document summarizes what I did and learned across the AI-Augmented Engineering exercises shared in this chat.
My main focus was **Python**, but some exercises provided sample code in JavaScript/Java — for those, I documented the verification/refactoring approach and takeaways (without forcing a full rewrite of non-Python code).

---

## 0) Scope & How I Worked
- **Goal:** show process and learning (prompts + verification), not “perfect production code”.
- **Approach pattern I reused:**
  1) clarify intent (what problem / expected behavior)
  2) confirm assumptions (read code/tests, find entry points)
  3) verify with tests + edge cases
  4) refactor only if I can prove behavior stays the same

---

## 1) Knowing Where to Start (Task Manager, Python)

### Part 1 — Project Structure & Tech Stack
**What I did**
- Looked at repo folders first (without deep reading).
- Skimmed config + README to infer how it runs.
- Identified likely entry points (where tests and main logic live).

**Initial assumptions vs. corrected**
- I initially assumed “framework-ish” structure; after scanning, it looked more like a small educational codebase: core logic + tests.
- I assumed dependencies might be in a requirements file; if missing, then the repo is intentionally minimal and relies on standard libs + pytest.

**Key components (my mental model)**
- “task model” (Task / enums like status/priority)
- “priority algorithm” (score + sorting + top N)
- “parsing or input processing” (if present)
- tests as the source of truth for expected behavior

### Part 2 — Finding Feature Implementation (“Export tasks to CSV”)
**Search strategy**
- Search terms: `export`, `csv`, `write`, `file`, `save`, `serialize`, `format`, `report`, `io`.
- If nothing exists: implement as a small function close to where tasks are listed (or a “service/util” module).

**Plan**
- Decide export format (columns, header row, date format).
- Convert tasks → rows (stringify enums/dates).
- Write CSV using Python `csv` module.
- Add tests: correct header, correct row values, correct escaping/commas.

### Part 3 — Domain Model Understanding
**Entities I expect**
- Task (title/description/tags/due_date/priority/status/created_at/updated_at)
- Enums: TaskPriority, TaskStatus
- The prioritization algorithm depends on those fields.

**Glossary (quick)**
- **Priority:** categorical importance (LOW..URGENT)
- **Status:** lifecycle stage (TODO/IN_PROGRESS/REVIEW/DONE etc.)
- **Due date:** deadline; affects score
- **Tags:** keywords; some tags boost score

### Part 4 — New Business Rule (“Overdue > 7 days => abandoned unless high priority”)
**Where it belongs**
- In the scoring logic (priority algorithm) or in a rule function that runs before sorting.

**Questions I would ask team**
- What is “abandoned” in this system? (new status? existing status?)
- Does it change the stored task, or only ranking/visibility?
- Should it happen automatically on read, or only on a scheduled job?

**My proposed change (minimal risk)**
- Start with non-destructive behavior: treat as “abandoned” only in ranking/filtering unless product explicitly requires persistent state change.

**Reflection**
AI prompts helped me quickly build a map: where code lives, what to read first, and what assumptions to test.

---

## 2) Codebase Exploration Challenge (Python Task Manager)

### Part 1 — Understand specific feature (task creation & status updates)
**What I looked for**
- where tasks are instantiated
- how status is updated
- whether updates also change `updated_at`
- where persistence/state lives (in-memory list, file, DB, etc.)

**Execution flow (generic, but what I verified conceptually)**
Input/request → create Task → store → later update status → persist changes → tests confirm expected state.

### Part 2 — Deepen understanding (task prioritization)
**What became clearer after guided questions**
- scoring is a weighted sum of factors (priority weight + due date + status penalties + tags + recency boost)
- “days” calculations can be tricky (timezone, rounding, `.days` behavior)

### Part 3 — Mapping data flow (mark as complete)
**What I track**
- which function receives the “complete” action
- which fields change (status, completed_at, updated_at)
- what errors can happen (missing task id, invalid status transition, etc.)

### Part 4 — Presentation notes (what I’d say in 3–5 minutes)
- Architecture is small: domain model + scoring algorithm + helpers + tests.
- Prompts were useful for: “find entry points”, “explain flow”, “spot edge cases”.
- Hardest part: confirming time/due-date logic and status transitions.

---

## 3) Algorithm Deconstruction Challenge (Python)

I focused on understanding the algorithms conceptually and identifying tricky parts to test.

### Algorithm 1 — Task Priority Sorting
**How I explain it to a junior**
- We compute a **score per task**.
- Higher score = higher priority in the sorted list.
- Then we sort tasks by score descending and take top N.

**Hard parts / pitfalls**
- date math: overdue vs due today vs due soon
- status penalties can dominate score
- tag matching (case sensitivity)
- recency boost: “< 1 day” boundary behavior

**How AI changed my understanding**
- It highlighted that time logic + sorting stability is where subtle bugs hide, not the arithmetic itself.

**Improvements I’d consider**
- compute scores once (avoid recalculating in comparator loops)
- extract “due date score” and “status penalty” into named helpers for readability + testability

### Algorithm 2 — Task Text Parser (free-form text → structured task)
**Key responsibilities**
- parse priority markers `!1..4` or `!urgent`
- parse tags `@tag`
- parse due date markers `#tomorrow`, weekdays, or `YYYY-MM-DD`
- clean final title (remove markers + normalize spaces)

**Hard parts**
- conflicting markers (multiple dates/tags)
- ambiguous tokens inside title
- invalid dates and fallback behavior

**Tests I would prioritize**
- minimal input (just title)
- multiple tags
- one priority marker
- date keywords
- invalid date token should not crash

### Algorithm 3 — Task List Merging (two-way sync)
**Key idea**
- union of IDs across local + remote
- resolve conflicts using `updated_at`
- special rule: DONE “wins”
- tags are unioned

**Hard parts**
- tie-breakers when timestamps equal
- avoiding accidental mutation of source objects
- ensuring “should_update_local/remote” flags are correct

---

## 4) AI Solution Verification Challenge (Buggy mergeSort sample)

### What was wrong
In the provided `merge()` function:
- the loop meant to append remaining `left` elements increments `j` instead of `i`, which can cause incorrect results or infinite behavior in some cases.

### Three verification strategies I applied

**1) Collaborative Solution Verification**
- I restated the intended behavior: merge two sorted arrays into one sorted array.
- I traced a small example by hand:
  - left = [1, 4], right = [2, 3]
  - verify indexes move correctly and all elements end up in result.

**2) Learning Through Alternative Approaches**
- Compared the buggy merge sort vs:
  - using Python’s `sorted()` as an oracle for expected output (conceptually)
  - writing a merge using `extend(left[i:])` / `extend(right[j:])` pattern (cleaner)

**3) Developing a Critical Eye**
- I looked for classic bug patterns:
  - wrong index increment
  - off-by-one loops
  - leftover copy step mistakes

### Final verified fix (conceptual)
- Change `j++` to `i++` in the leftover-left loop.

### Reflection
- My confidence increased after tracing + “oracle” comparison.
- The most scrutiny was required around index movement and leftover loops.
- The most valuable technique was manual tracing with a tiny input.

---

## 5) Using AI to Help With Testing (Python focus)

### Part 1.1 — Behavior Analysis (calculate_task_score)
**Behaviors to test (at least 5)**
1) Base score depends on priority weight
2) Due date: overdue adds large boost
3) Due date: today / next 2 days / next week thresholds
4) Status penalties (DONE and REVIEW reduce score)
5) Tag boost when tags include blocker/critical/urgent
6) Recent update boost when updated < 1 day

**Edge cases**
- due_date = None
- tags empty
- unknown priority value
- updated_at missing/None (should error or default; needs spec)

### Part 1.2 — Test plan for 3 related functions
**Unit tests**
- calculate_task_score: cover each factor independently (priority, due date buckets, status, tags, recency)
- sort_tasks_by_importance: ordering is correct based on known scores
- get_top_priority_tasks: returns N, handles N > len(tasks), handles empty list

**Integration tests**
- given a set of tasks, top N returned matches expected ordering and slicing

**Dependencies**
- stable “now” time: use fixed clock or freeze time for predictable tests

### Part 2 — Improving a single test
- make tests assert *behavior* not implementation:
  - check score value outcome, not internal variable names
- use clear naming:
  - `test_overdue_task_gets_overdue_bonus()`

### Part 3 — TDD practice (new feature + bug fix)
**Feature: “assigned to current user => +12 boost”**
- First failing test: same task, same inputs, only `assignee == current_user` changes score by +12.
- Minimal code change: add `if task.assignee == current_user: score += 12`

**Bug fix: “days since update” calculation wrong**
- Create test where updated_at is “yesterday”, ensure daysSinceUpdate = 1 (not 0 due to rounding).
- Fix should use `.days` (Python) rather than manual milliseconds style logic.

### Part 4 — Integration test
- Create tasks with different priority + due dates + statuses.
- Assert:
  - `get_top_priority_tasks` returns exactly expected tasks in order.

### Reflection
- Biggest lesson: tests get much easier when time is controlled (freeze “now”).
- AI was most useful to list behaviors/edge cases; I still had to decide expected outcomes.

---

## 6) Code Readability / Refactoring / Duplication (sample code exercises)

> These exercises were shared in Java/JS/Python samples. I didn’t rewrite the whole non-Python repo, but documented the refactoring patterns I’d apply safely (with tests).

### Exercise: Code Readability Improvement (Java sample UserMgr)
**Issues AI helped surface**
- cryptic names: `UserMgr`, `U`, `u_list`, methods `a()` and `f()`
- SQL string concatenation risk (injection)
- mixed responsibilities: validation + storage + DB write

**Improvements**
- rename to `UserManager`, `User`, `users`, `addUser()`, `findByUsername()`
- extract validation method
- use prepared statements (concept)

### Exercise: Function Refactoring (Python process_orders)
**Distinct responsibilities**
- validation (inventory/customer checks)
- pricing + discount
- shipping logic
- tax calculation
- inventory update
- building result output

**Decomposition plan**
- `validate_order(...)`
- `calculate_price(...)`
- `calculate_shipping(...)`
- `calculate_tax(...)`
- `apply_discount(...)`
- `build_result(...)`

### Exercise: Code Duplication Detection (JS calculateUserStatistics)
**Repeated pattern**
- compute average: sum loop + divide
- compute max: max loop

**Consolidation idea**
- generic helpers: `averageBy(userData, "age")`, `maxBy(userData, "age")`
- Keep it readable for juniors by limiting abstraction depth.

---

## 7) Function Decomposition Challenge (sample functions)

### JS validateUserData (nested conditionals)
**Responsibilities**
- required fields check by context (registration vs profile update)
- username rules + uniqueness checks
- password rules + confirmation
- email format + uniqueness
- date of birth constraints
- address schema + postal rules
- phone rules
- custom validations

**Benefit**
- smaller helper validators with clear names make it easier to extend and test.

### Python generate_sales_report (large function)
**Responsibilities**
- input validation
- date range filtering
- additional filtering
- metrics calculation
- grouping
- report type expansion (detailed/forecast)
- charts data
- output format generation

**Decomposition plan**
- `validate_inputs()`, `filter_by_date()`, `apply_filters()`, `compute_summary()`,
  `group_sales()`, `build_forecast()`, `build_charts()`, `render_output()`

### Java processCustomerData (pipeline + error handling)
**Responsibilities**
- load existing records for dedup
- source preprocessing
- validate fields
- normalize fields (email/phone/address/date)
- deduplicate
- track errors
- save to DB
- build report response

**Benefit**
- helper methods allow unit tests for each validation/normalization step.

---

## 8) Design Pattern Implementation Challenge (sample opportunities)

### Strategy (JS shipping cost)
- Replace nested conditionals with “shipping strategy objects” per method and per destination rule set.

### Factory (Python DB connections)
- Factory creates correct connection class/config per db_type, reduces constructor complexity.

### Observer (Java WeatherStation)
- Displays become observers; WeatherStation only notifies, doesn’t hardcode display update calls.

### Adapter (TypeScript payments)
- Wrap StripeAPI/PayPalAPI into a common `PaymentProcessor` interface, simplify PaymentService.

**Reflection**
Patterns mainly help when requirements change (new method, new payment gateway, new display).

---

## 9) Deepening Programming Language Understanding (journal prompts)

### Activity 1 — Idiomatic code transformation
**3 learnings**
- idioms improve readability more than micro-optimizations
- standard library often replaces custom loops
- naming + small functions matters as much as syntax

### Activity 2 — Code quality detective
**3 learnings**
- most code smells are about unclear intent (names, mixed responsibilities)
- tests give confidence to refactor
- consistent error handling and input validation prevents production surprises

### Activity 3 — Understanding a language feature
**3 learnings**
- learn feature by small practical example + edge cases
- compare to familiar concepts to avoid confusion
- always note “common mistakes” for that feature (e.g., misuse, hidden complexity)

---

## 10) Learning a New Programming Language with AI (exercise template)
I used the structured 4-step strategy as a reusable template:
1) conceptual differences
2) step-by-step breakdown
3) guided small implementation
4) verification and reflection

Key insight: AI is best as a tutor that asks questions, but I must still validate with docs/tests.

---

## 11) FastAPI Exercises (learning notes)

### Getting Started with FastAPI
- FastAPI is an API framework optimized for type hints + validation + auto docs.
- Key features: request validation (Pydantic), automatic OpenAPI docs (`/docs`), async support.

**Glossary (short)**
- **Path operation:** route handler like `@app.get("/items/{id}")`
- **Pydantic model:** schema + validation for request/response bodies
- **Dependency (Depends):** injectable function for shared logic (auth/db/etc.)
- **Uvicorn:** ASGI server that runs FastAPI

### Contextual Learning with FastAPI (mental model translation)
- Flask Blueprint ≈ FastAPI Router
- Middleware exists, but dependencies often replace what I’d do in middleware
- Validation is “first-class” via models/types, not manual parsing

### Documentation Navigation + Code Patterns
- The docs are easiest if read in this order:
  1) first steps + path params
  2) request body models
  3) dependency injection
  4) error handling
  5) security
- Complex pattern takeaway: dependency injection makes auth + db session handling explicit and testable.

---

## Common Themes Across All Exercises
- AI output is useful, but **must be verified**.
- Most subtle bugs hide in **edge cases** (time math, indexes, state transitions).
- Best workflow: **tests first**, then refactor.
- Clear naming + decomposing functions reduces confusion and makes testing simpler.

---

## Final Reflection
My biggest improvement was not “more code” — it was a better habit:
- clarify intent
- verify with small examples and tests
- refactor safely (only when behavior is protected by tests)
- document decisions so a teammate can follow my reasoning