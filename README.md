# Conversational UI and Native Shells — Study

## The question

> How do native, cross-platform conversational UI applications — especially
> ones built with Tauri (Rust backend + web frontend) — structure frontend
> state management, multi-conversation/multi-context switching (e.g.
> swapping between different "workspaces" or agent personas), local file
> persistence (chat history, settings), and integration with backend agent
> processes or MCP servers?

This study exists in service of a concrete decision: whether to extend an
existing working Tauri 2 native shell into a multi-context native chat app
that can swap between several different backend products at runtime,
rather than building a new native shell from scratch. Sibling study:
[[../agent-harnesses]] covers runtime/backend harness behavior; this one
covers frontend/native-shell architecture.

## What we are looking at, repo by repo

Working checklist per entry:

- **Stack.** Tauri, Electron, or other — and why that matters for a
  build-fresh-vs-extend-existing-shell decision.
- **State management.** How is conversation/app state modeled and kept in
  sync between the native shell and its backend?
- **Multi-context switching.** Is there a "workspace" or "persona" concept
  that can be swapped at runtime, and how does the adapter/data layer
  change underneath it?
- **Local persistence.** What's on disk — SQLite, JSON, a KV store — and
  where?
- **MCP-client integration.** If the app is an MCP host, how does it
  discover, enable/disable, and invoke MCP servers?

## The design space at a glance

| Bet | Entry |
|---|---|
| Tauri + React; clean local-first multi-provider persistence via Rust backend | [Kaas](./kaas) |
| Tauri + SvelteKit; multi-agent delegation, long-term memory, MCP, "Skills" loading — closest feature match, early-stage | [openagent](./openagent) |
| Tauri **and** Electron dual-shipped; MCP host (stdio + SSE), per-tool enable/disable, real Electron→Tauri migration | [Dive](./dive) |
| Electron (comparison); mature local persistence (SQLite→PGlite), MCP-client + server marketplace | [5ire](./5ire) |
| Electron (comparison); workspace-based multi-context switching — closest existing pattern to "swap backend products" (verdict: shallow, see inquiry notes) | [anything-llm](./anything-llm) |
| Tauri 2 desktop + real Rust/Axum server, dual-backend (also ships a Next.js web path) sharing one `api-contract.yaml`; genuine per-workspace adapter swap, ACP/MCP/A2A surfaces | [routa](./routa) |
| Tauri 2 + Rust (Tokio) + React; spawns one isolated `codex app-server` process per workspace over JSON-RPC/stdio — small, low-adoption, but an architecturally honest process-level adapter-swap example | [open-vibe](./open-vibe) |
| Web app (Node/Express + MongoDB), no native shell — pinned anyway for its backend/file-persistence design: per-file-type pluggable storage strategy (`local`/`s3`/`firebase`/`azure_blob`/`cloudfront`, mixable per avatar/image/document) and a `skill`/`routes/skills.js` surface | [librechat](./librechat) |
| Tauri (genuinely, ~10MB vs. Electron's 150MB); desktop app is a thin webview shell pointed at one `server_url` in a JSON config — switching backend means editing config and restarting, not a runtime adapter swap. Re-pinned 2026-07-20 for a second look despite the earlier exclusion verdict below; license shows as "Other/NOASSERTION" via GitHub API, reconcile against the in-repo LICENSE before treating as MIT | [onyx](./onyx) |
| **Doesn't fit this study's thesis** — Next.js data-portal framework (AI-scaffolded open-data portals, CKAN/DKAN-adjacent), not a native chat shell, no Tauri/Electron, no multi-context backend-swap concept. Pinned 2026-07-20 at explicit user request, same "pinned anyway" precedent as librechat's row above — flagging the mismatch rather than silently treating it as in-scope | [portaljs](./portaljs) |

## Candidates — surfaced, not yet pinned

Tools worth a submodule pin but not yet walked in. Listed here so the
consideration is on record before the pin decision is made.

- **[AionUi](https://github.com/iofficeai/aionui)** (`iofficeai/aionui`,
  Apache-2.0, 31.3k★) — Electron desktop app (TypeScript/Node frontend +
  a local Rust backend service, "AionCore"/"Aionrs"); SQLite local
  persistence, no server uploads; macOS/Windows/Linux. Relevant on three
  of this study's five checklist axes at once: **multi-context switching**
  (independent parallel conversations with isolated context, plus a
  "Multi-Agent Mode" that auto-detects and drives CLI tools —
  Claude Code, Codex, **Hermes Agent**, and 13+ others — simultaneously),
  **MCP-client integration** ("MCP Unified Management" — configure a tool
  once, sync to all agents), and **local persistence** (SQLite). Positions
  as the open-source, cross-platform answer to Claude Cowork (macOS-only,
  Claude-exclusive). Caveats before pinning: it's **Electron, not Tauri**,
  so it lands in the comparison lane alongside 5ire/anything-llm rather
  than the Tauri-shell core question — but its multi-agent-CLI-integration
  model is the closest thing on this list to "swap between several backend
  products at runtime," and it explicitly namechecks Hermes Agent, which
  ties it to the anchor tree's Hermes work ([[Hermes-Agent-Colocation-and-Hackability]],
  [[Hermes-Agent-Multi-User-Team-Access]]). Highest star count of anything
  considered for this study by a wide margin.
