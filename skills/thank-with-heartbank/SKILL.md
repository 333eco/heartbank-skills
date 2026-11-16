---
name: thank-with-heartbank
description: Add Thank with HeartBank® to an app, recording a thank on the gratitude ledger in three steps (your server prepares it, the person presses their own passkey at sign.333.eco, your server commits it) and reading a person's ring. Use when a developer wants their app to record thanks (Kiitos), handle the ledger's answers, or draw a person's ring. First-party apps only today; outside apps write from season 2 (7 January 2027), after review.
license: CC0-1.0 OR Apache-2.0
---

# Add Thank with HeartBank®

Thank with HeartBank® records a thank on the gratitude ledger. Your app never writes a thank alone: the person signs every thank with their own passkey, so no app and no agent can manufacture gratitude.

**An agent can do everything except the press.** You can write the whole integration, prepare and commit; the press is always the person's own. (The one exception is the sandbox below, where a test passkey presses for a test person, who is nobody.)

## Who can write today

- **First-party HeartBank® apps only.** Outside apps write from season 2 (7 January 2027), after review.
- **From season 2, an outside app writes Kiitos, and Kiitti only as a person's thanks to all of life:** a call with no direction, no twin and no evidence field, whose shape is not published yet. ⛔ An app never claims care for a plant, an animal or a thing: that is witnessed, never claimed by an app.
- If you are building for season 2 now, build the flow below, and do not ship a write until your app is admitted.

## The rules that come with it

- **The press is the person's.** No test, script, agent or server presses for a real person. If your design needs a thank without the person's press, the design is wrong. The sandbox is the one place a press isn't a person's, and nothing made there works anywhere else.
- ⛔ **Never pre-fill a thank.** The person starts it and chooses whom to thank. No thank prepared before they ask for one, no recipient chosen for them, no box already ticked, and never a thank that happens by default unless they opt out.
- ⛔ **No counts, streaks or countdowns on screen.** Never show how many thanks someone gave or received, never a streak, never a time left until anything. A person's ring is a color and a width: draw it as a ring, never as a number to chase. <!-- sig-ok: a prohibition (Sig-8) -->
- **A refusal says nothing.** When the ledger declines, the answer is only `{"written": false}`: no reason, no count, no next time. It protects the people involved, and it is not a bug. Tell the person once, plainly, that the thank wasn't recorded. Never guess a reason, never show a wait, and never retry in a loop.
- ⛔ **Your app never holds another person's token.** The receiver is not yours to carry. Today's `to` (a log-in token and `sub`) is for first-party apps; from season 2 an outside app names the receiver by their @name.
- **Your app gets only the receipt's id.** By design it cannot locate the thank in the public log (that would let apps count one another's thanks). The full receipt goes to the two people, in their own records; point them to https://earth.333.eco/you to see and check it.
- **A thank carries no money.** There is no amount field, and sending one is refused. Never describe a thank as a payment, a tip, a deposit or a balance.

## Steps

1. **Log the person in** with Log in with HeartBank® and the `ledger` scope (it needs review), which gives your server their `pid_token`.
2. **Prepare** on your server: `POST https://ledger.333.eco/v1/prepare`. Your server proves itself in the `x-client-assertion` header with a fresh ES256 JWT (`iss` = `sub` = your client id, `aud` = `https://ledger.333.eco`, at most 5 minutes old, a `jti` never used before). ⛔ Never from a browser or a phone. Give each thank your own `ref`: the same `ref` returns the same answer and writes once. The answer is `{pending, ticket}`, or `{"written": false}`. At most 6 prepares a minute per person.
3. **The person presses.** Send their browser to `https://sign.333.eco/press#ticket=…&return_to=…&state=…`. The page names your app and shows them exactly what they are signing, and they press their passkey. Their browser comes back to your `return_to` with `#press=…&state=…`.
4. **Exchange the press** on your server: `POST https://login.333.eco/press/wrap`, form field `result`, for a `pressToken`. A press is not a log-in.
5. **Commit**: `POST https://ledger.333.eco/v1/commit` with `ref`, `pending` and `pressToken`. The answer is `{written: true, receipt: {id}}`, or `{"written": false}`. It is safe to retry: the answer is kept under `ref`.
6. **Optionally place it on the public map**, later than the thank so the time doesn't point at it: `POST https://ledger.333.eco/v1/place` with the one-use token the commit returned for that side and a cell about 100 km across. There is no field for a coordinate.
7. **Read a ring** for the logged-in person: `GET https://ledger.333.eco/v1/aura`. A color and a width per ledger your app may read; no balance, no count.

Field-by-field detail, and every error, is in the reference, generated from the same source as the API's own description: https://developers.333.eco/thank.

## Try it in the sandbox first

`https://sandbox.333.eco` runs the ledger's own code, with test persons and test passkeys. Nothing in it is real. Docs: https://developers.333.eco/sandbox.

- **The same calls, another address:** `https://sandbox.333.eco/ledger/v1/prepare`, `/commit`, `/place`, `/aura` (your assertion's `aud` is `https://sandbox.333.eco/ledger`), and `https://sandbox.333.eco/press/hint`, `/press` (the page) and `/press/wrap` in place of login.333.eco's and sign.333.eco's. Same requests, same answers, and refusals just as silent.
- **Your app is in it** once it has asked for Log in on the dashboard (approval isn't needed there). The sandbox knows it by its Log-in client id (`hbc_…`) and checks its assertions against the key URL it registered.
- **Test persons** in place of a log-in: `POST https://sandbox.333.eco/persons` with `{"label": "Alice"}` (your assertion in `x-client-assertion`, `aud` = the exact address called) gives `{id, label, sub, pidToken}`; `GET /persons` lists them with fresh tokens. Or the app's Sandbox card on the dashboard, or the developer tool `sandbox_add_person`. At most 20 per app.
- **The press, here only, can come from code:** `POST https://sandbox.333.eco/press/api` with `{ticket, person}` gives the `result` the page would bring back; or the developer tool `sandbox_press`. Then `/press/wrap` and commit as above.
- A season a week: every Monday 00:00–00:15 UTC nothing is written while last week's log gets its last checkpoint, then the thanks and the places are wiped and a new log begins. Each week's log is its own, named `sandbox.333.eco/log/<its Monday>` as its receipts name it, and kept 7 days after the week. Test persons stay. At most 1,000 prepares per app per UTC day.
- ⛔ Nothing made in the sandbox works anywhere else, and a sandbox press can't even be read by the permanent ledger. Change the base address only when your app is admitted.

## Check your work

- Every thank in your app passes through the person's press at sign.333.eco.
- Nothing is pre-filled, and nothing thanks by default.
- No screen shows a count, a streak, a countdown or a number beside a ring. <!-- sig-ok: a prohibition (Sig-8) -->
- A `{"written": false}` is shown once, plainly, with no reason invented.
- No other person's token is stored, logged or sent anywhere by your app.
- The client assertion is made only on your server.

## Reference

https://developers.333.eco/thank · everything for agents in one file: https://developers.333.eco/llms.txt.
