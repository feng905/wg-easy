---
title: LiveWireGuard Import
---

LiveWireGuard is a WireGuard client for iOS and macOS. It can provision its tunnel configuration directly from wg-easy: the user signs in with OIDC inside the app, and the app fetches its config from this panel — no QR code or config file download needed.

## Endpoint

The provisioning endpoint is:

```http
GET https://<your-domain>/api/livewireguard/config
```

The full URL (based on the address you are visiting the panel with) is also shown on the admin panel under **Admin Panel → General Settings → LiveWireGuard → Endpoint**. Fill it into the LiveWireGuard app's import wizard.

The endpoint must be reachable from the device running the app — in production that means HTTPS with a valid certificate.

/// note

The endpoint does not use wg-easy session authentication. Requests are authenticated with the OIDC tokens the app obtained from your identity provider.

///

## Setup

1. Configure OIDC login as described in [External Authentication](./external-authentication.md). The app and the panel can share the same identity provider.
2. Register the LiveWireGuard app as a **public client** in your identity provider:

   | Setting           | Value                                                          |
   | ----------------- | -------------------------------------------------------------- |
   | Client ID         | `livewireguard` (or your own — see [Audience](#audience))      |
   | Flow              | Authorization Code + PKCE (S256), token endpoint auth: `none`  |
   | Redirect URI      | `com.wuxichaorongtechnologycoltd.livewireguard:/oidc/callback` |
   | Scopes            | `openid profile email offline_access`                          |

3. Enable the feature: **Admin Panel → General Settings → LiveWireGuard → Enabled**.

## Audience

Every OIDC token carries an `aud` (audience) claim: the identity of the client the token was issued **for**. When the app calls the endpoint, wg-easy validates that claim and accepts the token if the audience is either

- the **Audience** configured in the panel (default `livewireguard`), or
- `OAUTH_OIDC_CLIENT_ID` (the panel's own OIDC client id).

So set the panel's **Audience** field to the client id your identity provider actually stamps into the app's tokens. If you registered the app with client id `livewireguard`, the default already matches. If it is something else — or if your provider reuses the panel's client — adjust the field accordingly. A mismatch makes the endpoint answer `401 invalid_token`.

## User mapping

The endpoint matches the OIDC identity to a wg-easy account exactly like panel login does:

1. by the OIDC subject (`sub`) of a previously linked account,
2. otherwise by the account's email, if the token marks the email as verified.

If no account matches, the request is rejected with `403 forbidden` — accounts are never created through this endpoint (enable `OAUTH_AUTO_REGISTER` to have panel login create them).

## Provisioned clients

The first sync creates one wg-easy client per device, named `LiveWireGuard-<device id prefix>`, and returns its configuration to that device only. The private key never leaves the phone: the app generates it locally, uploads the public key, and the downloaded config intentionally contains **no `PrivateKey` line** — the app pairs the config with its own key.

In the panel these clients show a **LiveWireGuard** badge, and the QR code / config download actions are hidden because downloading the config would be useless without the private key.

If the device is reinstalled it gets a new device id and a new key; the next sync rotates the public key of the existing client so the client (and its IP address) is kept.

## Disabling and revocation

- Disable a single device: set the client to disabled in the panel. The next sync answers `410 Gone`, which the app treats as a final revocation and removes the tunnel.
- Turn the feature off entirely: the endpoint answers `404` (a transient state for the app) and panel login is unaffected.
- Requests are rate limited to 60 per minute per IP (`429 Too Many Requests` with `Retry-After`).

Responses are cached by the app and revalidated with an `ETag` (`If-None-Match` → `304 Not Modified`), so periodic syncs are cheap. Any configuration change on the panel — addresses, DNS, endpoint, allowed IPs — changes the ETag and the app picks it up on the next sync.
