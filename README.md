# 🦾 Codex-CLI als Grok-Harness über OpenCodex (Linux)

Der **`codex`-Coding-Agent** ist der Harness. **OpenCodex (`ocx`)** hängt sich als lokaler Proxy
dazwischen und lässt den ganzen Codex-Agent auf **xAI grok-4.5 High** laufen statt auf OpenAI.

> Du nutzt die **volle grok-CLI/Codex-Erfahrung** — planen, Dateien lesen, Befehle ausführen —
> nur mit Grok als Gehirn. **Kein** direkter API-Call im Code; der Proxy macht das transparent.

```
codex (Harness)  ──►  ocx proxy (:10100)  ──►  grok-4.5
```

## 🚀 In 4 Schritten

| # | Befehl | Ergebnis |
|--:|--------|----------|
| 1 | `git clone https://github.com/just-every/code ~/code/opencodex && cd ~/code/opencodex && npm install` | ocx gebaut |
| 2 | `ocx login xai` | Grok-Tokens per Browser-OAuth |
| 3 | `ocx init` (Provider xai) → `ocx start` → `ocx codex-shim install` | jeder `codex` läuft auf Grok |
| 4 | `~/.codex/config.toml`: `model = "xai/grok-4.5"`, `model_reasoning_effort = "high"` | Grok 4.5 High überall |

## ✅ Beweis
```bash
ocx status | grep -i provider          # Default provider: xai
codex "Welches Modell steuert dich?"   # normaler Codex-Lauf, jetzt auf Grok
ocx debug usage                        # Requests an api.x.ai
```

## ➕ Superpowers + Skill-Set
```bash
rsync -a ~/.claude/skills/ <linux-host>:~/.claude/skills/
```
Kern: `using-superpowers`, `skills-load-first`, `integrative-deep-research`, `knight`.

---

👉 **Volle Anleitung: [SKILL.md](SKILL.md)** · verifiziert 31.07.2026 · ocx v2.7.36 · keine Tokens im Repo.
