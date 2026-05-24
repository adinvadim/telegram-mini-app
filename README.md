# telegram-mini-app skill

A Claude Code / agent skill for building **Telegram Mini Apps** (TMA). Covers the bridge API, how it differs from regular web, the SDK ecosystem, end-to-end authentication, and a large recommendations block of production-tested lessons.

## Install

Via [skill.sh](https://www.skills.sh):

```bash
npx skills add adinvadim/telegram-mini-app
```

Install globally (available in every project):

```bash
npx skills add adinvadim/telegram-mini-app -g
```

Install only for a specific agent (e.g. Claude Code):

```bash
npx skills add adinvadim/telegram-mini-app -a claude-code -g
```

After install, the agent loads `SKILL.md` automatically when the user asks anything related to Telegram Mini Apps, `window.Telegram.WebApp`, `initData`, `@telegram-apps/sdk`, MainButton, CloudStorage, etc.

## Contents

- [`SKILL.md`](SKILL.md) — main skill: quick start, what's different from regular web, auth, SDKs, recommendations.
- [`REFERENCE.md`](REFERENCE.md) — full `window.Telegram.WebApp` API surface, events, theme vars, version timeline.
- [`AUTH.md`](AUTH.md) — deep dive on `initData` validation (Node, Python, Go), Ed25519 third-party flow, sessions, edge cases.
- [`ECOSYSTEM.md`](ECOSYSTEM.md) — curated SDKs, UI kits, templates, validators, debug tools.

## Sources

- [Official Telegram Web Apps docs](https://core.telegram.org/bots/webapps)
- [awesome-telegram-mini-apps](https://github.com/telegram-mini-apps-dev/awesome-telegram-mini-apps)
- [Community guide](https://docs.telegram-mini-apps.com)

## License

MIT
