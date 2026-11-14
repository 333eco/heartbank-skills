---
name: log-in-with-heartbank
description: Add Log in with HeartBank® to an app, a passkey log-in through OpenID Connect at login.333.eco on the FAPI 2.0 profile (pushed requests, PKCE, private-key client assertions, DPoP). Use when a developer wants people to log in with HeartBank®, add a Connect HeartBank® button to an account they already have, read the proof-of-personhood layers (poh), or log people out. Log out ends both the app's session and the HeartBank® one.
license: CC0-1.0 OR Apache-2.0
---

# Add Log in with HeartBank®

Log in with HeartBank® proves who someone is, with a passkey. There are no passwords and no shared secret to store: your server talks to `login.333.eco` with its own private key.

**An agent can do everything except the press.** You can write the whole integration; only the person, with their own passkey, logs in.

## The rules that come with it

- **Say Log in and Log out.** ⛔ Never "sign in" or "sign out", in code comments, buttons or messages. On HeartBank® *Sign* is a different thing: Sign with HeartBank® puts a person's passkey to one exact file, and a log-in can never sign.
- **Log out ends both sessions.** End your app's own session, then send the browser to `https://login.333.eco/session/end` (with `client_id` and `post_logout_redirect_uri`). Ending only your own leaves the HeartBank® session alive: the next Log in comes straight back with no passkey press, and on a shared phone the next person to tap it becomes the last one.
- **A `sub` belongs to your app alone.** Another app gets a different value for the same person, so two apps can't link a person through it. Never try to.
- **A `sub` can stop appearing.** If someone joins two accounts, the joined account's `sub` ends and they reach you with the other one, as a new user. Offer your own way to move their data.
- **Require a passkey by checking `amr`**, not `poh`: `amr` says how *this* log-in happened (`swk` or `hwk` is a passkey), `poh` describes the account.
- **`poh` is the person's to release.** It appears only with the `poh` scope and their consent (today `["passkey"]`), and it is absent otherwise, never `false`. ⛔ Never label a person "verified" in your own words; if you show their status, use the passkey mark from the badge kit.
- **`/session/state` is display only.** Use it to choose between a Log in and a Log out button; never trust it for anything else.
- **No refresh tokens.** Log the person in again, or keep your own session.
- **Nothing unbuilt is promised.** Self-serve registration is approved but not built: apps are added on the dashboard and reviewed by hand.

## Steps

1. **Register the app.** Add it at https://developers.333.eco/dashboard, under a business, with the address of your server's public keys (a JWKS). It works once it is approved. The `poh` and `ledger` scopes need their own review.
2. **Read the discovery document**, https://login.333.eco/.well-known/openid-configuration (MCP tool `log_in_discovery`). Everything a library needs is there, and it is the authority if anything here disagrees.
3. **Check the library supports all of this:** the authorization code flow with scope `openid` (add `poh` for the proof-of-personhood layers) · pushed authorization requests (PAR), required · PKCE `S256` · client authentication `private_key_jwt`, ES256 · DPoP, required, ES256 · ID tokens ES256. Many older OpenID libraries lack PAR or DPoP.
4. **Use a library that has been run against the real provider**, and only these: `openid-client` 6.8.9 for Node.js (no extra code) · `Authlib` 1.9.2 for Python (you write PAR and DPoP by hand; full sample at https://developers.333.eco/python-authlib.py) · `@333eco/firebase-bridge` to turn the ID token into a custom token in your own Firebase project.
5. **Write the flow.** In Node.js, with `openid-client`:

```js
import * as oc from "openid-client"; // openid-client 6.8.9: everything below is built in

const config = await oc.discovery(new URL("https://login.333.eco"), CLIENT_ID,
  { id_token_signed_response_alg: "ES256", token_endpoint_auth_signing_alg: "ES256" },
  oc.PrivateKeyJwt({ key: privateKey, kid: "your-key-id" }));

// 1. Send the person to log in. Keep verifier, nonce and state in your session.
const verifier = oc.randomPKCECodeVerifier(), nonce = oc.randomNonce(), state = oc.randomState();
const url = await oc.buildAuthorizationUrlWithPAR(config, {
  redirect_uri: REDIRECT_URI, scope: "openid", nonce, state,
  code_challenge: await oc.calculatePKCECodeChallenge(verifier), code_challenge_method: "S256",
});
redirect(url);

// 2. They come back to REDIRECT_URI.
const DPoP = oc.getDPoPHandle(config, await oc.randomDPoPKeyPair("ES256"));
const tokens = await oc.authorizationCodeGrant(config, callbackUrl,
  { pkceCodeVerifier: verifier, expectedNonce: nonce, expectedState: state, idTokenExpected: true },
  undefined, { DPoP });
const { sub, amr } = tokens.claims();
```

6. **Pick the button label by where it sits.** *Log in with HeartBank®* where people enter your app; *Connect HeartBank®* inside your app, for someone adding HeartBank® to the account they already have (tie the `sub` to that account and keep your own log-in). Same flow, same page; our page always says Log in.
7. **Wire log-out both ways.** Register a `backchannel_logout_uri`: login posts a signed logout token there when the person's HeartBank® session ends. Your own Log out button ends your session and then sends the browser to `https://login.333.eco/session/end`.
8. **If you read `poh` again later,** register a change address: login posts a signed event that says only that something changed, and your server asks `POST https://login.333.eco/poh` (at most once per person per 24 hours). The person can see every lookup and can disconnect your app at any time.

## Check your work

- After your Log out, tapping Log in asks for a passkey again.
- No screen, message or comment says "sign in".
- The app's private key stays on its server; nothing secret ships in a browser or a phone.
- Nothing calls a person "verified"; a passkey requirement reads `amr`.

## Reference

The guide and every operation, generated from the OpenAPI source: https://developers.333.eco/log-in. To have a person confirm one action your app names with a passkey press, see https://developers.333.eco/confirm. Everything for agents in one file: https://developers.333.eco/llms.txt.
