# CleanSlop MCP server and agent skills

Readiness scans from your coding agent. [CleanSlop](https://cleanslop.com) checks vibe-coded TypeScript and JavaScript apps for what could break once real people use them, and tells you what to fix first. With the MCP server or the skill, your agent runs the loop itself: it scans the commit you pushed, reads the findings worst first, fixes them and rescans to confirm.

This repository holds:

- **[`server.json`](./server.json)**: the [MCP Registry](https://registry.modelcontextprotocol.io) entry for the remote server `com.cleanslop/cleanslop`.
- **[`skills/cleanslop/SKILL.md`](./skills/cleanslop/SKILL.md)**: the agent skill. It covers when to scan, how to start and wait, how to read the report, and when to stop and ask you.
- **A Claude Code plugin and marketplace ([`.claude-plugin/`](./.claude-plugin))**: the skill bundled with the MCP server.

The CLI lives in [cleanslop/cli](https://github.com/cleanslop/cli).

## MCP server

| | |
|---|---|
| URL | `https://cleanslop.com/mcp` |
| Transport | Streamable HTTP (remote; nothing to install) |
| Auth | OAuth 2.1 with PKCE, approved in your browser. No API key. |
| Setup guide | <https://cleanslop.com/mcp> |

You need a CleanSlop account (GitHub sign-in) and the CleanSlop GitHub App on the repo. The free plan includes 10 scan credits a month.

### Connect

Claude Code:

```bash
claude mcp add --transport http --scope user cleanslop https://cleanslop.com/mcp
```

Codex:

```bash
codex mcp add cleanslop --url https://cleanslop.com/mcp
codex mcp login cleanslop
```

Cursor (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "cleanslop": { "url": "https://cleanslop.com/mcp" }
  }
}
```

VS Code (`.vscode/mcp.json`):

```json
{
  "servers": {
    "cleanslop": { "type": "http", "url": "https://cleanslop.com/mcp" }
  }
}
```

For other clients, see <https://cleanslop.com/mcp>.

### Tools

| Tool | What it does |
|---|---|
| `whoami` | Shows which CleanSlop account the agent is connected as. |
| `scan_start` | Starts a scan of a pushed commit on a repo the account can access. |
| `scan_wait` | Waits for a running scan, with an ETA and a hint for when to check again. |
| `scan_status` | Returns a scan's current status without waiting. |
| `report_get` | Reads the report in pages, worst first, optionally compared with an earlier scan. |
| `promo_redeem` | Redeems a promo code the human gives it for free scans. |
| `send_feedback` | Sends the human's feedback to the CleanSlop team, such as a finding they say is wrong. |
| `feedback_replies` | Shows the team's replies to that feedback. |
| `email_opt_in` | Emails the report to the account owner after the human agrees. |

## Claude Code plugin

The plugin bundles the skill and the MCP server. In Claude Code 2.1.275 or later:

```text
/plugin install cleanslop --marketplace cleanslop/skills
```

On older versions, add the marketplace first:

```text
/plugin marketplace add cleanslop/skills
/plugin install cleanslop@cleanslop
```

The first time Claude uses the server, Claude Code opens `cleanslop.com` in your browser. You sign in with GitHub and approve access once (scopes `scan`, `report` and `email`). You can also run `/mcp` and pick `cleanslop` to connect ahead of time.

For Cursor and Codex, `npx -y @cleanslop/cli@latest setup` installs the same skill as a Cursor rule or an `AGENTS.md` block and registers the MCP server.

## What it runs and sends

- **Scans of pushed code (GitHub sign-in).** For a clean commit that is pushed to github.com, the plugin sends the repository URL, commit and branch to CleanSlop, and CleanSlop reads the code from GitHub through its read-only GitHub App, as committed. Your local files are not sent.
- **The CleanSlop CLI, when MCP can't do the job:** when the server isn't connected, to sign in with Stripe Link, or to scan local files. The skill then runs `npx -y @cleanslop/cli@latest`, the open-source CLI published from [cleanslop/cli](https://github.com/cleanslop/cli) to npm with provenance. It stores its token in `~/.config/cleanslop/credentials.json` (mode `0600`) and never prints it.
- **Local files with a GitHub sign-in, only if you choose to.** When there are uncommitted or unpushed changes, or the repository isn't on GitHub, the CLI asks you first. Only if you pick "Scan local files" does it pack the working tree and upload it to CleanSlop.
- **Local files with a Stripe Link sign-in, on every scan.** A Link sign-in scans by upload: every scan packs the working tree and uploads it to CleanSlop without asking first.
- **What an upload contains.** The files git tracks or would track, except `.env` files, keys and credential files, which never leave your machine. `cleanslop scan --dry-run` lists the files before anything goes up. Uploads are not used for training.
- **Payments, only with your approval.** Claude never approves or pays. With a GitHub sign-in, you buy scan credits or a plan on cleanslop.com, and Claude only passes on the checkout link. With a Stripe Link sign-in, you approve every payment in your Link app.

Reports are private to your CleanSlop account.

## Links

- Setup guide and FAQ: <https://cleanslop.com/agents>
- Privacy policy: <https://cleanslop.com/privacy>
- Terms: <https://cleanslop.com/terms>
- Issues: <https://github.com/cleanslop/skills/issues>

This repository is a read-only mirror; changes land upstream and sync here. [MIT](./LICENSE) licensed.
