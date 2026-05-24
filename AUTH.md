# Authentication deep-dive

This file expands the auth section of SKILL.md. Read SKILL.md first.

## End-to-end flow

```
[Telegram client]                [Your Mini App]              [Your backend]
  starts mini app                                                
  injects initData  ───────────►  reads Telegram.WebApp.initData
                                  POST /auth { initData } ────► verify HMAC
                                                                check auth_date
                                                                upsert user
                                  ◄──── { sessionToken } ──────  sign JWT / set cookie
                                  store token
                                  use for subsequent API calls
                                  Authorization: Bearer <token> ─► verify JWT
```

Key idea: validate `initData` **once** at session creation. Issue your own short-lived token. Do not revalidate `initData` on every call — it is fine to, but it bloats every request.

## HMAC validation (first-party, you own the bot)

Algorithm:

```
secret_key   = HMAC_SHA256(message=BOT_TOKEN, key="WebAppData")
data_check   = sort(keys without "hash") joined by "\n" as "k=v"
computed     = HMAC_SHA256(message=data_check, key=secret_key)
ok           = constant_time_eq(computed.hex, provided_hash)
fresh        = (now - auth_date) < max_age
```

### Node / TypeScript

```ts
import crypto from 'node:crypto';

export function verifyInitData(initData: string, botToken: string, maxAgeSec = 86400) {
  const params = new URLSearchParams(initData);
  const hash = params.get('hash');
  if (!hash) return null;
  params.delete('hash');
  // optional: remove signature if present so it does not enter the check string
  params.delete('signature');

  const dataCheckString = [...params.entries()]
    .sort(([a], [b]) => a.localeCompare(b))
    .map(([k, v]) => `${k}=${v}`)
    .join('\n');

  const secret = crypto.createHmac('sha256', 'WebAppData').update(botToken).digest();
  const computed = crypto.createHmac('sha256', secret).update(dataCheckString).digest('hex');

  const a = Buffer.from(computed, 'hex');
  const b = Buffer.from(hash, 'hex');
  if (a.length !== b.length || !crypto.timingSafeEqual(a, b)) return null;

  const authDate = Number(params.get('auth_date'));
  if (!Number.isFinite(authDate)) return null;
  if (Date.now() / 1000 - authDate > maxAgeSec) return null;

  const userJson = params.get('user');
  return {
    user: userJson ? JSON.parse(userJson) : null,
    chat: params.get('chat') ? JSON.parse(params.get('chat')!) : null,
    queryId: params.get('query_id') ?? null,
    startParam: params.get('start_param') ?? null,
    authDate,
  };
}
```

### Python

```python
import hmac, hashlib, time
from urllib.parse import parse_qsl

def verify_init_data(init_data: str, bot_token: str, max_age: int = 86400):
    parsed = dict(parse_qsl(init_data, keep_blank_values=True))
    received_hash = parsed.pop("hash", None)
    parsed.pop("signature", None)
    if not received_hash:
        return None
    data_check = "\n".join(f"{k}={parsed[k]}" for k in sorted(parsed))
    secret = hmac.new(b"WebAppData", bot_token.encode(), hashlib.sha256).digest()
    computed = hmac.new(secret, data_check.encode(), hashlib.sha256).hexdigest()
    if not hmac.compare_digest(computed, received_hash):
        return None
    if time.time() - int(parsed.get("auth_date", 0)) > max_age:
        return None
    return parsed
```

### Go

Use `github.com/Telegram-Mini-Apps/init-data-golang`:

```go
import initdata "github.com/Telegram-Mini-Apps/init-data-golang"

if err := initdata.Validate(rawInitData, botToken, time.Hour); err != nil {
    // reject
}
data, err := initdata.Parse(rawInitData)
```

## Third-party validation (Ed25519)

Use when the service verifying `initData` does **not** have the bot token (partner backends, analytics, multi-tenant aggregators).

Algorithm:
- `signed_string = "{bot_id}:WebAppData\n" + data_check_string`
- Verify `signature` (base64url) against Telegram's known **Ed25519 public key** — there is a separate key for test and prod environments. The keys are published by Telegram and rotate rarely.
- Same freshness check on `auth_date`.

Field `signature` is present alongside `hash` since Bot API 7.x. If a launch lacks `signature` (older clients), fall back to HMAC if you own the bot, or refuse.

## Session and refresh

- Mint a JWT with `sub = telegram_user_id`, short TTL (e.g. 15 min), plus a longer-lived refresh token stored server-side.
- For the refresh path: the Mini App can always re-send the original `initData` (it stays in `window.Telegram.WebApp.initData` for the entire launch), so a "logout + reauth" without UI is free.
- Bind sessions to a single Telegram `user.id`. Do not key by `query_id` — it changes per launch.

## Multi-tenant bots

If you operate many bots from one backend:
- Map the bot from the **bot id** baked into a per-tenant subdomain / URL path, not from `initData` (the field has no `bot_id` in the first-party HMAC flow).
- Use a separate bot token per tenant; store them encrypted at rest.

## Edge cases that bite

- **Missing `user`.** Attachment-menu launches in groups/channels may omit `user` and only include `chat`/`receiver`. Branch your auth code: no user means "anonymous-in-chat-context", not "invalid".
- **`is_bot: true`.** Some launches can include a bot user. Decide policy explicitly.
- **`language_code` is BCP-47-ish but not exhaustive.** Default to `en` and treat it as a hint, not truth.
- **`photo_url` is optional and expires.** Cache server-side if you need stability.
- **Replay.** Without `auth_date` freshness, a leaked `initData` is a forever-credential. Enforce 24h max, ideally less.
- **URL-encoding gotchas.** Build `data_check_string` from the **raw** URL-decoded values. The `URLSearchParams` API decodes for you; do not double-decode.
- **Stripping fields.** Newer Telegram clients add fields (`signature`, future ones). The HMAC check must include every field present *except* `hash` (and exclude `signature`, which is for the Ed25519 path). If you whitelist fields you will break on new releases.

## Login Widget vs Mini App auth

The Telegram **Login Widget** (the iframe used on regular websites) uses a *different* signed payload than Mini Apps. The HMAC scheme is similar but the field set and the wrapping URL differ. Do not reuse Mini App validators for Login Widget data and vice versa.

## TON wallet attachment

If your auth also includes a TON wallet:
1. Verify `initData` first → you now know the Telegram user.
2. Initiate TON Connect via `@tonconnect/sdk` / `@tonconnect/ui`.
3. Ask the wallet to sign a **server-issued nonce** (not just the user id) — defeats replay.
4. Store the mapping `telegram_user_id → ton_address` server-side.

Never bind sessions to "the wallet that signed" alone; that lets anyone with a wallet impersonate any Telegram identity.

## Testing

- Generate fixtures with a fake `BOT_TOKEN` in tests, build a valid `initData` query string, and assert:
  - happy path returns the parsed user
  - tampered `hash` returns null
  - stale `auth_date` returns null
  - missing `hash` returns null
- Property-based test the sorting/joining step — easy to get wrong with Unicode keys (should not happen, but be defensive).
