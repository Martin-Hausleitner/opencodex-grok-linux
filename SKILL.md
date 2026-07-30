---
name: opencodex-grok-linux
description: "Codex Ultra als Harness mit Grok 4.5 High (xAI) über OpenCodex (ocx) auf Linux einrichten. Der ocx-Proxy sitzt zwischen Codex und xAI, sodass `codex` gegen grok-4.5 statt gegen OpenAI läuft. Zusätzlich: Superpowers + Skill-Set installieren. Trigger: OpenCodex, ocx, Grok in Codex, Codex mit xai, grok-4.5 harness, superpowers."
risk: safe
metadata:
  type: setup
  verified_on: "2026-07-31 (macOS-Referenzsystem, ocx v2.7.36, LIVE: xai oauth active, model=xai/grok-4.5)"
  tags: [opencodex, ocx, codex, grok, xai, linux, proxy, superpowers, skills]
---

# OpenCodex + Grok 4.5 High (xAI) als Codex-Harness (Linux)

> ⚠️ **Grok mit K = xAI Grok** (`api.x.ai`, Modell `grok-4.5`). NICHT „Groq" mit Q (groq.com) —
> das ist eine andere Firma und hier ausdrücklich **nicht** gemeint.

**Was das ist:** `codex` (das OpenAI-Codex-CLI, „Codex Ultra") wird als Harness benutzt, aber die
Modell-Requests laufen nicht zu OpenAI, sondern über den **OpenCodex-Proxy (`ocx`)** zu
**xAI Grok 4.5**. Codex spricht mit einer OpenAI-kompatiblen API — dazwischen sitzt `ocx` und
routet auf `xai/grok-4.5`.

```
codex  ──►  http://127.0.0.1:10100/v1  ──►  ocx proxy  ──►  https://api.x.ai/v1  (grok-4.5)
        (openai_base_url in ~/.codex/config.toml)      (xAI OAuth)
```

## Verifizierte Eckdaten (Referenzsystem, LIVE 31.07.2026)
- ocx **2.7.36**, Proxy-Port **10100**, Default-Provider **xai**, aktiver Account `xai oauth active`
- **Grok 4.5 „High"** = Modell `grok-4.5` + `model_reasoning_effort = "high"` (low/medium/high)
- grok-4.5 Kontextfenster **500.000** Token
- Codex-Bindung (real gesetzt): `model = "xai/grok-4.5"`,
  `openai_base_url = "http://127.0.0.1:10100/v1"`

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
ocx login xai     # OAuth im Browser; KEIN device-code-Flag
ocx account list  # muss  xai oauth … active  zeigen
```
Der OAuth-Token wird von ocx sicher in `~/.opencodex/` verwaltet — **nie** in ein Repo/Gist.
Headless-Linux: einmal auf einem Desktop einloggen und `~/.opencodex/` mitkopieren.

### 3. Grok als Default festlegen und Proxy starten
```bash
ocx init          # Provider = xai wählen; injiziert die Codex-Config
ocx start         # Proxy :10100, synct die Modelle in die Codex-Config
ocx status        # „Proxy: running", Default provider: xai
# Dauerbetrieb (überlebt Logout):
ocx service
ocx codex-shim install
```

### 4. Auf Grok 4.5 High nageln — „überall Grok"
`~/.codex/config.toml`:
```toml
model = "xai/grok-4.5"
model_reasoning_effort = "high"
openai_base_url = "http://127.0.0.1:10100/v1"
```
Pro Aufruf:
```bash
codex --model xai/grok-4.5 -c model_reasoning_effort=high "…dein Auftrag…"
```

### 5. Beweisen, dass wirklich Grok läuft
```bash
curl -s http://127.0.0.1:10100/v1/models | grep xai/grok-4.5   # Modell da?
ocx status | grep -i provider                                  # Default provider: xai
codex --model xai/grok-4.5 "Nenne Modell und Anbieter."
ocx debug usage                                                # zeigt Requests an api.x.ai
```
**Fertig erst, wenn** `ocx status` `xai` als Default zeigt UND `ocx debug usage` echte
xAI-Requests listet.

---

## Verfügbare xAI-Modelle über ocx
| Modell | Kontext | Reasoning |
|---|---:|---|
| **grok-4.5** (Default) | 500k | low/medium/**high** |
| grok-4.3 | 1M | — |
| grok-4.20-0309-reasoning | 1M | reasoning |
| grok-4.20-0309-non-reasoning | 1M | — |
| grok-build-0.1 | 256k | — |
| grok-composer-2.5-fast | — | — |

---

## Zusätzlich: Superpowers + Skill-Set installieren

Damit die Codex/agy/CC-Lanes dieselben Fähigkeiten haben wie das Referenzsystem.

### Superpowers
- **`using-superpowers`** — erzwingt, dass vor jeder Antwort der passende Skill geladen wird.
- **`skills-load-first`** — Skill-Discovery am Konversationsanfang.
Über den Skills-Marketplace/Plugin installieren.

### Das genutzte Skill-Set mitnehmen
```bash
rsync -a ~/.claude/skills/  <linux-host>:~/.claude/skills/
```
Kern-Skills, die immer mitmüssen: `integrative-deep-research` (IDR + NotebookLM),
`manager-standing-rules`, `report-to-github`, `operator-capture-openspec-qg`,
`notebooklm-mcp-cli`, `knight`, dieser `opencodex-grok-linux`.

---

## Zurück / Troubleshooting
```bash
ocx restore     # Codex wieder direkt auf OpenAI
ocx stop        # Proxy stoppen + Codex nativ
ocx doctor      # Umgebung/Netz diagnostizieren
ocx ensure      # Proxy sicher hochfahren + Codex-Config aktualisieren
```

## Fallstricke
- **`codex -p` ist verboten** — immer interaktives `codex`.
- **Grok mit K, nicht Groq mit Q** — dieser Skill nutzt ausschließlich **xAI Grok** (`api.x.ai`).
- xAI-OAuth-Tokens niemals committen — bleiben in `~/.opencodex/`.
- `ocx start` synct Modelle; nach manuellen Edits `ocx sync` / `ocx sync-cache`.
