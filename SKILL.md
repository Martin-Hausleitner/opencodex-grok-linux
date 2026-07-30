---
name: opencodex-grok-linux
description: "Codex Ultra als Harness mit GROQ (groq.com, LPU-Inference, Weekly-Abo) über OpenCodex (ocx) auf Linux einrichten. Der ocx-Proxy sitzt zwischen Codex und Groq, sodass `codex` gegen Groq-Modelle statt gegen OpenAI läuft. Zusätzlich: Superpowers + Skill-Set installieren. Trigger: OpenCodex, ocx, Groq in Codex, Codex mit groq, groq harness, superpowers."
risk: safe
metadata:
  type: setup
  verified_on: "2026-07-31 (macOS-Referenzsystem, ocx v2.7.36; Groq-Provider in ocx registry.ts bestätigt)"
  tags: [opencodex, ocx, codex, groq, linux, proxy, superpowers, skills]
---

# OpenCodex + Groq als Codex-Harness (Linux) + Superpowers/Skills

**Was das ist:** `codex` (das OpenAI-Codex-CLI, „Codex Ultra") wird als Harness benutzt, aber die
Modell-Requests laufen nicht zu OpenAI, sondern über den **OpenCodex-Proxy (`ocx`)** zu
**Groq** (groq.com, LPU-Inference-Cloud, Weekly-Abo). Codex spricht mit einer OpenAI-kompatiblen
API — dazwischen sitzt `ocx` und routet auf Groq.

> ⚠️ **Groq ≠ Grok.** Gemeint ist **Groq** (groq.com, `api.groq.com`), die schnelle
> Inference-Cloud, NICHT xAI Grok. Der ocx-Provider heißt exakt `groq`.

```
codex  ──►  http://127.0.0.1:10100/v1  ──►  ocx proxy  ──►  https://api.groq.com/openai/v1
        (openai_base_url in ~/.codex/config.toml)      (Groq, API-Key)
```

## Verifizierte Eckdaten (Referenzsystem 31.07.2026)
- ocx-Version **2.7.36**, Proxy-Port **10100**
- ocx-Provider **`groq`**: `adapter: openai-chat`, `baseUrl: https://api.groq.com/openai/v1`,
  **`authKind: "key"`**, Dashboard `https://console.groq.com/keys` (bestätigt in
  `opencodex/src/providers/registry.ts`)
- **Groq nutzt einen API-Key**, KEIN OAuth. Key aus dem Groq-Konto (Weekly-Abo) unter
  https://console.groq.com/keys

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

### 2. Groq als Provider einrichten (API-Key)
```bash
ocx login groq        # fragt den API-Key ab → aus https://console.groq.com/keys einfügen
# ocx speichert den Key sicher in ~/.opencodex/ (NICHT in ein Repo/Gist)
ocx account list      # muss  groq … active  zeigen
```

### 3. Groq als Default festlegen und Proxy starten
```bash
ocx init              # Provider = groq wählen; injiziert die Codex-Config
ocx start             # Proxy auf :10100, synct die Groq-Modelle in die Codex-Config
ocx sync              # Live-Modellliste von Groq holen
ocx status            # „Proxy: running", Default provider: groq
# Dauerbetrieb:
ocx service           # systemd-user-Service (überlebt Logout)
ocx codex-shim install  # Proxy startet automatisch, sobald `codex` läuft
```

### 4. Codex auf ein Groq-Modell nageln — „überall Groq"
Die **echte** Modell-ID kommt aus `ocx sync` / dem Groq-Katalog (nicht raten). Danach in
`~/.codex/config.toml`:
```toml
model = "groq/<modell-id-aus-ocx-sync>"
openai_base_url = "http://127.0.0.1:10100/v1"
```
Modelle listen:
```bash
curl -s http://127.0.0.1:10100/v1/models | grep -i groq
```
Pro Aufruf:
```bash
codex --model "groq/<modell-id>" "…dein Auftrag…"
```

### 5. Beweisen, dass wirklich Groq läuft
```bash
curl -s http://127.0.0.1:10100/v1/models | grep -i groq     # Groq-Modelle da?
ocx status | grep -i "provider"                              # Default provider: groq
codex --model "groq/<modell-id>" "Nenne dein Modell und deinen Anbieter."
ocx debug usage                                              # zeigt Requests an api.groq.com
```
**Fertig erst, wenn** `ocx status` `groq` als Default zeigt UND `ocx debug usage` echte
Groq-Requests listet.

---

## Zusätzlich: Superpowers + Skill-Set installieren

Damit die Codex/agy/CC-Lanes dieselben Fähigkeiten haben wie das Referenzsystem:

### Superpowers
```bash
# Superpowers-Skillframework (obra) über den Skills-Marketplace/Plugin installieren.
# Kern-Skills: using-superpowers (lädt Skills VOR jeder Antwort), skills-load-first.
```
- `using-superpowers` — erzwingt, dass vor jeder Antwort der passende Skill geladen wird.
- `skills-load-first` — Skill-Discovery am Konversationsanfang.

### Das genutzte Skill-Set mitnehmen
Die im Alltag genutzten Skills liegen unter `~/.claude/skills/`. Für ein neues Linux-System:
```bash
# vom Referenzsystem den Skill-Ordner übertragen
rsync -a ~/.claude/skills/  <linux-host>:~/.claude/skills/
```
Kern-Skills, die immer mitmüssen: `integrative-deep-research` (IDR + NotebookLM),
`manager-rules`, `report-github-comet`, `openspec-capture-qg`, `notebooklm-cli`,
`idr-knight-cli`, dieser `opencodex-grok-linux`.

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
- **Groq-Key niemals committen** — bleibt in `~/.opencodex/`, nicht in Repo/Gist.
- `ocx start` synct Modelle; nach manuellen Edits `ocx sync` / `ocx sync-cache`.
- Modell-IDs NICHT raten — immer aus `ocx sync` / `/v1/models` nehmen.
