---
name: sign-with-heartbank
description: Let the people who use an app sign what they made with Sign with HeartBank® (B-Origin℠), a fresh passkey signature over one file's fingerprint, made at sign.333.eco and recorded in a public log that anyone can check. Use when a developer wants a Sign and download button in a web app, the record of a signed file, a file's record found again after its metadata is gone, or a B-Trace℠ check of what a file carries. The file never leaves the person's device.
license: CC0-1.0 OR Apache-2.0
---

# Sign a file with Sign with HeartBank®

Sign with HeartBank® lets a person sign what they made, with their own passkey, on `sign.333.eco`. Your app gets a B-Origin℠ record: a signed, timestamped entry in a public log at `log.333.eco` that anyone can check with no help from HeartBank®. Only the file's fingerprint is signed; the file stays on the person's device.

**An agent can do everything except the press.** You can build the button and the return page; only the person signs.

## The rules that come with it

- **Sign is not Log in.** Logging in says who someone is for a while; signing puts their passkey to one exact file, with a fresh press every time. A log-in session can never sign, and ⛔ a log-in is never called "sign in".
- **It is the person's choice each time.** Signing makes a permanent public record, so keep a plain **Download** button beside **Sign and download**.
- **Going back without signing is fine.** A decline comes back as `back.declined`: give them their file anyway, with no question and no reminder.
- **Say once that it is on its way, never count it down.** *"Signed. It joins the public log within the hour."* is fine, once, as a plain fact. ⛔ Never a countdown or time ticking away. <!-- sig-ok: a prohibition (Sig-8) -->
- **Say what a record proves, and only that.** A passkey signed exactly this claim at this time, and the log can't later remove it. ⛔ It does not prove who imagined the work, or that one person holds one account: never call a file or a person "verified", "genuine" or "human-made" because of it, and never grade a work.
- **Showing the passkey is the person's call.** Pass `opening` only if they choose it: it adds the passkey's public key, so anyone can link every file carrying the same key.
- **Withdrawal is an event, never erasure.** A signer can withdraw a record; it shows as withdrawn.
- **Every record is real and permanent,** test signatures too. Don't sign test files you would not want public.

## Steps

1. **Ask to be a caller.** Signing identifies your app by the browser's origin and by your server's key, so your site and one return page are added by hand: ask on https://developers.333.eco/dashboard. ⏳ Signing from a native app is not built.
2. **Install the package:** `npm install @333eco/origin-attest`.
3. **The button.** Export the file once, and pass that same file:

```js
import { beginSign, finishSign, sourceFromHistory } from "@333eco/origin-attest";

// The button. Export once, and pass that file: a second export can change the bytes.
await beginSign(await exportPng(), {
  filename: "my-design.png",
  tool: "my-editor",                                   // your registered name
  source: sourceFromHistory({ imports, mostly: 0.8 }), // how it was made, from your own history
  rights: "CC0-1.0",                                   // optional
  vouch: (claim) => fetch("/api/vouch", { method: "POST", headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ sha256: claim.sha256, tool: claim.tool }) }).then((r) => r.text()), // your server signs for this file
});
// The person is now on sign.333.eco, which shows what they are signing and who asked. The file waits on their device, in IndexedDB.
// If this throws, nothing was signed: let them download anyway.
```

   **Your server vouches for each file.** The browser's origin names your app, and your server proves it asked: without the assertion below, sign.333.eco refuses the request (`unsigned`), and an app with no server key on file is refused (`no_key`). Give your app a server key URL on its dashboard page.

```js
import { SignJWT } from "jose";

// POST /api/vouch on YOUR server, for someone you let sign (signed in to your app; limit how often).
// privateKey: the ES256 key whose public half is at your app's server key URL (jwks_uri); kid: its id there.
const assertion = await new SignJWT({ sha256, tool })          // the two values your page sent, for this one file
  .setProtectedHeader({ alg: "ES256", kid })
  .setIssuer(SITE_ORIGIN)                                      // your website's origin, exactly as on your dashboard
  .setAudience("https://sign.333.eco/sign/par")
  .setIssuedAt().setExpirationTime("60s")                      // no more than 120 seconds
  .sign(privateKey);
```

