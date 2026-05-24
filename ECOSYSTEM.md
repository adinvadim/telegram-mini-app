# Ecosystem: SDKs, UI kits, templates, projects

Curated with one-line "use this when" notes. The exhaustive list lives in [awesome-telegram-mini-apps](https://github.com/telegram-mini-apps-dev/awesome-telegram-mini-apps).

## Official

- **`telegram-web-app.js`** (`https://telegram.org/js/telegram-web-app.js`) — the bridge script. Always loaded inside Telegram; load it directly only for vanilla projects.
- **[Official guide](https://core.telegram.org/bots/webapps)** — source of truth for the API surface and Bot API version notes.
- **[Community guide](https://docs.telegram-mini-apps.com)** — well-written deep-dive maintained by the `@telegram-apps` team.

## Core JS / TS SDKs

The `@telegram-apps/*` monorepo (`Telegram-Mini-Apps/telegram-apps` on GitHub, originally `@tma.js`) ships these packages:

| Package | Use when |
|---|---|
| `@telegram-apps/sdk` | Default core. Modular, tree-shakable, signals-based, framework-agnostic. `init()` + per-feature `mount()`. |
| `@telegram-apps/sdk-react` | React bindings. `useSignal(viewport.height)`-style hooks. |
| `@telegram-apps/sdk-vue` | Vue 3 composables. |
| `@telegram-apps/sdk-solid` | Solid.js bindings. |
| `@telegram-apps/sdk-svelte` | Svelte stores. |
| `@telegram-apps/bridge` | Low-level postEvent/onEvent wrapper if you do not want the high-level `sdk` API. |
| `@telegram-apps/signals` | Reactive primitives used internally by the SDK; usable standalone. |
| `@telegram-apps/init-data-node` | Server-side `initData` validation in Node. Battle-tested. |
| `@telegram-apps/create-mini-app` | Scaffolder. Picks framework + UI kit + TON. |
| `@telegram-apps/transformers` | Helpers to parse/serialize launch params and themes. |
| `@telegram-apps/toolkit` / `types` | Shared utilities and TS types. |
| `@telegram-apps/test-utils` | Mock the bridge in unit tests. |

Older / alternative SDKs still in wide use:

| Package | Use when |
|---|---|
| `@twa-dev/sdk` | Thin TS wrapper over `telegram-web-app.js`. Loads the script automatically. Also ships React components (`MainButton`, `SecondaryButton`, `BottomBar`, `BackButton`). Good for vanilla / small projects. |
| `@vkruglikov/react-telegram-web-app` | Predecessor of `sdk-react`. Provider `WebAppProvider` + hooks (see below). Pick `sdk-react` for new code. |
| `DavisDmitry/telegram-webapps` | TypeScript typings only — useful if you already have your own wrapper. |

### `@vkruglikov/react-telegram-web-app` hook/component surface

Provider `<WebAppProvider>` must wrap the tree. Then:

- **Components:** `<MainButton>`, `<BackButton>`, `<SettingsButton>`
- **Hooks:** `useWebApp`, `useInitData`, `useExpand`, `useThemeParams`, `useHapticFeedback`, `useShowPopup`, `useScanQrPopup`, `useReadTextFromClipboard`, `useSwitchInlineQuery`, `useCloudStorage`, `useBackButton`

`useCloudStorage` returns Promise-based versions of the callback API — that alone is a reason to use it over raw `tg.CloudStorage`.

## UI kits

| Package | Use when |
|---|---|
| `@telegram-apps/telegram-ui` | You want a "looks native" component library matching iOS/Android Telegram styles. Wrap the tree in `<AppRoot>`; import `'@telegram-apps/telegram-ui/dist/styles.css'`. Playground/Storybook: [tgui.xelene.me](https://tgui.xelene.me/). |
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
