---
name: verify-with-heartbank
description: Check what HeartBank® publishes without trusting HeartBank®, from the public logs at log.333.eco. A B-Origin℠ record, a person's own thank receipts, the gratitude ledger's zero-sum proof and a B-Mail's signature, free forever, with no key and no account. Use when a developer or agent needs to verify a record or a receipt, check the zero-sum proof, read the logs' checkpoints and timestamps, or say plainly what a check does and does not show.
license: CC0-1.0 OR Apache-2.0
---

# Verify a record, a receipt and the zero-sum proof

Two public, append-only logs at `log.333.eco` hold what HeartBank® publishes: `origin` (every record signed with Sign with HeartBank®) and `ledger` (every thank, with its zero-sum proof). They are plain files in the open tlog-tiles format, so any HTTP client can read them. Nothing here is metered, and nothing asks who you are.

**An agent can do everything except the press.** Checking needs no press at all: an agent can run every check below on its own.

## The rules that come with it

- **Each result stands alone.** `included` (the operator's keys and the Merkle proof) · `witnessed` (outside witnesses) · `timestamped` (each timestamp authority's word) · `bitcoin` (Bitcoin's proof of work). None implies another. ⚠️ `witnessed` is not counted yet: the logs are on the Witness Network's testing list. Report it as not counted, never as passed.
- **A check verifies; it never grades.** A record shows that a passkey signed exactly this claim at this time, never who held the passkey, who imagined the work, or that one person holds one account. ⛔ Never turn a check into "verified human", "genuine" or a score.
- **A failure to read is not a finding.** If a log or a file can't be fetched, say the check couldn't run. ⛔ Never report "invalid" or "not signed" because the network failed.
- **The logs only grow, and anyone can test that.** They do not yet stop a log from showing different people different histories; outside witnesses catch that, and they aren't counted yet. Say so when asked what a check rests on.
- **An app holds only a thank's id.** By design it cannot locate the thank in the log. Only the two people hold the full receipt, so only they check their own thanks.
- **A B-Trace℠ check is a different thing.** Looking up what a file carries is metered through the API (free in the portal); checking a record is free.

## Steps

**A B-Origin record** (a signed file), with the open package, which has no runtime dependencies and pins the log's keys:

```sh
npx @333eco/origin-verify file picture.png    # finds the file's records by what it still is, and checks each one
npx @333eco/origin-verify record 0            # one record: signature, place in the tree, witnesses, clocks, Bitcoin
npx @333eco/origin-verify pack 0 record.json  # saves a pack
npx @333eco/origin-verify check record.json   # checks it, reading no network
npx @333eco/origin-verify audit               # the whole log, recomputed from its public tiles
```

`file` finds a file's records by what it still is, even after its metadata is gone. In a browser, https://explorer.333.eco/origin recomputes the same checks.

**A person's own thanks.** They open their record at https://earth.333.eco/you, after a fresh passkey log-in; each row's receipt can be checked at https://explorer.333.eco/ledger. After a season closes, they download the record and run the checker on their own computer (the file is sent nowhere); it names any thank of theirs that did not check out, never a balance, a total or a count:

```sh
node check-mine.mjs heartbank-record.json
```

**The zero-sum proof.** It shows, without revealing a thank, a pair, a person, a place, a balance or an amount, that every counted thank carries a passkey signature, that each moves exactly its season's rule, that none is counted twice, and that each sea of Kiitos and Kiitti adds up to zero. Only published windows exist: list them first with the MCP tool `list_zero_sum_proofs`, then fetch one with `verify_zero_sum_proof` and its exact `from` and `to`. Never guess a range. Fetching a file is not checking it; the checker proves it:

```sh
npm ci
node check.mjs           # the newest file's proofs, and the hashes and chain of all
node check.mjs --deep    # every proof
npm test                 # offline: a real published proof from the test tree, then tampered copies
```

The same checker runs in a browser at https://earth.333.eco/proof. It does not check the log's own signature; https://explorer.333.eco/ledger does that.

**A B-Mail.** The MCP tool `check_statement` checks a signed statement against the log and the @name it claims. Pass only its SHA-256 and nothing of the message reaches the server. Pass the statement and the delivered mail's fields to compare them too; a mismatch means the message is not what was signed. ⛔ Give the mail's HTML part as `html` whenever it has one: it is the part a reader is shown, and it must say the signed words too. The statement and its proof are in the mail's check link (`https://explorer.333.eco/b-mail#m=…&p=…`, base64url), which may sit behind the words *Signed with HeartBank®* or in the mail's text; a link inside quoted text belongs to the mail being quoted, not to this one. A mail security service may rewrite every link in a mail (Proofpoint URL Defense, Microsoft Safe Links): unwrap it on your own machine, never by asking the rewriter, before you look for the check link, and take a link as the check link only if it is then `https://explorer.333.eco/b-mail#…`; `check_statement` compares a rewritten link in the text as its original. Pass the SHA-256 of each file the mail carries as `attachments`, leaving out any file named `b-mail.txt` or `b-mail-proof.json`, in any case; they must be exactly the signed fingerprints, so a fingerprint listed twice needs two files. The text is compared as words, so a mail app's re-wrapped lines change nothing, and an added or changed word does. In a browser: https://explorer.333.eco/b-mail.

**The logs themselves.** The MCP tools `verify_checkpoint`, `verify_checkpoint_at` and `verify_stamps_index` read the signed checkpoints and which of them carry a timestamp authority's token or a Bitcoin proof.

## What a check cannot show

- A passkey shows that someone holds a key, not that they are one human.
- Two people can still thank each other for show.
- A thank left out of the proof looks like padding; the person whose thank it is can find that out when the season ends.

## Reference

https://developers.333.eco/verify · the checker: https://github.com/333eco/zero-sum-proof.333.eco · the package: https://www.npmjs.com/package/@333eco/origin-verify · everything for agents in one file: https://developers.333.eco/llms.txt.
