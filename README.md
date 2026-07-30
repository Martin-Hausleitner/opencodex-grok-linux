# 🦾 OpenCodex + Grok 4.5 High — Codex-Harness auf Linux

> **Codex Ultra** als Harness · **OpenCodex (`ocx`)** als Proxy · **xAI Grok 4.5 High** als Modell.
> `codex` läuft dadurch gegen `grok-4.5` statt gegen OpenAI.

⚠️ **Grok mit K** (xAI, `api.x.ai`) — **nicht** „Groq" mit Q (groq.com). Andere Firma, hier nicht gemeint.

---

## ⚡ Was passiert hier

```
codex  ──►  ocx proxy (:10100)  ──►  xAI Grok 4.5
```

Der ocx-Proxy tarnt sich als OpenAI-API. Codex merkt nichts — es bekommt Grok.

## 🚀 In 4 Schritten

| # | Befehl | Ergebnis |
|--:|--------|----------|
| 1 | `git clone https://github.com/just-every/code ~/code/opencodex && cd ~/code/opencodex && npm install` | ocx gebaut |
| 2 | `ocx login xai` | Grok-Tokens per Browser-OAuth |
| 3 | `ocx init` → Provider **xai**, dann `ocx start` | Proxy läuft, Grok als Default |
| 4 | `~/.codex/config.toml`: `model = "xai/grok-4.5"`, `model_reasoning_effort = "high"`, `openai_base_url = "http://127.0.0.1:10100/v1"` | überall Grok 4.5 High |

## ✅ Beweis

```bash
ocx status                # Default provider: xai
curl -s http://127.0.0.1:10100/v1/models | grep xai/grok-4.5
ocx debug usage           # echte Requests an api.x.ai
```

## 📚 Modelle

| Modell | Kontext | Reasoning |
|--------|--------:|-----------|
| **grok-4.5** ⭐ | 500k | low · medium · **high** |
| grok-4.3 | 1M | — |
| grok-4.20 reasoning | 1M | ✓ |

## ➕ Superpowers + Skill-Set

```bash
rsync -a ~/.claude/skills/ <linux-host>:~/.claude/skills/
```
Kern: `using-superpowers`, `skills-load-first`, `integrative-deep-research`,
`manager-standing-rules`, `report-to-github`, `knight`.

---

👉 **Volle Anleitung: [SKILL.md](SKILL.md)** · Verifiziert 31.07.2026 · ocx v2.7.36 · keine Tokens im Repo.
