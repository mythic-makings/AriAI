# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

A single static HTML file (`index.html`, ~330 lines) deployed to Vercel. No framework, no build step, no package manager, no tests. Inline `<style>` and `<script>` — vanilla everything. Edits go straight to `index.html`; commits to `main` auto-deploy.

The chat UI is intentionally a placeholder: the bot reply is hardcoded (`"I'm still learning. Full responses coming soon."`) with a 1-second simulated typing delay. Real responses will require a backend, which is why Supabase is already linked but unused.

## Commands

### Deploy

- **Auto-deploy**: every `git push` to `main` triggers a Vercel production build via the GitHub integration. The stable alias `https://ariai-weld.vercel.app` always points to the newest production build.
- **Manual deploy** (rarely needed): `vercel --prod --yes` from the repo root.
- **List recent deployments**: `vercel ls ariai`.

### Supabase

- The project is linked to ref `tzbgwvdmjabtgmbovcqo` (East US, region: us-east-1). Link state lives in `supabase/.temp/`; credentials come from `.env`.
- **Re-link after a fresh clone**:
  ```bash
  SUPABASE_DB_PASSWORD="$(grep ^SUPABASE_DB_PASSWORD= .env | cut -d= -f2-)" \
    supabase link --project-ref "$(grep ^SUPABASE_PROJECT_REF= .env | cut -d= -f2-)"
  ```
- No migrations yet. When added: `supabase migration new <name>` to create, `supabase db push` to apply.

### No-ops

There is no `package.json`, no test runner, no linter, no build script. Do not run `npm install`, `npm test`, `npm run build`, or similar — there's nothing to install or build. The "build" is just the HTML file itself.

## Architecture notes (non-obvious)

- **Vercel project is `ariai` (lowercase); GitHub repo is `mythic-makings/AriAI` (mixed case).** Use the right case for each — `vercel ls`, `vercel deploy`, etc. expect lowercase.
- **Auto-deploy requires the Vercel GitHub app on the `mythic-makings` org.** If the integration breaks (repo move, app uninstall), reinstall at https://github.com/apps/vercel, then `vercel git connect` from the repo root.
- **`STITCH_API_KEY` is duplicated by design.** The human-facing copy lives in `.env`; the operational copy lives in `~/.claude.json` under `mcpServers.stitch.env.STITCH_API_KEY` (the Stitch MCP server doesn't auto-load `.env`). If rotated, update both. `.env` is the source of truth.
- **`.env` and `.vercel/` are gitignored** and must stay that way — both contain credentials or deploy state.

## Working in this repo

Because there's no build or test step, the loop is just: edit `index.html` → commit → push → wait ~10s for Vercel to publish. Verify visually at the live URL or via `vercel ls ariai`.

When the chat is wired to a real backend, this file should be updated to reflect the actual request/response flow and where API keys for the LLM provider live.
