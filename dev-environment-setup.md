# OrchardGuard V2 — Development Environment Setup

> **Status.** Draft for Phase 1. Revise as tooling choices are finalized during Foundation work.
> **Audience.** The project author and Claude Code sessions. Written to get a new working environment from zero to "can run V2 locally" with no ambient knowledge required.
> **Companion to.** `orchardguard-v2-plan.md` Section 9 (Technology Decisions).

---

## 1. Prerequisites

Install these before anything else. Versions match Section 9.

| Tool | Version | Install |
|---|---|---|
| Node.js | 22 LTS | Via `fnm` or `nvm`; pinned in `.nvmrc` |
| pnpm | 9+ | `npm install -g pnpm` or `corepack enable pnpm` |
| Docker Desktop | Recent stable | [docker.com](https://www.docker.com/) |
| Git | 2.40+ | System package manager |
| A code editor | — | VS Code recommended; extensions listed in §7 |

On macOS, `brew install fnm pnpm docker git` covers most of this. On Windows, WSL2 + Ubuntu is strongly recommended; running V2's toolchain on native Windows is possible but painful.

Verify:

```
node --version   # v22.x.x
pnpm --version   # 9.x.x
docker --version # 24.x+
git --version    # 2.40+
```

## 2. Clone and Install

```
git clone <repo-url> orchardguard-v2
cd orchardguard-v2
fnm use            # picks up .nvmrc → Node 22
pnpm install
```

First install takes a couple of minutes. Subsequent installs are seconds.

## 3. Local Database

V2 uses Postgres 16. Locally, Docker Compose runs it.

```
docker compose up -d postgres
```

This starts Postgres on `localhost:5432` with credentials from `.env.local` (see §4). Data persists in a Docker volume between restarts. To reset completely:

```
docker compose down -v     # destroys the volume; next `up` starts clean
```

Verify Postgres is running:

```
docker compose ps
```

Should show `postgres` with status `running (healthy)`.

## 4. Environment Variables

Copy the template:

```
cp .env.example .env.local
```

Open `.env.local` and fill in values. Required keys at Phase 1:

| Key | What it is | Where to get it |
|---|---|---|
| `DATABASE_URL` | Postgres connection string | Defaults work for local Docker Postgres |
| `NEXTAUTH_URL` or `CLERK_PUBLISHABLE_KEY` | Auth config | Clerk dashboard (dev instance) once auth is wired |
| `CLERK_SECRET_KEY` | Clerk server key | Clerk dashboard |
| `RESEND_API_KEY` | Email sending | [resend.com](https://resend.com) dev API key |
| `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET` | Cloudflare R2 | Cloudflare dashboard |
| `SENTRY_DSN` | Error tracking | Sentry dev project |

Keys that are optional at Phase 1:

| Key | When needed |
|---|---|
| `TWILIO_*` | When SMS alerts are wired (Phase 3) |
| `AXIOM_TOKEN` | When structured logging is wired (Phase 4) |
| `WEATHERLINK_*` | When station integration goes live (2027, per Section 9.10) |

**Never commit `.env.local`.** The `.gitignore` already excludes it. Verify with `git status` that `.env.local` is not staged before every commit.

## 5. Run Migrations

Drizzle generates and runs migrations.

```
pnpm db:migrate
```

This applies every migration in `db/migrations/` to the local Postgres. Should be idempotent. Running twice has no effect.

To generate a new migration after editing `db/schema.ts`:

```
pnpm db:generate
```

Inspect the generated SQL in `db/migrations/<timestamp>_<name>.sql` before committing.

## 6. Seed Data

```
pnpm db:seed
```

This populates:

- One farm ("Dev Farm")
- One orchard with test coordinates (Toronto area)
- One block with one planting
- A test user linked to the farm as owner
- ~5 catalogue products for smoke-testing the advisor flow

Seed data is for development only. Production farms are created through the invitation flow.

To reset seed:

```
pnpm db:reset    # drops all data and re-runs migrations + seed
```

## 7. Run the App

```
pnpm dev
```

Next.js starts on `http://localhost:3000`. First load compiles; subsequent loads are hot-reloaded.

Log in as the seed user. Credentials are printed by `db:seed`.

## 8. Recommended Editor Setup

VS Code extensions:

- **ESLint** — lints on save
- **Prettier - Code formatter** — formats on save
- **Tailwind CSS IntelliSense** — class name completion
- **Drizzle ORM** — schema inspection
- **SQLTools** + **SQLTools PostgreSQL Driver** — inspect local Postgres without leaving the editor
- **Vitest** — test running from the editor

Settings to add to workspace (`.vscode/settings.json`):

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "typescript.tsdk": "node_modules/typescript/lib"
}
```

## 9. Running Tests

```
pnpm test             # unit + integration + fixture tests via Vitest
pnpm test --watch     # re-run on change
pnpm test:e2e         # end-to-end tests via Playwright
pnpm test:fixtures    # just the model fixture tests
```

CI runs `pnpm test` and `pnpm test:e2e` on every PR. Fixture tests are part of `pnpm test`; they block merge if they fail on a Tier-1 model (Section 6.7).

## 10. Common Workflows

**Add a database column:**

1. Edit `db/schema.ts`
2. `pnpm db:generate` — inspect the migration SQL
3. `pnpm db:migrate` — apply locally
4. Update any affected Drizzle queries
5. Add test coverage
6. Commit schema, migration, and code together

**Add a rule:**

1. Create `lib/rules/<category>/<slug>.ts` per the rule authoring template
2. Create `lib/rules/<category>/<slug>.test.ts` with positive, negative, and edge cases
3. Register the rule in `lib/rules/registry.ts`
4. Run `pnpm test` — new tests should pass
5. Verify the rule evaluates via the advisor in dev

**Add a model:**

1. Create `lib/models/<slug>/model.ts` with citation header
2. Create `lib/models/<slug>/fixtures.json` with published fixtures
3. Create `lib/models/<slug>/model.test.ts` with fixture tests + unit tests
4. Register in `lib/models/registry.ts`
5. For Tier-1 models: merge is allowed when fixtures pass, but `advisor_enabled` flag stays false until scenario replay review lands

**Pull remote changes:**

```
git pull
pnpm install           # in case package.json changed
pnpm db:migrate        # in case new migrations landed
pnpm dev
```

## 11. Troubleshooting

**`pnpm install` fails with Node version error.** Run `fnm use` first. If that doesn't pick up the version, check `.nvmrc` exists and contains `22`.

**`pnpm db:migrate` fails with "database does not exist".** Run `docker compose up -d postgres` and wait 10 seconds for health check to pass.

**Next.js build errors about missing types.** Run `pnpm install` again; occasionally TypeScript types fall out of sync after a branch switch.

**Clerk sign-in loops.** Check that `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` matches the `CLERK_SECRET_KEY` environment in the Clerk dashboard — using a production key locally causes loops.

**Drizzle complains about schema drift.** Either run `pnpm db:migrate` (you forgot to apply a teammate's migration) or `pnpm db:generate` (you changed schema without generating a migration).

**Local Postgres won't start.** Port 5432 is probably in use by a local Postgres installation. Stop it (`brew services stop postgresql` on macOS) or change the port in `docker-compose.yml`.

**Vitest complains it can't find a test.** Check the file is named `*.test.ts` and is not excluded by `vitest.config.ts`.

## 12. Where to Go Next

- **For scope and posture:** read `orchardguard-v2-plan.md` Sections 1 and 7.
- **For the data model:** `orchardguard-v2-plan.md` Section 3.
- **For the rule engine:** `orchardguard-v2-plan.md` Section 4.
- **For the current work list:** `orchardguard-v2-plan.md` Section 10.
- **For domain terminology:** `GLOSSARY.md`.
- **For the catalogue state:** `catalogue-tracker.md`.

---

> **End of setup guide.** Open a PR if anything here is stale.
