# ADR 0008 — Harness-velger (Claude Code + Copilot + Gemini CLI)

**Status:** Vedtatt · **Dato:** 2026-06-29 · **Oppdatert:** 2026-09-10 (Gemini CLI lagt til)

## Kontekst
Workshopen var låst til Claude Code. Deltakere bruker ulike agenter, og vi vil at tekster og kodesnutter skal tilpasse seg valgt verktøy (som leverandør-bytteren i den opprinnelige workshopen).

## Beslutning
- En **dropdown øverst til høyre** velger AI-harness. `body[data-harness="..."]` styrer visning.
- Harness-spesifikt innhold ligger i `.ho`-elementer (`.ho.claude`, `.ho.copilot`, `.ho.gemini`); skjult med `display:none`, vist med `display:revert` for valgt harness. `revert` gjør at samme mekanisme funker både for inline-spans (CLI-navn, filnavn, stier) og blokker (hele kodeeksempler).
- **Claude Code er default.** Valget huskes i `localStorage` (`ws_harness`).
- **Scope nå: Claude Code + GitHub Copilot + Gemini CLI.** Codex er research-verifisert (kommandoer, instruksjonsfil, skills-mappe, MCP) og kan legges til senere ved å utvide dropdownen og legge til `.ho.codex`-varianter (CSS-hooken finnes allerede).
- **Antigravity CLI ble vurdert og forkastet (2026-09-10).** Antigravity var kortvarig lagt til, men fjernet igjen: Figma-MCP-oppgaven i nivå 3 er en sentral del av workshopen, og Figma har ikke godkjent Antigravity CLI for OAuth dynamic client registration — innlogging feiler med «OAuth app with client id … doesn't exist» ([antigravity-cli #496](https://github.com/google-antigravity/antigravity-cli/issues/496), bekreftet av Figma på forumet august 2026, ingen løsning eller tidslinje). Verifisert kommandosyntaks (`agy`, `AGENTS.md`, `.agents/skills`, `agy mcp add --type <type> <navn> <kommando/url>` — flagg *før* navnet) kan gjenbrukes hvis Figma hvitelister Antigravity senere.

## Harness-spesifikke punkter som byttes
CLI-start, instruksjonsfil (`CLAUDE.md` vs `AGENTS.md` vs `GEMINI.md`), skills-mappe (`.claude/skills` vs `.github/skills` vs `.gemini/skills`, global `~/.copilot/skills` / `~/.gemini/skills`), MCP-koble-på (`claude mcp add` vs `/mcp add` / `~/.copilot/mcp-config.json` vs `gemini mcp add`), planmodus (Shift+Tab vs `/plan`), Figma-MCP-oppkobling.

## Konsekvenser
- Bredere relevans uten å fragmentere strukturen; delt innhold (baseline, vaner, skills.sh, tokenoptimalisering, sandboxing) forblir felles.
- openclaw-seksjonen er bevisst holdt som en Claude Code-rute (openclaw kjører `claude-cli` som runtime); ikke byttet med harness.
- Mer innhold å vedlikeholde per harness-spesifikt punkt. Verifiser kommandoene mot installert versjon før workshop, da CLI-ene endrer seg raskt.
