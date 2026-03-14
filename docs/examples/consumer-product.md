# Consumer Product Example

This example demonstrates Ouro Loop applied to a consumer iOS/macOS creative collaboration app. The agent remediated ESLint errors in a React/Next.js frontend — a simple task where the ROOT_CAUSE verification gate caught a lazy fix and pushed the agent toward a genuinely superior architectural solution.

---

## Project Profile

| | |
|---|---|
| **Stack** | Next.js 15 + React 19 + TypeScript |
| **Platform** | iOS/macOS (Swift/SwiftUI frontend, web companion) |
| **Architecture** | AVAudioEngine, Core Data, CloudKit sync, CRDT conflict resolution |
| **Task** | Eliminate all ESLint errors to establish clean lint baseline |
| **Complexity** | Simple (config + 2 files, no DANGER ZONE touched) |

---

## BOUND Definition

### DANGER ZONES

```
Sources/Audio/Engine/              — Core audio engine. Crashes = audio glitches in live sessions.
Sources/Sync/ConflictResolver.swift — CRDT conflict resolution. Wrong merge = data loss.
Sources/IAP/                       — In-app purchases. Apple review compliance critical.
CoreData/Model.xcdatamodeld        — Core Data model. Migration errors = user data loss.
```

### NEVER DO

- Never block the audio thread with synchronous I/O or network calls
- Never change Core Data model without a tested migration mapping
- Never modify IAP product identifiers
- Never ship without testing on minimum supported iOS version
- Never silently drop audio frames
- Never store audio files in the app bundle

### IRON LAWS

- Audio callback latency never exceeds 10ms
- Core Data migrations tested with production-schema snapshots
- All network requests have timeout and retry logic
- UI updates happen only on MainActor
- Audio format conversions preserve sample rate and bit depth
- App launch to interactive time under 2 seconds on iPhone 13

---

## The Session

### Baseline

```
$ npx eslint .
3 errors, 3 warnings

Errors:
1. privacy/page.tsx:14  — <a> should be <Link>
2. privacy/page.tsx:182 — <a> should be <Link>
3. ReadingPage.tsx:78   — setState in useEffect
```

### Fix 1: Replace `<a>` with `<Link>`

Straightforward fix. Imported `Link` from `next/link`, replaced two `<a href="/">` with `<Link href="/">`. No DANGER ZONE involvement.

### Fix 2: Eliminate setState in useEffect (Attempt 1)

Restructured the effect but kept `setState` inside it. The React Compiler's `set-state-in-effect` rule is stricter than expected — even wrapping setState in a conditional inside useEffect still triggers it.

!!! failure "ROOT_CAUSE gate: FAIL"
    The gate correctly identified this as symptom-patching. The real fix was to eliminate the effect entirely, not restructure it.

```
[REMEDIATED] gate=ROOT_CAUSE action=revert_and_retry
  was: rewrote the effect but still called setState inside it
  did: realized the rule prohibits ANY setState in effect body
  now: eliminating the effect entirely — pure derived state
```

### Fix 2: Eliminate setState in useEffect (Attempt 2)

Replaced the `useState` + `useEffect` pattern with a pure derived value:

```typescript
// BEFORE: useState + useEffect (React Compiler rejects this)
const [readingComplete, setReadingComplete] = useState(false);
useEffect(() => {
  if (!readingComplete && !isLoading && messages.length > 1) {
    if (messages.some(m => m.role === "assistant")) {
      setReadingComplete(true);
    }
  }
}, [isLoading, messages, readingComplete]);

// AFTER: Pure derivation (no effect, no setState)
const readingComplete =
  readingCompleteOverride ||
  (!isLoading && messages.length > 1 &&
   messages.some(m => m.role === "assistant"));
```

!!! success "VERIFY: PASS"
    `npx eslint .` — 0 errors, 3 warnings. `npx next build` — successful.

---

## Results

```
phase:        1/1
verdict:      PASS
bound_check:  PASS — no DANGER ZONE touched, build passes (IRON LAW)
lint:         0 errors (was 3), 3 warnings (unchanged)
remediation:  1 — ROOT_CAUSE gate caught symptom-level fix
```

---

## Methodology Observations

1. **ROOT_CAUSE gate caught a lazy fix** — the first attempt "fixed" the lint error by restructuring the effect but keeping setState. The gate correctly identified this as symptom-patching.

2. **The derived state pattern was architecturally superior** — not just lint-clean, but better aligned with React's mental model. The methodology pushed toward a genuinely better solution, not just a passing test.

3. **Simple complexity was correct** — the task touched 2 files, no DANGER ZONE, clear scope. The complexity router correctly identified this as "execute directly" without needing a phase plan.

4. **IRON LAW verification caught nothing but provided confidence** — running `next build` after every change served as a safety net even for a simple task.

[:material-file-document: Full Session Log](../session-logs/consumer-product.md)
