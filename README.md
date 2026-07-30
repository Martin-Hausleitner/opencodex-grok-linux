# OpenCodex + Grok 4.5 High als Codex-Harness (Linux)

Setup-Skill: **Codex Ultra** als Harness, dazwischen der **OpenCodex-Proxy (`ocx`)**,
dahinter **xAI Grok 4.5 (High)**. `codex` läuft dadurch gegen `grok-4.5` statt gegen OpenAI.

👉 Die vollständige Anleitung steht in **[SKILL.md](SKILL.md)**.

Kurz: `ocx init` → `ocx login xai` → `ocx start`, dann in `~/.codex/config.toml`
`model = "xai/grok-4.5"`, `model_reasoning_effort = "high"`,
`openai_base_url = "http://127.0.0.1:10100/v1"`.

Verifiziert am Referenzsystem 31.07.2026 (ocx v2.7.36, Proxy :10100).
Keine Zugangsdaten enthalten — die xAI-OAuth-Credentials bleiben in `~/.opencodex/`.