- **[Claudable](https://github.com/opactorai/Claudable)** (`opactorai/Claudable`,
  MIT, ~4k★) — Next.js app-builder (Lovable alternative) with an Electron
  desktop variant; local SQLite for dev state, Supabase/Vercel for the
  production/deploy path. Same lane as AionUi above: an Electron shell that
  **auto-detects and drives multiple agent CLIs** — Claude Code, Codex CLI,
  Cursor CLI, Qwen Code, Z.AI GLM-4.6 — rather than owning the agent runtime
  itself. Relevant to this study's **multi-context switching** axis (which
  backend agent is in scope, swapped per project) and, indirectly, to the
  sibling [[../agent-harnesses]] study's tool/scoping question — though the
  caveat is that Claudable **wraps** harnesses rather than being one ("file
  operations occur through the connected CLI agent's terminal integration"),
  so it's a multi-harness *orchestrator* one level above the harnesses that
  study collects. Also note it's an app-*builder* (live preview, deploy),
  not a general chat shell — narrower product surface than AionUi.

## Sub-inquiries driving this reading pass

1. How does `Dive`'s Electron→Tauri migration inform an extend-existing-shell
   vs. start-fresh decision?
2. Does `anything-llm`'s workspace-switch model achieve a genuine swappable
   adapter per context (one adapter per workspace, swappable at runtime), or
   is it shallower than that?
3. How does `openagent`'s "Skills" loading work, and does it generalize to
   swapping an entire skillset when switching between unrelated backend
   products?

Notes go in `context-v/inquiry/`, cited by path
(`studies/conversational-ui-and-native-shells/<repo>/<file>:<line>`), not
paraphrased as prose.

## Excluded (verified, not just assumed)

- **`chatboxai/chatbox`** — confirmed Electron despite reputation; an open
  issue on the repo literally asks the maintainer to switch to Tauri.
- **`DrJonBrock/luke-desktop`** — considered and excluded: essentially a
  single-commit MCP-integration scaffold from Dec 2024, not actively
  developed; Dive and 5ire already cover MCP-client ground more robustly.
- **Msty** — confirmed closed-source/proprietary, not inspectable.
- **`CherryHQ/cherry-studio`** — confirmed Electron despite reputation as a
  "modern" client; wrong runtime for this study's Tauri focus.
- **Enconvo** — no verifiable open-source repo surfaced; appears closed.
- **`onyx-dot-app/onyx`** — original verdict (still the working assessment):
  genuinely Tauri (~10MB vs Electron's 150MB) but the desktop app is a thin
  webview shell pointed at one `server_url` in a JSON config; switching
  backend means editing config and restarting, not a runtime adapter swap.
  **Re-pinned as a submodule on 2026-07-20** for a second, closer look —
  see the design-space table above — despite this verdict standing;
  license needs reconciling (GitHub API shows "Other/NOASSERTION", this
  note originally said MIT).
- **`Austin-Patrician/multi-cli-studio`** — real Tauri repo, but adoption
  signals too thin to recommend confidently yet.
- **Open WebUI** (`open-webui/open-webui`) — the de facto market-standard
  chat UI, but verified as primarily a self-hosted web app: Python backend,
  SvelteKit frontend, server-side SQLite/Postgres persistence (multi-user
  by design, not per-user local files). It does have an official desktop
  wrapper, `open-webui/desktop` — Electron, "Early Alpha," architecturally
  a wrapper around the same web app rather than a ground-up native client.
  Not pinned: doesn't meet the native-shell bar Kaas/Dive/5ire/AnythingLLM
  set, though its MCP support (native since v0.6.31, Streamable-HTTP) and
  "Workspace" concept (model + knowledge + tools + prompt, bound and
  switchable — directly comparable to AnythingLLM's workspaces) are worth
  knowing about if the market-standard-UX question ever outweighs the
  native-shell one.

## Related

- [[../agent-harnesses]] — the sibling study for runtime/backend harness
  behavior
