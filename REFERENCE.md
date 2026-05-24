# Reference: `window.Telegram.WebApp` API surface

This is a working reference, not a replacement for the [official docs](https://core.telegram.org/bots/webapps). Use it for quick lookups during development.

## Top-level fields

| Field | Notes |
|---|---|
| `initData` | Raw signed query string. Send to backend for validation. |
| `initDataUnsafe` | Parsed object: `query_id`, `user`, `receiver`, `chat`, `chat_type`, `chat_instance`, `start_param`, `can_send_after`, `auth_date`, `hash`. **Do not trust on server.** |
| `version` | Bot API version of the host client. |
| `platform` | One of `android`, `android_x`, `ios`, `macos`, `tdesktop`, `weba`, `webk`, `unigram`, `unknown`. |
| `colorScheme` | `'light' \| 'dark'`. |
| `themeParams` | See theme section below. |
| `isExpanded` | Whether the app fills the screen. |
| `viewportHeight`, `viewportStableHeight` | Heights in px. Stable is the height after gestures settle. |
| `headerColor`, `backgroundColor`, `bottomBarColor` | Host chrome colors. |
| `isClosingConfirmationEnabled`, `isVerticalSwipesEnabled` | Behavior toggles. |
| `safeAreaInset`, `contentSafeAreaInset` | `{ top, right, bottom, left }` (Bot API 8+). |

## Methods

### Lifecycle
- `ready()` — call after first paint.
- `expand()` — go full-height.
- `close()` — close the app.
- `enableClosingConfirmation()` / `disableClosingConfirmation()`.
- `enableVerticalSwipes()` / `disableVerticalSwipes()` (8.0+).
- `requestFullscreen()` / `exitFullscreen()` (8.0+).

### Buttons
- `MainButton` / `SecondaryButton`: `setText`, `setParams({text, color, text_color, is_active, is_visible, has_shine_effect, position})`, `show`, `hide`, `enable`, `disable`, `showProgress(leaveActive?)`, `hideProgress`, `onClick(fn)`, `offClick(fn)`.
- `BackButton`: `show`, `hide`, `onClick`, `offClick`.
- `SettingsButton`: `show`, `hide`, `onClick`, `offClick` (7.0+).

### Feedback
- `HapticFeedback.impactOccurred('light'|'medium'|'heavy'|'rigid'|'soft')`
- `HapticFeedback.notificationOccurred('error'|'success'|'warning')`
- `HapticFeedback.selectionChanged()`

### Popups
- `showAlert(message, cb?)`
- `showConfirm(message, cb?)` — `cb(ok: boolean)`
- `showPopup({ title?, message, buttons? }, cb?)` — buttons: `{ id?, type?: 'default'|'ok'|'close'|'cancel'|'destructive', text? }`.
- `showScanQrPopup({ text? }, cb?)` / `closeScanQrPopup()`
- `readTextFromClipboard(cb?)`

### Storage
- `CloudStorage.setItem(k, v, cb?)`, `getItem(k, cb)`, `getItems(keys, cb)`, `removeItem(k, cb?)`, `removeItems(keys, cb?)`, `getKeys(cb)`.
- `DeviceStorage.*` (9.0+) — 5MB local, same API shape.
- `SecureStorage.*` (9.0+) — 10 items, Keychain/Keystore-backed.

### Sharing & links
- `openLink(url, { try_instant_view? })`
- `openTelegramLink(url)`
- `openInvoice(url, cb?)` — cb status: `paid|cancelled|failed|pending`.
- `shareToStory(media_url, { text?, widget_link? })` (8.0+)
- `shareMessage(msg_id, cb?)` — prepared inline message id (8.0+)
- `switchInlineQuery(query, choose_chat_types?)`
- `sendData(data)` — only for keyboard-button launches; closes the app.

### Permissions / identity
- `requestWriteAccess(cb?)`
- `requestContact(cb?)`
- `requestEmojiStatusAccess(cb?)`
- `setEmojiStatus(custom_emoji_id, { duration? }, cb?)`

### Sensors (8.0+)
- `Accelerometer`, `Gyroscope`, `DeviceOrientation` — each: `start({ refresh_rate }, cb?)`, `stop(cb?)`, properties `x/y/z` or angles.
- `LocationManager.init(cb)`, `getLocation(cb)`, `openSettings()`.

### Biometrics
- `BiometricManager.init(cb)`, `requestAccess({ reason? }, cb)`, `authenticate({ reason? }, cb)`, `updateBiometricToken(token, cb?)`, `openSettings()`. Read-only flags: `isInited`, `isBiometricAvailable`, `biometricType`, `isAccessRequested`, `isAccessGranted`, `isBiometricTokenSaved`, `deviceId`.

### Misc
- `isVersionAtLeast(v)` — feature-detect every newer call.
- `setHeaderColor('bg_color'|'secondary_bg_color'|'#rrggbb')`
- `setBackgroundColor(color)` / `setBottomBarColor(color)`

## Events

`onEvent(name, fn)` / `offEvent(name, fn)`. Names:

- `themeChanged`
- `viewportChanged` → `{ isStateStable }`
- `safeAreaChanged`, `contentSafeAreaChanged`
- `mainButtonClicked`, `secondaryButtonClicked`
- `backButtonClicked`, `settingsButtonClicked`
- `invoiceClosed` → `{ url, status }`
- `popupClosed` → `{ button_id }`
- `qrTextReceived` → `{ data }`
- `clipboardTextReceived` → `{ data }`
- `writeAccessRequested`, `contactRequested`
- `biometricManagerUpdated`, `biometricAuthRequested`, `biometricTokenUpdated`
- `accelerometerStarted/Stopped/Changed/Failed`, `deviceOrientationStarted/Stopped/Changed/Failed`, `gyroscopeStarted/Stopped/Changed/Failed`
- `locationManagerUpdated`, `locationRequested`
- `fullscreenChanged`, `fullscreenFailed`
- `homeScreenAdded`, `homeScreenChecked`
- `emojiStatusSet`, `emojiStatusAccessRequested`, `emojiStatusFailed`
- `activated`, `deactivated`

## Theme CSS variables

```
--tg-theme-bg-color
--tg-theme-text-color
--tg-theme-hint-color
--tg-theme-link-color
--tg-theme-button-color
--tg-theme-button-text-color
--tg-theme-secondary-bg-color
--tg-theme-header-bg-color
--tg-theme-bottom-bar-bg-color
--tg-theme-accent-text-color
--tg-theme-section-bg-color
--tg-theme-section-header-text-color
--tg-theme-section-separator-color
--tg-theme-subtitle-text-color
--tg-theme-destructive-text-color
```

Plus runtime mirrors in `Telegram.WebApp.themeParams.bg_color`, etc.

## Bot API version timeline (load-bearing methods)

| Version | Adds |
|---|---|
| 6.0 | Mini Apps launch, MainButton, themeParams, CloudStorage stub |
| 6.1 | HapticFeedback, BackButton, `setHeaderColor` |
| 6.2 | `showPopup`, `showAlert`, `showConfirm` |
| 6.4 | `QrScanner`, clipboard read, inline mode helpers |
| 6.7 | `switchInlineQuery`, `openInvoice` |
| 6.9 | `CloudStorage`, `requestWriteAccess`, `requestContact` |
| 7.0 | `SettingsButton`, extended `themeParams` |
| 7.2 | `BiometricManager` |
| 7.7 | Disable vertical swipes |
| 7.10 | `SecondaryButton`, button positions, bottom bar color |
| 8.0 | Fullscreen, accelerometer/gyro/orientation, location, emoji status, `shareToStory`, `downloadFile` |
| 9.0 | `DeviceStorage`, `SecureStorage` |
| 9.5 | `iconCustomEmojiId` on buttons |

Always gate with `isVersionAtLeast()` — there are many old clients.

## Launch contexts

| Source | Has `query_id` | Can `sendData` | Notes |
|---|---|---|---|
| Keyboard button (`web_app`) | no | **yes** | The classic flow; closes app on sendData. |
| Inline button | yes | no | Use `answerWebAppQuery` on the bot side. |
| Menu button | yes | no | |
| Direct link `t.me/<bot>/<app>` | varies | no | Best for sharable URLs; `?startapp=...` → `start_param`. |
| Attachment menu | yes | no | May have no `user`; has `chat`/`receiver`. |
| Main Mini App (profile launch) | yes | no | Concurrent usage supported. |
| Inline mode | n/a | n/a | App composes a result via `switchInlineQuery`. |

## Init data field list

Sent in `initData` (query string) and parsed into `initDataUnsafe`:

- `query_id` — present for inline/menu/main launches
- `user` — JSON: `{id, is_bot, first_name, last_name?, username?, language_code?, is_premium?, allows_write_to_pm?, photo_url?}`
- `receiver` — for attachment menu launches in a chat
- `chat` — for attachment menu launches
- `chat_type` — `sender|private|group|supergroup|channel`
- `chat_instance`
- `start_param` — payload from `?startapp=`
- `can_send_after` — seconds, bot answer rate-limit hint
- `auth_date` — unix seconds; validate freshness
- `hash` — HMAC-SHA256, the thing you verify
- `signature` — Ed25519 signature (third-party validation flow)
