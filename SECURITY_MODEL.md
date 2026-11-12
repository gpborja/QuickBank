# QuickBank threat model

Written from the code (gpborja/QuickBank) and the owner's answers (2026-11-12). Points marked *(draft)* are Devin's reading of the code; points marked **(owner)** come from the repository owner.

## Owner decisions

- **(owner)** Purpose: QuickBank is a **reference/teaching app**: every insecure pattern should be flagged and rated on its merits. "It's only a demo" is not a reason to lower a finding.
- **(owner)** Hosting is **not decided yet**: assume it may be served from any origin, including a shared one (e.g. GitHub Pages under `gpborja.github.io`). Hosting/origin-dependent findings stay in scope.
- **(owner)** **Accepted risk**: a visitor tampering with their own `localStorage` or bypassing client-side checks in their own browser (fake balances, overdraft bypass). Findings that only matter for that attacker are low priority or not worth fixing.

## System overview and trust boundaries

QuickBank is a single static file, `index.html` (markup, inline `<style>`, inline `<script>`, lines 96-267). The README describes it as "a front-end demo only: no authentication, no backend, no real money". There is no server-side code, no build step, no third-party scripts, no network requests.

Trust boundaries:
- **Visitor's browser ↔ page origin**: all state lives in `localStorage` under the key `quickbank.v1` for whatever origin serves the file. Any other page served from the same origin can read and write it.
- **Stored data ↔ DOM**: values read back from `localStorage` (`load()`, line 116) are rendered with template strings into `innerHTML`.
- **Hosting**: no infrastructure repository; README suggests opening the file directly or `python3 -m http.server 8080`. **(owner)** Deployment target not decided.

## Components

| Component | Code it covers |
|---|---|
| **UI Rendering** | `renderAccounts`, `renderSelects`, `renderHistory`, `render`, `setOp`, `showMsg`, `esc` (index.html ~147-208); all `innerHTML` sinks |
| **Transaction Logic** | `parseAmount`, `addTxn`, `submit`, overdraft check, transfers incl. `__external__` payee (index.html ~125-145, 210-244) |
| **Persistence** | `KEY`, `seed`, `load`, `save`, Reset demo handler (index.html ~97-123, 260-264) |
| **Hosting** | how `index.html` is served: page origin, HTTP headers (CSP, framing), static server; no code in repo |

## Entry points and who can reach them

- Form inputs: amount, memo (maxlength 60), payee (maxlength 40), search, account selects. Reached by the visitor themself.
- `localStorage['quickbank.v1']`: reached by the visitor (devtools) and by any script running on the same origin.
- The page itself: any website can link to or frame it (no framing protections visible; depends on Hosting).
- No URL parameters, `postMessage`, fetch, or server endpoints are read.

## Assets and data sensitivity

- Account balances and transaction history: fictional demo data (`seed()`), no real money. Integrity against the visitor themself is accepted risk **(owner)**; integrity against other origins/pages is in scope.
- Masked account numbers (`••••4821`, `••••9037`): fake.
- Free-text memo/payee entered by the visitor: low sensitivity, may contain whatever a visitor types.
- The page origin itself: script execution on it would also expose anything else on the same origin (relevant if hosted on a shared origin such as `<user>.github.io`).

## Attacker profiles

- **Same-origin co-tenant** (in scope, hosting undecided **(owner)**): another page/app on the same origin (e.g. other GitHub Pages projects under one user site) that writes crafted data into `quickbank.v1`, turning stored fields into script injection when rendered.
- **Remote web attacker** (in scope): a malicious site that links to or frames QuickBank (clickjacking the Reset or Transfer buttons). Limited impact without real money.
- **The visitor themself** — accepted risk **(owner)**: can edit `localStorage` or run JS to fake balances or bypass overdraft checks.

## Existing controls

- `esc()` HTML-escapes `name`, `desc`, `memo`, payee-derived descriptions before `innerHTML` insertion. Not applied to `a.number`, `a.id` (option values, `data-id`), or computed numbers.
- `parseAmount()` strict regex `^\d+(\.\d{1,2})?$`, integer cents, `Number.isSafeInteger`, positive only.
- Overdraft check `cents > src.balance` for withdraw/transfer; same-account transfer rejected.
- `load()` checks only that `accounts`/`txns` are arrays; no per-field validation of stored data.
- `maxlength` on memo/payee inputs (client-side only).

## Out of scope / accepted risk

- **(owner)** Visitor tampering with their own `localStorage` or bypassing client-side checks.
- No authentication or authorization exists by design.

## Open questions

1. Final hosting target (and therefore HTTP headers such as CSP / `frame-ancestors`) is undecided.
2. Assets the owner considers most critical were not named beyond the above.

## Finding tags

- `Component`: one of the components above.
- `OWASP`: OWASP Top 10 (2021) category, e.g. `A03:2021-Injection` **(owner)**.
