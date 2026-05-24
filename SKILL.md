---
name: telegram-mini-app
description: Build, debug, and ship Telegram Mini Apps (TMA / Telegram Web Apps). Use when the user works with `window.Telegram.WebApp`, `@telegram-apps/sdk`, `@twa-dev/sdk`, `telegram-web-app.js`, initData validation, MainButton/BackButton, CloudStorage, BiometricManager, Telegram theming, BotFather mini-app setup, or asks how Mini Apps differ from regular web apps, how to authenticate a Telegram user on the backend, how to deep-link via `startapp`, or how to integrate TON Connect / Telegram Stars.
---

# Telegram Mini App

Telegram Mini Apps (TMA, formerly "Web Apps") are HTML/JS apps that run **inside the Telegram client** as a WebView, with a JS bridge to the host (`window.Telegram.WebApp`). They look like native UI but are web tech — with extra rules and capabilities you do not get in a normal browser.

This skill assumes the developer knows web fundamentals. It teaches: what is different from the regular web, which SDKs to use, how authentication actually works end-to-end, and a large recommendations block of things that break in production.

## Quick start

Two paths. Pick **modern SDK** for any new project; pick **raw script** only for prototypes or vanilla HTML.

### Modern: `@telegram-apps/sdk` (recommended)

```bash
npm i @telegram-apps/sdk
```

```ts
import { init, miniApp, themeParams, viewport, mainButton, backButton, initData } from '@telegram-apps/sdk';

init();                       // mounts bridge, restores launch params
miniApp.mount();              // mount the mini-app component
themeParams.mount();          // sync theme CSS vars
viewport.mount();             // track viewport / safe areas
initData.restore();           // expose parsed initData

miniApp.ready();              // tell Telegram we are painted
viewport.expand();            // fill the screen

mainButton.setParams({ text: 'Pay', isVisible: true, isEnabled: true });
mainButton.onClick(() => { /* ... */ });
```

### Raw: official script

```html
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<script>
  const tg = window.Telegram.WebApp;
  tg.ready();
  tg.expand();
  tg.MainButton.setText('Pay').show().onClick(() => tg.sendData('paid'));
</script>
```

Always include the official script (or use an SDK that loads it) — without it, `window.Telegram.WebApp` is undefined.

## How Mini Apps differ from regular web

