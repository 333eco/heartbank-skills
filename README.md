# HeartBank® API skills

Agent Skills for building with the HeartBank® APIs: how to add each one to an app, with the rules that come with it.

| Skill | What it teaches |
|---|---|
| [`log-in-with-heartbank`](skills/log-in-with-heartbank/SKILL.md) | Add Log in with HeartBank®: passkey log-in, and a log-out that ends both sessions |
| [`thank-with-heartbank`](skills/thank-with-heartbank/SKILL.md) | Add Thank with HeartBank®: prepare, the person's own press, commit |
| [`sign-with-heartbank`](skills/sign-with-heartbank/SKILL.md) | Let people sign what they made with Sign with HeartBank® (B-Origin℠) |
| [`verify-with-heartbank`](skills/verify-with-heartbank/SKILL.md) | Check a record, a receipt and the zero-sum proof from the public logs |
| [`look-up-with-heartbank`](skills/look-up-with-heartbank/SKILL.md) | Look up one exact @name and what its public page shows |
| [`confirm-with-heartbank`](skills/confirm-with-heartbank/SKILL.md) | Add Confirm with HeartBank®: one passkey press for one named action |

**An agent can do everything except the press.** Every thank and every signature needs the person's own passkey, so an agent can write the whole integration, and no tool here makes a thank or a signature for anyone. ⛔ Never hold a passkey for a person: a key an agent holds signs words nobody approved.

## Using them

- **As a Claude Code plugin.** This repo is a plugin and its own marketplace: run `/plugin marketplace add 333eco/heartbank-skills`, then `/plugin install 333eco@missaquarius` (in a shell: `claude plugin marketplace add 333eco/heartbank-skills` and `claude plugin install 333eco@missaquarius`). The skills arrive as `333eco:<skill>`, and the MCP server below as `heartbank`. Its public tools work at once; the tools for your own developer apps ask you to connect it once with your passkey (`/mcp`, then Authenticate). On claude.ai, installing the plugin doesn't add the server: add `https://developers.333.eco/mcp` on the Connectors tab.
- **With the MCP server.** Add `https://developers.333.eco/mcp` to your agent as a remote MCP server (streamable HTTP). It serves these same skills as `skill://` resources (the MCP Skills extension, SEP-2640), each file with its SHA-256 digest and size, alongside public, read-only tools.
- **Without it.** Copy a skill's folder into your agent's skills directory (for Claude Code, `.claude/skills/`). Each folder is a standard Agent Skill: a `SKILL.md` with `name` and `description`.

These files are the same bytes the MCP server serves; a change is made there first and published here unchanged.

## License

Your choice of CC0 1.0 (no rights reserved) or Apache 2.0: see `LICENSE`. Each skill's frontmatter says the same, so a copied folder carries it. Neither grants any right in the HeartBank® or other trademarks, which are not licensed here.

## More

The docs: https://developers.333.eco · everything for agents in one file: https://developers.333.eco/llms.txt

HeartBank® is a data bank of gratitude. It is not a chartered bank or financial institution; it does not provide banking services, hold deposits, or extend credit. No money moves through these APIs.
