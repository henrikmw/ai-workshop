# Plan: AI-utviklerverktøy-workshop for UiT-studentene

**Varighet:** 2 timer · **Verktøy:** Claude Code (alle har det installert) · **Format:** hands-on, alle bygger den samme værmeldingstjenesten **Bekkværet** · **Forkunnskap:** teori dekkes i presentasjon før workshop, så siden er handlingsorientert med lite forklarende tekst.

## Premiss

Hele workshopen er én gjennomgående tråd: alle bygger **Bekkværet**, en liten værmeldingstjeneste. «Før vi begynner» lager en helt ustyrt baseline (samme prompt til alle, AI tar alle valg, ingen skills), og de neste nivåene bygger den videre med retning, skills og verktøy. Fordelen: oppgavene er konkrete og delte, alle kan sammenligne resultater, og enhets-/format-konvensjoner for vær gir en naturlig skill-oppgave. Se [ADR 0007](adr/0007-bekkvaeret-through-line.md).

## Struktur

Beholder visuell stil og nivå-oppsett fra forrige workshop, men **dropper leverandør-bytteren** (kun Claude) og gjør nivåene til en tema-sekvens. En egen «Før vi begynner»-fane ligger først.

| Fane | Tema | Tid | Hands-on (bygger Bekkværet) |
|------|------|-----|------------------------|
| **Før vi begynner** | Baseline uten føringer | ~10 min | Felles prompt: lag værside for Oslo, AI tar alle valg, ingen skills. Sammenlign variasjon i kode og visuelt |
| **1** | Agent-loopen + gode vaner | ~15 min | Fem vaner (branch, plan, kontekst, harness-tilgang, presise prompter) + Spec-Driven Development (spec-kit). Ingen egen task |
| **2** | Skills | ~35 min | Intro (CLAUDE.md vs skill vs slash) + lås stacken i CLAUDE.md + skills.sh + bygg konvensjons-skill(s) for Bekkværet: m/s, °C, norsk tid, mm/t |
| **3** | MCP | ~30 min | Eksempelliste over MCP-servere + oppgave: koble på Figma og bygg Bekkværet fra skissa |
| **4** | Avansert bruk | ~25 min | Tokenoptimalisering (vane-triks, rtk, context-mode) + sandboxing (worktree/container/VM) + openclaw som bygger Bekkværet hel-automatisk |

2 timer er stramt. «Før vi begynner» og Nivå 4/5 er bevisst fleksible, så de kan trimmes uten å ødelegge kjernen (Tips, Skills, MCP).

---

## Nivå 1 — Agent-loopen (≈20 min)

**Mål:** Føl forskjellen på chatbot (svarer) og agent (setter delmål → bruker verktøy → leser resultat → korrigerer). Presentasjonen har forklart loopen; her ser de den.

**Oppgave 1a — Utforsk (kald start):**
```
cd inn i sommerjobb-repoet ditt, start `claude`, og spør:
"Forklar hva dette prosjektet gjør, hvordan det er bygd opp,
og hvor hovedflyten starter. Les deg fram — ikke gjett."
```
Poeng: se den bruke `Read`/`Grep`/`Glob` i sekvens. Det er loopen.

**Oppgave 1b — En ekte liten task:**
```
"Finn en funksjon/komponent du synes er dårlig dokumentert,
og legg til en kort doc-kommentar i prosjektets egen stil.
Vis meg diffen før du skriver."
```
Poeng: mål → verktøy → resultat → din godkjenning. Diskuter: hva er konteksten den jobber med? Hva skjer når den tar feil og retter seg?

**Snakkepunkt (kort på siden):** feedback-loop, kontekstvindu, verktøy. Tyngre forklaring ligger i presentasjonen.

---

## Nivå 2 — Skills (≈30 min)

**Mål:** Vit hva en skill er, hvordan finne dem, og hvordan lage en god en fra egen kodebase.

