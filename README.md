# OpenCodex + Groq als Codex-Harness (Linux) + Superpowers/Skills

Setup-Skill: **Codex Ultra** als Harness, dazwischen der **OpenCodex-Proxy (`ocx`)**,
dahinter **Groq** (groq.com, LPU-Inference, Weekly-Abo). `codex` läuft dadurch gegen
**Groq-Modelle** statt gegen OpenAI. Plus: Superpowers + das genutzte Skill-Set installieren.

⚠️ **Groq ≠ Grok** — gemeint ist groq.com (`api.groq.com`), nicht xAI Grok.

👉 Vollständige Anleitung in **[SKILL.md](SKILL.md)**.

Kurz: `ocx login groq` (API-Key aus console.groq.com/keys) → `ocx init` (Provider groq) →
`ocx start` → `ocx sync`, dann in `~/.codex/config.toml`
`model = "groq/<modell>"`, `openai_base_url = "http://127.0.0.1:10100/v1"`.
Fertig, sobald `ocx status` groq als Default zeigt und `ocx debug usage` echte Groq-Requests listet.

Verifiziert am Referenzsystem 31.07.2026 (ocx v2.7.36, Groq-Provider in registry.ts).
Keine Zugangsdaten enthalten — der Groq-Key bleibt in `~/.opencodex/`.
