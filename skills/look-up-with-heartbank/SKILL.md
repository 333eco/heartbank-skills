---
name: look-up-with-heartbank
description: Look up one exact @name on B-Registry℠ with Look up with HeartBank® and get exactly what that person's public page already shows, the name as they wrote it, their one line and links, their photo, their own recording of their name, the language they chose to show, and the passkeys that sign as that name. Free, with no key. Use when an app needs to show or say a name the way its holder wrote it, link to their page, write to them in the language they chose, or check a B-Mail's passkey against the name it was signed as.
license: CC0-1.0 OR Apache-2.0
---

# Look up a name

Look up with HeartBank® answers one exact `@name` with what that person's page at `333.eco/@name` already shows, and nothing else. It is free and needs no key.

**An agent can do everything except the press.** Looking up needs no press; it only reads what a person chose to make public.

## The rules that come with it

- **One exact name per call.** ⛔ Never an identity (no account, no ID, no token), never from a person to their name, and never a list. Don't build a directory, a search, a crawler or an index of names from it.
- **One answer for every name with no page.** A name nobody holds, a name held with no page and a name let go all get the same `404`. ⛔ Never tell anyone a name is free to claim, has lapsed or was let go because of it.
- **`searchable` is the holder's choice.** When it is `false`, show the name only to someone who already typed it, and never put it in a list or an index of your own.
- **`lang` is shown only because they chose to.** Write to them in it. A `null` says nothing about what they read: ⛔ never guess a language from a name, a photo or a place.
- **Show their autograph only as a drawing.** When `autograph` is there, it is their B-Autograph℠, drawn by hand and shown by their own choice. Draw its strokes as they are. Anyone can copy a drawing: never use it to check who someone is, and never compare it with another.
- **Say their name the way they say it.** When `voice` is there, it is their own recording of their name: play it rather than a synthesized guess.
- **`keys` says which passkey signs as this name, never who held it.** An empty list says nothing at all: ⛔ never show a person as "unverified" because of it, and never call anyone "verified".
- **60 lookups a minute per address.** More are refused with a `429` and a `Retry-After`: wait that long, never retry in a loop. A change the holder saves shows within a minute, so don't keep an answer long.
- **@names and free addresses.** A person with no @name has a free address, `B:7K3M-9XD2-QH4N`, which works the same way (exact, the `B:` required). `#tag` names have no public record yet.

## Steps

1. **Look it up:** `GET https://333.eco/api/v1/names/{name}`, or the MCP tool `lookup_by_name`. The name may come with or without `@`; the registry folds case and punctuation, so `sokha.keo`, `SOKHA-KEO` and `sokhakeo` are one name. A free address goes in the same place, with its `B:` (`B:7K3M-9XD2-QH4N`, any case, hyphens optional); one whose holder sends visitors to their @name answers that name's page.
2. **Use what comes back:** `name` (in the form its holder chose, with `@`, or the address as shown) · `page` (link to it) · `said` (what they are called, as they wrote it, in any script) · `line` · `links` (up to three) · `photo` · `voice` · `autograph` (strokes on a 100 × 60 pad) · `searchable` · `keys` · `lang`.
3. **To check a B-Mail's passkey against the name it claims,** use the MCP tool `check_statement`, which reads `keys` for you (see the `verify-with-heartbank` skill). If the passkey that signed is listed, and the mail was signed before that key's `until`, it was signed with this name's passkey. That still says which passkey, never who held it.

## Check your work

- Every lookup is one name someone typed or followed a link to; nothing enumerates names.
- A `404` reads as "no page for this name" and nothing more, for an address too: never "this address isn't anyone's".
- A non-searchable name never lands in a list, a sitemap or a search index.
- No language is guessed; no one is called "verified" or "unverified".

## Reference

https://developers.333.eco/lookup · everything for agents in one file: https://developers.333.eco/llms.txt.
