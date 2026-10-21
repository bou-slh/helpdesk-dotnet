# RapidsolDestek

Turkish/English helpdesk product (portal + agent + admin panels) built from scratch in C#/ASP.NET Core.
osTicket (separate checkout) is a **domain-model reference only** — never copy its code (GPLv2).

## Golden rule: the mockups are the spec

`mockups/` contains 72 static TR/EN pages (portal 12, agent 18, admin 42). The shipped product must look
and behave exactly like them — **every visible component must work**, including controls that are dead in
the static HTML. The roadmap, parity checklists, and canonical sample data live in **`mockups/ROADMAP.md`**;
markup/i18n conventions live in `mockups/CONVENTIONS.md`. A page is done when its ROADMAP checklist is green
in TR+EN, light+dark — never when it merely renders.

## Stack

- .NET 10 · ASP.NET Core MVC, Razor areas `Portal` / `Agent` / `Admin`
- EF Core + PostgreSQL (migrations only via `dotnet ef` — never hand-edit `Migrations/`)
- Hangfire (jobs: mail fetch, SLA sweeps, retention) · SignalR (live board) · MailKit/MimeKit (email)
- xUnit (`tests/RapidsolDestek.Tests`) · Playwright E2E (`tests/RapidsolDestek.E2E`)

## Layout

```
src/RapidsolDestek.Domain          entities, domain services, no framework deps
src/RapidsolDestek.Infrastructure  EF Core, file store, email, external services
src/RapidsolDestek.Web             MVC app; Areas/{Portal,Agent,Admin}
tests/                             unit + E2E
mockups/                           the binding spec (do not edit without S0-style sign-off)
```

## Commands

- Run: `docker compose up -d db` then `dotnet watch --project src/RapidsolDestek.Web`
- Build: `dotnet build` · Tests: `dotnet test`
- Full stack: `docker compose up --build`

## AI tooling (committed in this repo — use it)

- **Skills**: `/port-page <mockup>` (the S5–S7 unit of work: port → i18n audit → visual diff), `/add-entity <Name>` (S3), `/add-setting <key>` (S7), `/verify` (gate before every commit), `/run` (db + watch + open).
- **Subagents** (`.claude/agents/`): `page-porter`, `schema-reviewer`, `i18n-auditor`, `visual-differ`, `test-writer`. For bulk page porting, fan out one page-porter per page in isolated worktrees; audit + diff each page before merge. All agents treat `mockups/ROADMAP.md` + `mockups/CONVENTIONS.md` + this file as the shared contract.
- **MCP** (`.mcp.json`): `playwright` (browser for visual verification against localhost and the mockup twins), `postgres` (inspect schema/data on the compose db).
- **Hooks** (`.claude/settings.json`): C# edits auto-format + build-check; view/mockup/resx edits run `tools/check-i18n.mjs`; edits under `Migrations/` and `git push` to main are blocked.
- **CSS parity rule**: `src/RapidsolDestek.Web/wwwroot/css/{base,portal,admin}.css` must stay byte-identical to `mockups/assets/css/` — product-only styles go in `wwwroot/css/app.css`.
- **i18n converter**: `dotnet run tools/i18n-convert.cs -- shared` (mockup dicts → SharedResources resx) and `-- page <mockup.html> <viewPath>` (PAGE_I18N → per-view resx).

## Conventions

- Two auth principals: Customer (portal) and Staff (agent/admin), separate cookies (`rd.customer` / `rd.staff`). Admin = staff + `Admin` role + mandatory TOTP 2FA (`AdminOnly` policy).
- i18n: every UI string via `IStringLocalizer` + resx (tr default, en); no hardcoded literals in views.
  Key names mirror the mockup `data-i18n` keys.
- Every mutation writes an AuditEvent (EF interceptor). Status enum and seed data must match the canon
  table in `mockups/ROADMAP.md` §2.
- Effort approval (Efor Onayı) is the flagship flow — state machine `none→pending→approved|rejected→(revise→pending)`.

## AI triage (extension in progress)

- AI-assisted triage is built as a switchable module inside this solution. The spec, v1 scope, conventions
  and build order (slices T0–T7) live in **`docs/ai-triage/ANALYSIS.md`** — read it before any triage work.
- v1 rule: the AI never changes a ticket or contacts a customer on its own; an agent action is always the trigger.
- Work one slice at a time, plan before code, `/verify` before every commit.