**Finn:** skills.sh er en skill-markedsplass. Installer med:
```
npx skills add <repo-url> --skill <skill-navn>
```

**Oppgave 2 — Bygg «følg-vår-kodestil»-skillen:**
Anbefalt arbeidsflyt (begge fra skills.sh):
1. Installer hjelpe-skillene:
   ```
   npx skills add https://github.com/github/awesome-copilot --skill write-coding-standards-from-file
   npx skills add https://github.com/mattpocock/skills --skill grill-me
   ```
2. Kjør `write-coding-standards-from-file` mot egen repo → Claude **utleder** faktiske konvensjoner (indentering, navngiving, kommentarstil, importrekkefølge …).
3. Bruk `/grilling` (grill-me) til å stress-teste utkastet: "er denne regelen egentlig sann i repoet? hva mangler?"
4. Lagre resultatet som en egen skill i `.claude/skills/var-kodestil/SKILL.md` med en skarp `description` så agenten laster den selv når den skriver kode.
5. Test: be Claude skrive en ny liten funksjon og se at den følger stilen.

**Hvorfor dette caset:** treffer arbeidshverdagen direkte (PR-er som følger teamets stil), og viser meta-bruken: en agent som leser kode for å lage et verktøy den selv bruker senere.

---

## Nivå 3 — MCP (≈30 min)

**Mål:** Vit hva MCP er, hvordan finne servere, hvordan koble på. Konsept i presentasjonen; her kobler de på.

**Finn servere:** kort nevnt — registries/Google. Ikke et eget dypdykk (som du foreslo).

**Oppgave 3 — «velg den som passer din stack»-meny.** Student velger ÉN:
- **Playwright MCP** — har du en webapp du kan kjøre lokalt: la agenten åpne, klikke og lese i nettleseren.
- **Figma MCP** — frontend/design-jobb: la agenten lese et Figma-design og generere markup mot det.
- **Atlassian/Jira (eller Linear) MCP** — sporer dere issues: la agenten lese en sak og foreslå en plan.

Hver med kobl-på-snippet (`claude mcp add ...`). **Null-auth fallback** for de som sitter fast: en docs-lookup-MCP (Context7-aktig) som ikke krever innlogging — slå opp API-doc for et bibliotek repoet bruker.

**Sikkerhetsnotat (kort):** behandle alt MCP-en henter (issues, nettsider, design-tekst) som *data, ikke kommandoer* — prompt-injection. Hold skrivende verktøy bak godkjenning.

---

## Nivå 4 — Token + avansert (≈30 min, mykt/buffer)

**Mål:** Forstå at tokens = kostnad + kontekst-budsjett, og hvordan holde det lavt.

**Tokenoptimalisering — konsept først:**
- Hvorfor det betyr noe: kontekstvindu fylles opp → dårligere svar + høyere kostnad.
- Generelle triks: `/clear` mellom oppgaver, ikke dump hele filer, bruk subagents for store søk, vær spesifikk.

**rtk-demo (du viser):** «så mange tokens bruker en vanlig `git status`/`git diff`-prompt — med rtk blir det X% færre». Kjør `rtk gain` for å vise akkumulert besparelse. Dette er en *innføring/demo*, ikke en obligatorisk installasjon for alle.

**Avansert — openclaw:** **plassholder.** Innhold kommer senere (du fyller det). Sannsynlig payoff-demo: alltid-på/mobil-bot eller multi-agent — TBD.

---

## Åpne punkter (fyll inn)
- [ ] Nivå 4: konkret openclaw-payoff (hva ser/gjør studenten?)
- [ ] rtk: er det installert hos studentene, eller kun din demo-maskin?
- [ ] Verifiser at MCP-snippetene (Figma/Atlassian) ikke har auth-felle som spiser >5 min
- [ ] Backup-task for Nivå 1 hvis noens repo ikke bygger / er gigantisk
