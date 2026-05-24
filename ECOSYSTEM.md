# Ecosystem: SDKs, UI kits, templates, projects

Curated with one-line "use this when" notes. The exhaustive list lives in [awesome-telegram-mini-apps](https://github.com/telegram-mini-apps-dev/awesome-telegram-mini-apps).

## Official

- **`telegram-web-app.js`** (`https://telegram.org/js/telegram-web-app.js`) — the bridge script. Always loaded inside Telegram; load it directly only for vanilla projects.
- **[Official guide](https://core.telegram.org/bots/webapps)** — source of truth for the API surface and Bot API version notes.
- **[Community guide](https://docs.telegram-mini-apps.com)** — well-written deep-dive maintained by the `@telegram-apps` team.

## Core JS / TS SDKs

| Package | Use when |
|---|---|
| `@telegram-apps/sdk` | Default for new projects. Modular, tree-shakable, signals-based, framework-agnostic. |
| `@telegram-apps/sdk-react` | React projects using `@telegram-apps/sdk`. Hooks like `useSignal(viewport.height)`. |
| `@telegram-apps/sdk-vue` / `-solid` / `-svelte` | Same idea, other frameworks. |
| `@telegram-apps/init-data-node` | Server-side `initData` validation in Node. Battle-tested. |
| `@telegram-apps/create-mini-app` | Scaffolder. Picks framework, SDK, UI kit, deploy target. |
| `@twa-dev/sdk` | Older but still good thin wrapper over the official script. Pick for vanilla JS or small projects. |
| `@vkruglikov/react-telegram-web-app` | Predecessor of `sdk-react`. Still works; pick `sdk-react` for new code. |
| `DavisDmitry/telegram-webapps` | TypeScript typings only — useful if you already have your own wrapper. |

## UI kits

| Package | Use when |
|---|---|
| `@telegram-apps/telegram-ui` | You want a "looks native" component library matching iOS/Android Telegram styles. |
| `@twa-dev/Mark42` | Lighter alternative, tree-shakable. |
| Telegram Graphics Figma file | You need the official icon set / design tokens. |

## Backend validators

| Library | Language |
|---|---|
| `@telegram-apps/init-data-node` | Node / TS |
| `telegram-webapp-auth` | Python |
| `Telegram-Mini-Apps/init-data-golang` | Go |
| (roll your own — algorithm is ~30 lines) | Rust, Elixir, Ruby, etc. |

## Payments and wallets

- **`@tonconnect/sdk`** / **`@tonconnect/ui`** — TON wallet connect. Use for any crypto flow on TON.
- **Telegram Stars** via `openInvoice` — native in-app payments. No external processor.
- **Telegram Bot Payments** — classic bot payments (Stripe, etc.) via the bot's `sendInvoice`; surface in Mini App via `openInvoice` URL.

## Templates and starters

- `@telegram-apps/create-mini-app` — official scaffolder; covers React/Vue/Solid + Tailwind/UI kit + TON.
- `@twa-dev/vite-boilerplate`, `webpack-boilerplate`, `vanilla-js-boilerplate` — minimal starts.
- `ton-community/twa-template` — TON integration baked in.
- `DKeken/turborepo-ton-trpc` — fullstack monorepo example (Next.js + tRPC + TON).
- `devflex-pro/tma-starter-kit`, `Easterok/telegram-onboarding-kit` — production-shaped starters.

## Notable open-source apps to read

- `neSpecc/telebook` — hotel booking concept.
- `Quatern1on/ChessNowBot` — real-time multiplayer.
- `UselessStudio/TeleOTP` — OTP generator using `CloudStorage`.
- `deptyped/notepher-bot` — notes synced via `CloudStorage`.
- `kubk/memo-card` — spaced repetition flashcards.
- `mauriciobraz/next.js-telegram-webapp` — Next.js integration example.

## Debugging

- `liriliri/eruda` — drop-in mobile console. Gate behind `?eruda=1`.
- `websashka/eruda-tma-cloudstorage` — Eruda plugin to inspect `CloudStorage`.
- Telegram Desktop dev menu — enable in Settings → Advanced → Experimental.

## Communities

- `@twa_dev` Telegram chat — active help.
- [Telegram-Mini-Apps GitHub org](https://github.com/Telegram-Mini-Apps) — community SDKs, validators, docs.
- [`@telegram-apps`](https://github.com/Telegram-Web-Apps) — the org behind `@telegram-apps/sdk`.