4. **The return page,** in the browser:

```js
const back = await finishSign();
if (back) {
  back.download();                  // the same bytes they started with
  showAButtonThatCalls(back.download);
  // back.index is the record's number. back.declined means they went back without signing, and it's their file either way.
}
```

5. **Later, a pack** holds the record and everything needed to check it offline, once the hourly checkpoint has published:

```js
const pack = await packFor(back.index);   // { waiting: true } until the checkpoint
```

6. **Checking a record is free, forever,** with no account: see the `verify-with-heartbank` skill, or `npx @333eco/origin-verify`.

## A mail: hand someone a draft to sign (B-Mail)

Anyone can sign a mail at https://sign.333.eco/b-mail, with no key and no account: they write it, press their passkey over the exact words, and send it from their own mail app. You can hand a person a draft with a link shaped like `mailto:`. Everything after the `#` stays in their browser.

`https://sign.333.eco/b-mail#to=dara%40example.com&subject=Lunch%20on%20Friday&body=Shall%20we%20meet%20at%20noon%3F&made=trainedAlgorithmicMedia`

- **Fields:** `to`, `cc`, `subject`, `body`, `from`, `fromname`, `name` (their @name) and `made`, one of `digitalCreation`, `humanEdits`, `composite`, `compositeSynthetic` or `trainedAlgorithmicMedia`. Percent-encode each value. If you drafted the words, say so: `made=trainedAlgorithmicMedia`, or `compositeSynthetic` when the person wrote part.
- **The person reads and may change every word.** The page says a link filled it in, shows exactly what the press will sign, and makes a random salt of its own, so your link never knows the fingerprint.
- **Nothing comes back to you.** The page gives the signed message to the person alone. If you send mail for them, ask them to paste it back, and send exactly that: from the same address, to the same people, with the same subject, and with no files attached (attachments are not signed yet). A changed message shows its readers a mark that doesn't match.
- **Say what it shows, and only that:** a passkey signed these exact words. ⛔ Never that a person wrote them or pressed, and never "verified".
- ⛔ **Never hold a passkey for a person, and never press for one.** A passkey kept in software can press with nobody there, so nothing in the signature would show it; the only thing that keeps a signed mail honest is that the key stays with the person.
- To check one you received, see the `verify-with-heartbank` skill.

## B-Trace℠: what a file carries, from your server

`POST https://origin.333.eco/api/v1/trace`, with the file as the body (up to 25 MB) and your business's API key as a bearer token. The answer lists the file's B-Origin records, its C2PA manifest and IPTC fields, generator markers and watermarks found, and `notChecked`, what the answer did not look for.

- ⚠️ **It is metered** through the API, paid from your business's balance at billing.thonly.com (free in the portal). Prices are on https://developers.333.eco/sign.
- **Keep the key on your server.** A key inside an app isn't secret. A key can only look: it can never sign.
- ⛔ **B-Signal℠ is never a measure of how human a work is,** and is never shown as a grade of someone's work.
- A `503` with `log` means the log couldn't be read, so there is no answer rather than a wrong one. It is not a finding about the file.

## What costs what

| What | Cost |
|---|---|
| A person signing their own file at sign.333.eco | free, always |
| A record signed through your app | metered |
| A B-Trace check of a file through the API | metered (free in the portal) |
| Verifying a record's signature | free, anywhere, no key |

No one ever pays to be proven human.

## Check your work

- A plain Download button sits beside Sign and download.
- A decline still hands the person their file.
- No countdown, and no word that the record proves a person or a work. <!-- sig-ok: a prohibition (Sig-8) -->
- The API key never reaches a browser or a phone.

## Reference

https://developers.333.eco/sign · the package: https://www.npmjs.com/package/@333eco/origin-attest · everything for agents in one file: https://developers.333.eco/llms.txt.