| Concern | Regular web | Telegram Mini App |
|---|---|---|
| Auth | Cookies / OAuth | Signed `initData` query string, validated server-side with bot token |
| Identity | Email / OAuth profile | Telegram `user.id` (numeric), no email guaranteed |
| Storage | `localStorage`, IndexedDB | Works, **but** is per-WebView and may be wiped — use `CloudStorage` for cross-device, `SecureStorage` for tokens |
| Theming | Your design system | Must respect Telegram theme via CSS vars (`--tg-theme-*`) and react to `themeChanged` |
| Navigation | Browser back, History API | Telegram **BackButton** in the header; the browser back gesture often closes the app |
| Viewport | `window.innerHeight` | `viewportHeight` / `viewportStableHeight` (changes on keyboard, fullscreen, drag); plus safe-area insets |
| External links | `<a href>` works | Must call `openLink()` / `openTelegramLink()` — `<a target=_blank>` is unreliable |
| Payments | Stripe etc. | `openInvoice()` + Telegram Stars / Bot Payments |
| Sharing | Web Share API | `shareToStory`, `switchInlineQuery`, `shareMessage` |
| Closing | User closes tab | `tg.close()` or system swipe; persist state proactively |
| Console / DevTools | Open in browser | Use Telegram **Desktop** with dev menu, or [Eruda](https://github.com/liriliri/eruda) overlay on mobile |
| Versioning | Evergreen browsers | Many client versions in the wild — gate features with `isVersionAtLeast()` |

The single most common mistake: treating the app like a normal SPA and ignoring `initData`, theme, viewport, and BackButton. Each one will produce a bug in production.

## Authentication (read this carefully)

There are two objects exposed by the bridge:

- **`initDataUnsafe`** — already parsed, **never trust on the server**. Fine for showing the user's first name in the UI.
- **`initData`** — the raw query string. **This is what you send to your backend.** It is signed by Telegram.

### Server-side validation (HMAC, with bot token)

1. Parse `initData` as `URLSearchParams`. Extract `hash`. Keep `auth_date`.
2. Build `data_check_string`: take all remaining keys, sort alphabetically, join as `key=value` lines separated by `\n`.
3. `secret_key = HMAC_SHA256(key="WebAppData", message=BOT_TOKEN)`.
4. `computed = HMAC_SHA256(key=secret_key, message=data_check_string).hex()`.
5. Constant-time compare `computed === hash`. **Also** reject if `auth_date` is older than ~24h (replay protection).

Node example:

```ts
import crypto from 'node:crypto';

export function verifyInitData(initData: string, botToken: string, maxAgeSec = 86400) {
  const url = new URLSearchParams(initData);
  const hash = url.get('hash');
  if (!hash) return null;
  url.delete('hash');

  const dataCheckString = [...url.entries()]
    .map(([k, v]) => [k, v] as const)
    .sort(([a], [b]) => a.localeCompare(b))
    .map(([k, v]) => `${k}=${v}`)
    .join('\n');

  const secret = crypto.createHmac('sha256', 'WebAppData').update(botToken).digest();
  const computed = crypto.createHmac('sha256', secret).update(dataCheckString).digest('hex');

  if (!crypto.timingSafeEqual(Buffer.from(computed, 'hex'), Buffer.from(hash, 'hex'))) return null;

  const authDate = Number(url.get('auth_date'));
  if (!authDate || Date.now() / 1000 - authDate > maxAgeSec) return null;

  return { user: JSON.parse(url.get('user') ?? 'null'), authDate };
}
```

Use a vetted library where possible:

- Node/TS: `@telegram-apps/init-data-node`
- Python: `telegram-webapp-auth`
- Go: `init-data-golang` (`github.com/Telegram-Mini-Apps/init-data-golang`)
- Rust: `tma-auth`, or implement directly — the algorithm is small.

### Third-party validation (no bot token)

Since Bot API 7.x there is **Ed25519** validation: Telegram signs `bot_id:WebAppData\n<data_check_string>` with a known public key per environment (test/prod). Use this when validating from a service that does not own the bot token (e.g. a partner backend, an analytics pipeline).

### Session model after validation

`initData` proves identity *for this launch*. Do not re-validate on every request. After verifying once, mint your **own** short-lived JWT / session cookie bound to the Telegram user id, and use that going forward. Refresh by re-reading `initData` (it stays in `window.Telegram.WebApp.initData` for the launch lifetime).

See [AUTH.md](AUTH.md) for: validation in non-Node backends, Ed25519 third-party flow, login widget vs Mini App auth, attaching a TON wallet, and edge cases (no `user` on attachment-menu launches in some clients).

## Additional SDKs and what they are for

### JS / TS core

- **`@telegram-apps/sdk`** — modern, modular, type-safe, framework-agnostic. The default choice in 2025+. Evolution of the old `@twa.js`. Sibling packages: `@telegram-apps/bridge` (low-level), `@telegram-apps/signals` (reactivity), `@telegram-apps/transformers`, `@telegram-apps/test-utils`.
- **`@telegram-apps/sdk-react`** / **`@telegram-apps/sdk-vue`** / **`@telegram-apps/sdk-solid`** / **`@telegram-apps/sdk-svelte`** — framework bindings with hooks/composables/stores.
- **`@twa-dev/sdk`** — thin TS wrapper over the official script (auto-loads it). Also ships React components (`MainButton`, `SecondaryButton`, `BottomBar`, `BackButton`). Older but stable.
- **`@vkruglikov/react-telegram-web-app`** — `<WebAppProvider>` plus hooks (`useShowPopup`, `useCloudStorage`, `useInitData`, `useExpand`, `useThemeParams`, `useHapticFeedback`, `useScanQrPopup`, `useReadTextFromClipboard`, `useSwitchInlineQuery`, `useBackButton`) and components (`<MainButton>`, `<BackButton>`, `<SettingsButton>`). Useful migration target before `sdk-react`.

### UI kits

- **`@telegram-apps/telegram-ui`** — official-feeling component library matching Telegram's iOS/Android look, theme-aware.
- **`@twa-dev/Mark42`** — lighter, tree-shakable alternative.
- **Telegram Graphics Figma file** — design assets and icons.

### Backend init-data validators

- `@telegram-apps/init-data-node` (Node)
- `init-data-golang` (Go)
- `telegram-webapp-auth` (Python)

### TON / payments

- **`@tonconnect/ui`** + **`@tonconnect/sdk`** — wallet connect for TON.
- **Telegram Stars** + **`openInvoice`** — native in-app payments. No external processor needed.

### Templates worth knowing

- `@telegram-apps/create-mini-app` — scaffolder, picks framework + SDK preset.
- `@twa-dev/vite-boilerplate`, `@twa-dev/webpack-boilerplate`, `@twa-dev/vanilla-js-boilerplate`.
- `ton-community/twa-template` — TON-integrated starter.

A fuller catalogue is in the [awesome-telegram-mini-apps](https://github.com/telegram-mini-apps-dev/awesome-telegram-mini-apps) list. See also [REFERENCE.md](REFERENCE.md) for the full bridge API surface, event list, and Bot API version table.

## Recommendations (the long block)

These are field lessons. Read them before shipping.

### Setup and BotFather

- **Use HTTPS with a real cert.** Self-signed will not work on iOS and silently fails on some Android builds.
- **Configure two URLs in BotFather:** the menu-button URL **and** the direct-link mini app URL (`/newapp`). They are separate features and people confuse them.
- **Set theme colors and short name** in BotFather — they control the loading screen and the OS share preview.
- **Test on iOS, Android, Desktop, and macOS Telegram.** The four WebViews behave differently (WKWebView, GeckoView/Chromium, CEF, WKWebView). Especially: keyboard handling, safe areas, and `vh` units.

### Auth and security

- Validate `initData` on **every** privileged request — or once at session start, then issue your own token. Never trust `initDataUnsafe`.
- Treat `BOT_TOKEN` as a top-tier secret — anyone with it can impersonate any user of your bot.
- Enforce `auth_date` freshness (24h is the conventional cap; tighter is fine).
- Be aware that some launches (attachment menu in non-private chats) **do not include `user`** — only `chat`. Handle gracefully.
- When you need to authenticate from a service that does **not** own the bot token, use the Ed25519 third-party validation flow, not "trust the frontend".

### UI and UX

- **Respect the theme.** Use `var(--tg-theme-bg-color)` / `--tg-theme-text-color` / etc. Listen to `themeChanged` and re-apply where CSS vars are not enough (e.g., SVG fills).
- **Use MainButton for the primary action**, not a button you drew yourself. Users expect it; it is keyboard-aware and stays above the safe area.
- **Use BackButton** (`tg.BackButton.show()` / `onClick`). If you ignore it, users will swipe-close your app and lose state.
- **`tg.ready()` matters.** Call it once your first paint is done — Telegram hides the loading screen on this event. Calling it too early shows a blank app; too late shows the spinner for seconds.
- **`tg.expand()` early.** The default half-height sheet is rarely what you want.
- **Safe areas:** read `safeAreaInset` and `contentSafeAreaInset` (Bot API 8+). Pad the top so your header is not under the notch and the bottom so it is not under the gesture bar.
- **Avoid `100vh`** — it lies. Use `tg.viewportStableHeight` (or the SDK's reactive viewport) and re-layout on `viewportChanged`.
- **HapticFeedback** on important interactions (`impactOccurred('light')` for taps, `notificationOccurred('success'|'error')` for outcomes). Subtle but very "native-feeling".
- **Keyboard:** on iOS the keyboard pushes the WebView, on Android it resizes. Test forms on both.

### Storage

- Use **`CloudStorage`** for anything cross-device (1024 keys, 4096 chars/value). It is async and rate-limited — batch writes.
- Use **`SecureStorage`** (Bot API 9+) for tokens — backed by iOS Keychain / Android Keystore.
- `localStorage` is OK for caches but the WebView storage can be cleared without warning.
- Never store secrets in `localStorage` or in the URL.

### Networking and performance

- Cold start is on a low-end Android in a WebView in a chat — **budget aggressively.** Aim < 1.5s to first meaningful paint.
- **Code-split** by route; lazy-load TonConnect and any heavy SDKs.
- **Inline critical CSS** to avoid a flash of unthemed content (the WebView starts before theme params arrive — read `themeParams` synchronously on first paint).
- Watch bundle size: the WebView ships every byte over a possibly bad mobile network.
- **CORS:** your backend must allow your Mini App origin. The WebView's Origin is your hosted URL, not Telegram's.

### Versioning and feature detection

- Always wrap newer APIs with `tg.isVersionAtLeast('7.0')` (or the SDK's `isSupported`). Failing to do this throws `WebAppMethodUnsupported` and crashes flows on older clients.
- Modern features by minimum version (rough map): BackButton/Haptics 6.1, CloudStorage 6.9, SettingsButton 7.0, SecondaryButton 7.10, fullscreen + sensors 8.0, DeviceStorage/SecureStorage 9.0.

### Links and navigation

- Internal Telegram links (`t.me/...`) → `tg.openTelegramLink(url)`. External → `tg.openLink(url, { try_instant_view: true })`.
- Plain `<a href>` works inconsistently — wrap in handlers.
- Avoid client-side hash routing; some platforms drop the hash on reopen. Prefer path routing with a fallback parser for `tgWebAppStartParam`.

### Deep linking

- Direct link format: `https://t.me/<bot>/<app>?startapp=<payload>`. The payload appears as `start_param` (in `initDataUnsafe`) and as `tgWebAppStartParam` (URL).
- Validate / sanitize `start_param` — it is user-controllable input.

### Payments

- `openInvoice(url, cb)` callback statuses: `paid`, `cancelled`, `failed`, `pending`. **Treat `paid` as "the user clicked paid"**, not as money received — confirm via your backend webhook from Telegram.
- For Stars-only flows, you do not need a Stripe-like merchant — Telegram handles it.

### Closing and persistence

- The app can be closed at any moment (swipe, MainButton handler, `tg.close()`). Persist progress on every meaningful interaction, not at unload.
- `unload` / `beforeunload` are **not** reliable in the WebView.

### Debugging

- **Telegram Desktop** has a dev menu (`Settings → Advanced → Experimental → Enable webview inspector` on Desktop; on macOS, Safari Web Inspector attaches).
- For mobile: drop in **[Eruda](https://github.com/liriliri/eruda)** behind a query flag (`?eruda=1`) — gives you a console overlay.
- iOS Telegram supports Safari Web Inspector when the Mini App is open (since 2024).
- Log to your own backend in production — local logs are unreachable.

### Testing

- Bot token for tests should be a **separate test bot** — never your prod token.
- For init-data testing, generate fixtures with a known bot token; assert both happy and tampered-hash paths.
- Visual regression: capture on real iOS + Android Telegram, not just a browser — fonts and safe areas differ.

### Anti-patterns to avoid

- ❌ Trusting `initDataUnsafe.user` on the backend.
- ❌ Calling `tg.ready()` before rendering anything.
- ❌ Hard-coded colors instead of theme variables.
- ❌ Showing a custom "Back" button in the body instead of using `BackButton`.
- ❌ Building a multi-screen flow without listening to `viewportChanged`.
- ❌ Forgetting `isVersionAtLeast` and crashing for users on older clients.
- ❌ Storing JWTs in `localStorage` when `SecureStorage` is available.
- ❌ Assuming `user` is always present (it is not on some attachment-menu launches).

## When to consult the extras

- Full bridge surface, event names, parameter shapes → [REFERENCE.md](REFERENCE.md)
- Deep authentication walkthroughs (Ed25519 third-party, sessions, multi-tenant bots, edge cases) → [AUTH.md](AUTH.md)
- Curated SDK / template / project links with one-line "use this when" notes → [ECOSYSTEM.md](ECOSYSTEM.md)
