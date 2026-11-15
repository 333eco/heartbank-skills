---
name: confirm-with-heartbank
description: Add Confirm with HeartBank® to an app, one passkey press for one named action by a person the app already knows from Log in with HeartBank®, a human in the loop so the app knows the person, not a script, did it. Use when a developer wants a person to confirm an action such as an upload with their own passkey, redeem the result on their server, or decide between Confirm and Sign. Nothing is kept, and the press page shows only words HeartBank® wrote.
license: CC0-1.0 OR Apache-2.0
---

# Add Confirm with HeartBank®

Confirm with HeartBank® asks a person your app already knows to press their own passkey for one action your app names. Your server gets a yes it can check once. Nothing is kept: no public record and no history line.

**An agent can do everything except the press.** That is what this API is for: a press with the person's own passkey, which no tool here makes for them. ⚠️ It is presence only as far as the device's own check goes: a passkey kept in software can press with nobody there.

## The rules that come with it

- **It proves presence, not personhood.** A result says the holder of this person's passkey pressed it, with their device's own check, for this action, just now. A PIN can satisfy that check, and a passkey can be shared or synced. ⛔ Never present a confirmed person as "verified".
- **Your app never puts its own words on the press page.** The kind of action comes from a closed list (today only `upload`), and the page says it in HeartBank®'s own words with your app's registered name. That is what keeps the page from being turned into a form that says something else.
- **Let them decide in peace.** ⛔ Never time them or nudge them on your side while they decide. A decline comes back as `#declined=1`: accept it as an answer, with no reminder.
- **Redeem once, and check what came back.** The `sub` must be the person you asked about, and the `hash` must be the file you hold, before you accept the action.
- **Confirm is not Sign.** Confirm leaves no record. For a file the person should put their name to, in public and for good, use Sign with HeartBank® (see the `sign-with-heartbank` skill).
- **It is metered to your app, never to the person.**
- **Your private key stays on your server.** Both calls are made with your app's own client assertion. ⛔ Never from a browser or a phone.

## Steps

1. **Have the person logged in** with Log in with HeartBank®, so your app has its own `sub` for them (see the `log-in-with-heartbank` skill). A person who last logged in to your app before 2026-11-11 is found from their next log-in.
2. **Ask:** `POST https://login.333.eco/confirm/request`, as form fields, with your client assertion (`client_assertion_type` and `client_assertion`, an ES256 JWT with `iss` = `sub` = your client id, `aud` = this endpoint's own address, a one-use `jti`; the same keys as your Log in client), the person's `sub`, the `kind` (`upload`), optionally the `hash` (the SHA-256 of the file, 64 lowercase hex; the page shows its first eight) and `return_to` (one of your registered redirect URLs). The answer is a `ticket`, good for 10 minutes. A `404` is the one answer for a `sub` nobody holds, a person not seen since 2026-11-11 and one with no passkey known.
3. **The person presses.** Send their browser to `https://sign.333.eco/confirm#ticket=…&state=…`, with the ticket in the fragment, never a query. The page shows your app's registered name and what is being confirmed, and the passkey sheet opens. The browser comes back to your `return_to` with `#confirm=<result>&state=…`, or `#declined=1&state=…`.
4. **Redeem** on your server: `POST https://login.333.eco/confirm/redeem`, form fields, your client assertion and `result`. The answer is `{confirmed: true, sub, kind, hash, at}`, once; a second redeem is refused as `replayed`, and a press by anyone but that person as `not_this_person`.
5. **Check** that `sub` is the person you asked about and `hash` is the file you hold. Only then accept the action.

Field-by-field detail and every error are in the reference: https://developers.333.eco/confirm.

## Check your work

- The action goes ahead only after a redeemed result whose `sub` and `hash` match.
- No screen in your app calls the person "verified" because they confirmed.
- Nothing of your app's own text is sent to the press page.
- No timer and no nudge while they decide; a decline is accepted.

## Reference

https://developers.333.eco/confirm · everything for agents in one file: https://developers.333.eco/llms.txt.
