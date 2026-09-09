# Beavermind fork of Cap

Runs at https://rec.beavermind.cloud. The only change from upstream is that sign-in
goes through Beavermind Identity (https://auth.beavermind.cloud) instead of
Google / email codes, so access is granted from the portal like every other
internal app.

## What differs from upstream

| File | Change |
|---|---|
| `packages/env/server.ts` | `BEAVERMIND_ISSUER`, `BEAVERMIND_CLIENT_ID`, `BEAVERMIND_CLIENT_SECRET` |
| `packages/database/auth/auth-options.ts` | `beavermindProvider()` (generic OIDC via next-auth); when configured it is the **only** provider |
| `apps/web/app/Layout/AppProviders.tsx`, `apps/web/utils/public-env.tsx` | `beavermindAuthAvailable` flag for the client |
| `apps/web/app/(org)/login/form.tsx` | auto-starts the Beavermind sign-in; one button as fallback; no email form |
| `apps/web/app/(org)/signup/page.tsx` | redirects to `/login` (accounts come from Identity) |
| `apps/web/app/s/[videoId]/_components/AuthOverlay.tsx` | share-page sign-in button uses Beavermind |
| `.github/workflows/docker-build-web.yml` | amd64 image on push to `beavermind`, tagged `bm-<sha>` |

`GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` stay set in production: the
organization's Google Drive storage integration authorizes with them directly and
does not go through next-auth.

## Branches

- `main` tracks `CapSoftware/Cap` and is never deployed.
- `beavermind` is `main` at the last commit we validated plus the patch above.
  It is the workflow's trigger and the default branch of this fork.

## Upgrading

```bash
git fetch upstream               # git remote add upstream https://github.com/CapSoftware/Cap.git
git checkout beavermind
git rebase upstream/main         # resolve conflicts in the files listed above
git push --force-with-lease
```

The push builds `ghcr.io/beavermindai/cap-web:bm-<sha>`. On the VPS, change the
`image:` tag in `/home/ruben/cap/docker-compose.yml` to that tag and
`docker compose up -d cap-web`. Check `packages/database/migrations` in the diff
first: a new migration runs against the live MySQL on boot and is one-way, so
take a `mysqldump` before moving across one.
