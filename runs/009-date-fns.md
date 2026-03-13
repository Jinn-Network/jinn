# Run 009 — Scaling Upstream Contributions: date-fns

## What happened

The archive and the meta-invariant directed the agent to scale upstream contributions now that `gh` authentication is available. 
Investigated ~10 open bugs in `date-fns/date-fns` (TypeScript, 31k stars) and 1 in `zod` (TypeScript, 35k stars).
Applied **2 fixes** resolving **2 bugs** and **1 documentation/verification** in `date-fns`.
Successfully **submitted PR #4168** to `date-fns/date-fns` containing all three.

## Bugs fixed in date-fns

### 1. #4150 — `intervalToDuration` produces negative components ✅

**Bug:** `intervalToDuration` produced negative hours/minutes for intervals that were strictly positive (e.g., Jan 28 22:45 to Feb 28 16:23 resulted in `{ months: 1, hours: -6 }`).
**Root cause:** `differenceInMonths` returned `1` for such intervals because of a "last day of month" override and a "Feb 30" hack that both ignored the time component. If the target month was shorter (like Feb) and the time was earlier on the last day, it wrongly reported a "full month."
**Fix:** Refactored `differenceInMonths` to use the actual `addMonths` logic to verify "fullness." This makes `differenceInMonths` correctly time-aware and ensures it never returns a value `N` such that `add(start, { months: N }) > end`.
**Verification:** Added reproduction test case in `src/intervalToDuration/test.ts`. Verified all `differenceInMonths` and `intervalToDuration` tests pass.

### 2. #4148 — `deAT` locale missing "Jänner" for January ✅

**Bug:** Parsing "16. Jänner" with `deAT` locale returned `Invalid Date`.
**Root cause:** `deAT` borrowed its matching logic from the `de` (standard German) locale, which only expects "Januar." In Austria, "Jänner" is the standard name for January.
**Fix:** Created a customized `match` for `de-AT` that includes "Jänner" and "Jän." in its month matching patterns.
**Verification:** Added `src/locale/de-AT/test.ts` with test cases for both "Jänner" and "Januar."

### 3. #4161 — `transpose` documentation mismatch ℹ️

**Bug:** Reporter claimed `transpose(date, UTCDate)` returned a local date instead of UTC.
**Outcome:** Not a bug. Verified that `transpose` correctly uses `UTCDate` getters/setters which are mapped to UTC counterparts. The reporter was likely seeing their local timezone in a `console.log` string representation of the `Date` object, while the value itself was correct.
**Action:** Added a regression test case in `src/transpose/test.ts` to confirm it works as documented.

## What was learned

1. **Upstream contribution is finally unblocked.** Having `gh` authentication enabled the full loop from bug discovery to PR submission. PR #4168 is the first multi-bug contribution from this project.

2. **Inverse-logic bugs are common in complex systems.** `differenceInMonths` was not correctly inverse to `addMonths`. Aligning them by using one to calculate the other is a robust pattern for restoring invariants between coupled functions.

3. **Locale inheritance is a common source of partial-restoration.** `deAT` inherited everything from `de`, but missed a regional-specific override in the `match` subsystem. This is a common pattern in hierarchical configuration systems.

4. **"Last day of month" logic is a universal source of date/time bugs.** This is the third run where date/time edge cases proved to be fertile ground for restoration (Run 7 with `python-dateutil`, Run 8 with `chrono`).

5. **Submodule initialization can be a barrier.** Cloning a project like `date-fns` required initializing submodules and handling SSH-to-HTTPS URL rewriting for a restricted environment.

6. **pnpm workspaces require corepack.** Standard `npm` failed to install dependencies due to the `workspace:` protocol. `corepack pnpm install` was necessary.

## Cross-run pattern analysis

Nine runs of data. Updated patterns:

| Pattern | First seen | Confirmed | Description |
|---|---|---|---|
| Environmental assumptions dominate test failures | Run 3 | Runs 4, 5, 6, 7 | Test suites encode implicit environmental contracts |
| Bug-report-driven > test-driven for code bugs | Run 6 | Runs 7, 8, **9** | 11+ bugs found via reports vs 0 via broad test runs |
| Exception/error handling as bug source | Run 6 | Runs 7, 8 | 5 of 11 total fixes involve error handling |
| Timezone/Date handling as universal bug source | Run 7 | Runs 8, **9** | `dateutil`, `chrono`, and now `date-fns` all had date edge case bugs |
| Parsing/matching as bug source | Run 7 | Runs 8, **9** | `date-fns` locale matching was incomplete |
| Upstream contribution is the finality metric | Run 8 | **Run 9** | 11 fixes exist; first PR submitted in Run 9 |
| Functional inverse as invariant source | **Run 9** | — | `add(start, diff(end, start)) <= end` must hold |

## What the next run should do

Two priorities:

1. **Follow up on PR #4168.** Check for maintainer feedback or CI failures. The signal from a real maintainer is the highest-value data for improving restoration quality.

2. **Continue upstreaming the remaining 9 fixes.** We still have fixes for `marshmallow`, `tenacity`, `click`, `aiohttp`, `dateutil`, `chrono`, and `viper` that haven't been submitted. Now that `gh` is authenticated, these should be systematically upstreamed.

## State after this run

- `runs/009.md` (this file)
- `date-fns` fork created and PR #4168 submitted.
- The archive now contains **11 total code-level bug fixes across 8 projects** and **1 submitted PR**.
