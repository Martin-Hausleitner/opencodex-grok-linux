---
name: opencodex-grok-linux
description: "Die Codex-CLI als Grok-Harness betreiben: OpenCodex (ocx) routet den `codex`-Coding-Agent auf xAI grok-4.5, statt gegen OpenAI. Kein direkter API-Aufruf im Code — das ganze Codex-CLI läuft auf Grok 4.5 High. Zusätzlich: Superpowers + Skill-Set. Trigger: OpenCodex, ocx, Grok in Codex, grok CLI harness, grok-4.5 harness, superpowers."
risk: safe
metadata:
  type: setup
  verified_on: "2026-07-31 (macOS-Referenzsystem, ocx v2.7.36, LIVE: xai oauth active, model=xai/grok-4.5)"
  tags: [opencodex, ocx, codex, grok, xai, linux, harness, superpowers, skills]
---

# Codex-CLI als Grok-Harness über OpenCodex (Linux)

**Worum es geht:** Der **`codex`-Coding-Agent** (die Codex-CLI, „Codex Ultra") ist der Harness — das
Werkzeug, das Dateien liest, Befehle ausführt, plant. **OpenCodex (`ocx`)** hängt sich als lokaler
Proxy dazwischen und lässt diesen Codex-Harness auf **xAI grok-4.5** laufen statt auf OpenAI.

Es wird also **die grok-CLI-Erfahrung genutzt** — der volle Codex-Agent, nur mit Grok als Gehirn.
**Kein** direkter `api.x.ai`-Call im eigenen Code; das erledigt der Proxy transparent.

```
codex (Harness)  ──►  ocx proxy (:10100)  ──►  grok-4.5
     du arbeitest hier          transparent          das Modell
```

## Verifiziert (LIVE, Referenzsystem 31.07.2026)
- ocx **2.7.36**, Proxy **:10100**, Default-Provider **xai**, Account `xai oauth active`
- **Grok 4.5 High** = `grok-4.5` + `model_reasoning_effort = "high"` · 500k Kontext
- Codex-Bindung real gesetzt: `model = "xai/grok-4.5"`, `openai_base_url = "http://127.0.0.1:10100/v1"`

---

## Installation auf Linux

### 1. OpenCodex holen und auf den PATH
```bash
git clone https://github.com/just-every/code ~/code/opencodex
cd ~/code/opencodex && npm install
ln -sf "$PWD/bin/ocx" ~/.local/bin/ocx
ln -sf "$PWD/bin/ocx" ~/.local/bin/opencodex
export PATH="$HOME/.local/bin:$PATH"     # dauerhaft in ~/.bashrc/~/.zshrc
ocx --version
```

### 2. xAI-Login (Browser-OAuth) — liefert die Grok-Tokens
```bash
ocx login xai
ocx account list      # muss  xai oauth … active  zeigen
```
Der Token wird von ocx in `~/.opencodex/` verwaltet — **nie** in Repo/Gist.

### 3. Den Codex-Harness auf Grok umlenken
```bash
ocx init              # Provider = xai; injiziert die Codex-Config
ocx start             # Proxy :10100, synct Modelle in die Codex-Config
ocx codex-shim install  # `codex` startet den Proxy automatisch mit — nahtlos
ocx service           # Dauerbetrieb (überlebt Logout)
ocx status            # „Proxy: running", Default provider: xai
```
Ab jetzt ist **jeder `codex`-Aufruf** ein Grok-Harness — normal `codex` starten, fertig.

### 4. Auf Grok 4.5 High festnageln
`~/.codex/config.toml`:
```toml
model = "xai/grok-4.5"
model_reasoning_effort = "high"
openai_base_url = "http://127.0.0.1:10100/v1"
```

### 5. Beweisen, dass der Codex-Harness auf Grok läuft
```bash
ocx status | grep -i provider              # Default provider: xai
codex "Welches Modell steuert dich gerade?"  # interaktiv — Antwort nennt Grok
ocx debug usage                            # Requests an api.x.ai
```
**Fertig erst, wenn** ein normaler `codex`-Lauf nachweislich über Grok geht.

---

## xAI-Modelle über ocx
| Modell | Kontext | Reasoning |
|---|---:|---|
| **grok-4.5** (Default) | 500k | low/medium/**high** |
| grok-4.3 | 1M | — |
| grok-4.20-0309-reasoning | 1M | reasoning |

---

## Zusätzlich: Superpowers + Skill-Set
```bash
rsync -a ~/.claude/skills/  <linux-host>:~/.claude/skills/
```
- **`using-superpowers`** — lädt vor jeder Antwort den passenden Skill.
- **`skills-load-first`** — Skill-Discovery am Konversationsanfang.
Kern-Skills, die mitmüssen: `integrative-deep-research`, `manager-standing-rules`,
`report-to-github`, `operator-capture-openspec-qg`, `notebooklm-mcp-cli`, `knight`, dieser Skill.

---

## Zurück / Troubleshooting
```bash
ocx restore     # Codex wieder direkt auf OpenAI
ocx stop        # Proxy stoppen + Codex nativ
ocx doctor      # Umgebung/Netz diagnostizieren
```

## Fallstricke
- **`codex -p` ist verboten** — immer interaktives `codex`.
- Tokens niemals committen — bleiben in `~/.opencodex/`.
- `ocx start` synct Modelle; nach manuellen Edits `ocx sync` / `ocx sync-cache`.
