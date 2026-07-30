---
name: opencodex-grok-linux
description: "Codex Ultra als Harness mit Grok 4.5 High über OpenCodex (ocx) auf Linux einrichten. Der ocx-Proxy sitzt zwischen Codex und xAI, sodass `codex` gegen grok-4.5 statt gegen OpenAI läuft. Trigger: OpenCodex, ocx, Grok in Codex, Codex mit xai, grok-4.5 harness."
risk: safe
metadata:
  type: setup
  verified_on: "2026-07-31 (macOS-Referenzsystem, ocx v2.7.36)"
  tags: [opencodex, ocx, codex, grok, xai, linux, proxy]
---

# OpenCodex + Grok 4.5 High als Codex-Harness (Linux)

**Was das ist:** `codex` (das OpenAI-Codex-CLI, „Codex Ultra") wird als Harness benutzt, aber die
Modell-Requests laufen nicht zu OpenAI, sondern über den **OpenCodex-Proxy (`ocx`)** zu **xAI Grok
4.5**. Codex denkt, es spricht mit einer OpenAI-kompatiblen API — in Wahrheit sitzt `ocx` dazwischen
und routet auf `xai/grok-4.5`.

```
codex  ──►  http://127.0.0.1:10100/v1  ──►  ocx proxy  ──►  https://api.x.ai/v1  (grok-4.5)
        (openai_base_url in ~/.codex/config.toml)      (xAI OAuth)
```

## Verifizierte Eckdaten (Referenzsystem 31.07.2026)
- ocx-Version **2.7.36**, Proxy-Port **10100**, Health `http://127.0.0.1:10100/healthz`
- Default-Provider **xai**, Default-Modell **grok-4.5**
- **Grok 4.5 „High"** = Modell `grok-4.5` mit `model_reasoning_effort = "high"`
  (unterstützt sind `low`, `medium`, `high`)
- grok-4.5 Kontextfenster **500.000** Token
- Codex-Bindung in `~/.codex/config.toml`: `model = "xai/grok-4.5"` +
  `openai_base_url = "http://127.0.0.1:10100/v1"`

---

## Installation auf Linux

### 0. Voraussetzungen
```bash
# Codex CLI muss da sein
codex --version
# Node/Bun bringt ocx gebündelt mit; Git für den Clone
```

### 1. OpenCodex holen und bauen
```bash
git clone https://github.com/just-every/code ~/code/opencodex   # ocx-Quelle
cd ~/code/opencodex
npm install            # zieht das gebündelte Bun-Runtime
# ocx auf den PATH (Linux: ~/.local/bin oder /usr/local/bin)
ln -sf "$PWD/bin/ocx" ~/.local/bin/ocx
ln -sf "$PWD/bin/ocx" ~/.local/bin/opencodex
export PATH="$HOME/.local/bin:$PATH"     # in ~/.bashrc / ~/.zshrc dauerhaft eintragen
ocx --version
```

### 2. Interaktives Setup + xAI-Login
```bash
ocx init          # fragt Provider ab → xai wählen; injiziert die Codex-Config
ocx login xai     # OAuth im Browser (NUR Browser-Login, KEIN device-code-Flag)
ocx account list  # muss xai … active zeigen
```
⚠️ **Auf headless-Linux ohne Browser:** `ocx login xai` in einer Umgebung ausführen, in der ein
Browser erreichbar ist (SSH-Port-Forward des Login-Callbacks oder einmal auf einem Desktop
einloggen und `~/.opencodex/` mit-kopieren). Der xAI-Login ist **reiner Browser-OAuth**.

### 3. Proxy starten
```bash
ocx start                 # startet Proxy auf :10100 und synct die Modelle in die Codex-Config
ocx status                # „Proxy: running", Default provider: xai
# Dauerhaft als Dienst (überlebt Logout):
ocx service               # install + start als systemd-user-Service
ocx codex-shim install    # startet den Proxy automatisch, sobald `codex` läuft
```

### 4. Auf Grok 4.5 High festnageln
In `~/.codex/config.toml`:
```toml
model = "xai/grok-4.5"
model_reasoning_effort = "high"
openai_base_url = "http://127.0.0.1:10100/v1"
```
Oder pro Aufruf:
```bash
codex --model xai/grok-4.5 -c model_reasoning_effort=high "…dein Auftrag…"
```

### 5. Prüfen, dass es wirklich Grok ist
```bash
curl -s http://127.0.0.1:10100/v1/models | grep -i grok      # muss xai/grok-4.5 listen
codex --model xai/grok-4.5 "Sag mir in einem Satz, welches Modell du bist."
# ocx debug usage  → zeigt die xAI-Requests
```

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

## Zurück auf natives Codex
```bash
ocx restore        # Codex zeigt wieder direkt auf OpenAI (Proxy läuft weiter)
ocx restore back   # wieder auf den Proxy
ocx stop           # Proxy stoppen + Codex auf nativ zurück
ocx uninstall      # alles entfernen
```

## Fallstricke
- **`codex -p` ist verboten** im Manager-Kontext — immer interaktives `codex`.
- ocx **synct die Modelle bei jedem `ocx start`** in die Codex-Config; nach manuellen Edits ggf.
  `ocx sync` / `ocx sync-cache`.
- Bei „ChatGPT unreachable"/WSL-Problemen: `ocx doctor`.
- **Keine Tokens committen.** Die xAI-OAuth-Credentials liegen in `~/.opencodex/` und `~/.codex/`,
  gehören NICHT in einen Gist oder ein Repo.

## Troubleshooting-Einzeiler
```bash
ocx doctor         # Umgebung/Netz/WSL diagnostizieren
ocx ensure         # Proxy sicher hochfahren + Codex-Config aktualisieren
ocx status         # läuft der Proxy? welcher Default-Provider?
```
