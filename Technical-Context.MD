# Technical Context

Engineering-practice contract for code-writing agents in this project. Every implementation must respect what's written here. If a constraint conflicts with the user's request, flag the conflict — don't silently bypass it.

## Operating Context

Every later section of this contract is written for the Operating Context stated here.

- **Users:** The general public, with no authentication. They use HotTea, a simple website that converts temperatures between Celsius and Fahrenheit.
- **Operating Scale:** A few visitors a day, built by one developer, with no delivery deadline.
- **Where it runs:** GitHub Pages, as a static site with no server-side code. It is built in Claude Code cloud sessions (Linux containers).
- **Credentials:** The site holds none. The developer gets GitHub access through a GitHub login via the Claude app connector, and uses Azure DevOps for the board.
- **Alongside it:** None.
- **Not the Product's concern:** GitHub Pages infrastructure, CDN and TLS, which GitHub owns, and visitors' browsers and devices.
- **Last confirmed:** 2026-10-12

## Overriding Principles
1. Static only. The site runs entirely in the browser, with no server-side code, because GitHub Pages can only serve static files.
2. No secrets, no tracking. No credentials, API keys or personal data live in the repo or the site, and there are no cookies or analytics, because visitors are anonymous.
3. No runtime dependencies without a reason. Plain HTML, CSS and JavaScript are the default, and a library is added only when it earns its place, because the site only needs to do temperature arithmetic.
4. Conversion logic is tested. Celsius to Fahrenheit and back must be correct, including negative values, decimals and invalid input, because a wrong answer is the one way this site can fail.

## Frameworks
- **HTML5, CSS and vanilla JavaScript** — the whole site; no UI framework and no build step.
- **Node.js (>=20)** — runs the tests only; the site itself does not use it.
- **`node:test`** — the test runner, built into Node.

## 3rd-party tech
- **GitHub Pages** — hosts the static site.
- **GitHub** — holds the source and pull requests.
- **Azure DevOps (`EnateInternal` / `HotTea`)** — the work-item board.
- None at runtime: the site calls no external APIs, fonts or scripts.

## Packages in use
**Testing:** `node:test` and `node:assert` (built into Node; no packages installed)

No runtime packages. A new dependency needs a recorded reason (Overriding Principle 3).

## Toolchain pinning
- Policy: range — the conscious per-project decision, because Node is used only for testing.
- Pinned tools: Node.js `>=20`, set in the `engines` field of `package.json`; `node:test` is stable from Node 20.
- Bumping: raise the floor in a pull request only when a test needs a newer Node feature.

## Instrumentation
No logging, metrics or tracing in the site: it writes nothing to the console in production and sends nothing anywhere. Test output goes to the terminal through `node:test`.

## User Feedback Approach
None defined for now. HotTea is a simple page; this section is revisited if that changes.

## Branching and Environments
- Model: trunk-based. `main` moves only through a pull request, with squash merges, linear history and no force-push.
- Branches: short-lived `story/<id>-<slug>` branches, cut from `feature/<id>-<slug>` when the Story has a parent Feature, otherwise from `main`.
- Environments: production only. GitHub Pages serves `main`; there is no dev or staging environment.
- Protected: `main`

## Git Guardrails
- Never push to `main` and never force-push any branch.
- Never run `git reset --hard`, `git clean`, `git branch -D` or rewrite history on a shared branch without explicit confirmation.
- Never skip a pre-commit hook (`--no-verify`) instead of fixing the underlying issue.
- Hooks in place: `.githooks/pre-commit`.
- **HITL delivery opens and closes its own board state — nothing else will.** No skill in the HITL pipeline (`/factory-to-stories`, `/factory-check-security-design`, `/factory-triage`) sets an assignee, an iteration, or an in-progress state at creation — they deliberately leave a HITL-tagged Story, Bug, or wholly-HITL Feature untouched for a human to clear. So: **before** making any change while hand-delivering one, set `System.AssignedTo` to yourself, `System.IterationPath` to the current sprint, and `System.State` to `Active`. For a Feature, delivery starts with its first Story; while it is planned, the planning skills set its state. **When** the delivering PR merges and the work item is done, set `System.State` to `Resolved`/`Closed` yourself before ending the session. Skipping either half leaves the item unassigned, out of sprint, and looking untouched for the entire time it's actually being worked.

## Testing & the ratchet
> Seeded from the Factory standard and trimmed by a deliberate, recorded decision (2026-10-12): HotTea has no LLM, no external seams, no OS-specific code and no test substrate, so the lines about LLM contracts, seam real-IO, the platform matrix and disposable substrate are dropped. Full standard: <https://github.com/enateltd/factory-skills/blob/main/docs/testing-and-the-ratchet-standard.md> · the *why*: <https://github.com/enateltd/factory-skills/blob/main/docs/agentic-e2e-testing-principles.md>.

- **Assert on observable behaviour, not internal state.** Tests call the conversion function's public interface and check the page's visible result.
- **One tier applies.** Tier 1: fast local tests, run on every commit with `node --test`. Tier 2 and Tier 3 are not used, because there is no live LLM, no live substrate and no external seam.
- **The ratchet — every live defect pins a Tier 1 regression.** If a defect is found on the published site, its fix lands with a Tier 1 regression test that encodes the **class** of defect, not just the exact reproducer. A Story that surfaced from a live defect carries that test as an acceptance criterion.
- **Plan test coverage at the Feature level, not the Story level.** Each Feature declares at kickoff which tiers it needs.
