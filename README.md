# Agile Analytics local AI connector

Ask Claude, or another assistant that supports the Model Context Protocol
(MCP), about your team's Agile Analytics numbers: sprint summary, cycle
time, flow efficiency, work in progress, aging and forecast. The connector
runs on your own computer, reads Azure DevOps with your own access, and
answers with the same numbers the Agile Analytics extension shows, because it
runs the extension's own metric code.

It is part of your Agile Analytics subscription: it answers only when the
extension is installed in your organisation and a workflow mapping is saved
for the project. Nothing is sent to Baytek.

Full guide: https://ado-analytics.baytekdev.com/docs/local-ai-connector/

## Before you start

- **Node.js 22 or later** on your computer (`node --version`). The connector
  runs with `npx`, which comes with Node.js.
- **Agile Analytics installed** in your Azure DevOps organisation, with a
  **workflow mapping saved** for the project you ask about (Configuration →
  Workflow Mapping in the extension).
- **The admin switch on** (it is on unless an admin turned it off; see
  "Turning it off for your organisation" below).

## 1. Create a personal access token

In Azure DevOps, open **User settings → Personal access tokens → New Token**.
Pick your organisation, an expiry date, and exactly these three scopes:

- **Work Items: Read**
- **Project and Team: Read**
- **Extension Data: Read**

Nothing else is needed. The token only lets the connector read; it cannot
change anything.

### Where the token goes

- **Windows:** set it as a **user environment variable named
  `AZURE_DEVOPS_PAT`** (Start → "Edit environment variables for your
  account" → New). Keep it out of the assistant's config file. Restart your
  assistant afterwards so it sees the new variable.
- **macOS and Linux:** desktop assistants do not read your shell profile, so
  put the token in the server entry's **`"env"` block** as
  `AZURE_DEVOPS_PAT` (shown in the examples below), and keep that config file
  readable by your user account only (`chmod 600 <file>`).

The token stays in the connector's memory. It is never printed, logged or
written anywhere.

## 2. Add the connector to your assistant

The connector starts with:

```
npx -y @baytekdev/agile-analytics-mcp
```

**Claude Desktop** (Settings → Developer → Edit Config, which opens
`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "agile-analytics": {
      "command": "npx",
      "args": ["-y", "@baytekdev/agile-analytics-mcp"]
    }
  }
}
```

On macOS and Linux, add the token to the same entry:

```json
{
  "mcpServers": {
    "agile-analytics": {
      "command": "npx",
      "args": ["-y", "@baytekdev/agile-analytics-mcp"],
      "env": { "AZURE_DEVOPS_PAT": "<your token>" }
    }
  }
}
```

**Claude Code** (one command, stored for your user account only):

```
claude mcp add --scope user agile-analytics -- npx -y @baytekdev/agile-analytics-mcp
```

On macOS and Linux, pass the token into the entry with `-e`:

```
claude mcp add --scope user agile-analytics -e AZURE_DEVOPS_PAT=<your token> -- npx -y @baytekdev/agile-analytics-mcp
```

On Windows, Claude Code starts `npx` through `cmd /c`:

```
claude mcp add --scope user agile-analytics -- cmd /c npx -y @baytekdev/agile-analytics-mcp
```

**VS Code** (Command Palette → "MCP: Open User Configuration", which opens
your user `mcp.json`):

```json
{
  "servers": {
    "agile-analytics": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@baytekdev/agile-analytics-mcp"]
    }
  }
}
```

On macOS and Linux, add `"env": { "AZURE_DEVOPS_PAT": "<your token>" }` to
that entry. Do not put the token in a workspace `.vscode/mcp.json` or a
project `.mcp.json`: those files are often shared or committed.

## 3. Ask

Name the organisation (the `dev.azure.com/<org>` part), the project and the
team. For example: "Sprint summary for team Platform in project Contoso Web,
organisation contoso". Each answer is the tool's numbers; your assistant
turns them into words.

## Signing in with the Azure CLI instead

If you would rather not create a token, the connector can use the Azure CLI:
run `az login` once, and leave `AZURE_DEVOPS_PAT` unset. This works for
organisations connected to Microsoft Entra; Microsoft-account-only
organisations cannot use it. To pick the Entra tenant, add
`"--tenant", "<tenant-guid>"` to `args` (anything but a GUID is refused).

The connector looks for credentials in this order: `AZURE_DEVOPS_PAT`, then
the older `AZURE_DEVOPS_MCP_TEST_PAT` (still honoured, will be removed
later), then the Azure CLI.

## Tools

- `sprint_summary` (`sprintId`, default the last finished sprint): delivered
  items and points, cycle time, carryover, and commitment (committed at the
  end of day one, completed of committed, reliability, scope added and
  removed). Commitment is `null` until the end of day one has passed.
- `cycle_time` and `flow_efficiency` (`sprints` 3, 6 or 12, default 6): the
  last N **finished** sprints, never the running one.
- `wip`: the running sprint's items in progress right now.
- `aging`: open work in the current sprint, carried-over items from the
  previous sprint included, per item its stage, days in that stage and days
  in flow. `weekendBasis` (`business-days` or `calendar-days`) says which
  day basis the ages use: your saved Aging choice in the extension, else the
  same default the Aging view uses.
- `forecast` (`mode` `howMany` with `windowDays`, default 30; or `when`
  with `backlog`): the Monte Carlo view's default forecast.

Every answer also carries `scope`: `includesSubTeams` (the extension's
"Include the sub-teams' work" applies to this team and is on) and
`excludedTeamCount` (how many other teams' items this answer leaves out),
so it counts the same items as the extension does.

## What it reads, and what it never reads

- **Azure DevOps requests:** a fixed list of read requests (plus one work
  item query that selects ids only). Anything else is refused before it is
  sent.
- **Work item fields:** state, type, dates, sprint and area path,
  estimates, board column and the blocked flag. **Never** the title,
  assigned person, tags, description or comments.
- **Team settings:** the team's sprints and working days, and which teams
  sit under this one.
- **Agile Analytics settings:** the saved workflow mapping, the "Exclude
  from metrics" list, the Item Types mapping, the admin switch, your own
  WIP limit and weekend preferences, the organisation and project weekend
  defaults, and the sub-teams choice: named settings only. It never lists
  the settings, so it cannot see licence or other values.
- **Never** guesses a workflow mapping: no saved mapping means the answer
  `workflow-not-mapped`.

Answers contain numbers, dates, fixed words, ids and work item ids only,
never a title, a name or a comment.

## What leaves your computer

Requests to `dev.azure.com` and `extmgmt.dev.azure.com`, made with your own
access. **Nothing is sent to Baytek**: no licence check, no telemetry, no
usage counts. The answers go to your assistant, and what your assistant
does with them is between you and your assistant's provider.

## Turning it off for your organisation

An organisation admin can turn the connector off for everyone in the
extension: **Configuration → General, "Allow the local AI connector"**. It
is on unless turned off. The connector reads this setting before every
answer; when it is off, every tool answers `disabled-by-admin`, and when it
cannot be read the answer is `settings-unreadable`, never "on". The switch
is honoured by the connector on each person's computer; it does not change
anyone's Azure DevOps permissions.

## Errors

Errors are a bare code, for example `{"error":"workflow-not-mapped"}`:

- `azure-cli-sign-in-required`: no token was found and the Azure CLI is not
  signed in. Set `AZURE_DEVOPS_PAT` (see step 1) or run `az login`.
- `workflow-not-mapped`: save a workflow mapping for this project in the
  extension first.
- `disabled-by-admin`: an admin turned the connector off.
- `settings-unreadable`: the extension's settings could not be read (is the
  extension installed, and does the token have Extension Data: Read?).
- `ado-request-failed`: Azure DevOps refused or failed a request (check the
  token's scopes and expiry).
- `team-not-found`, `sprint-not-found`, `no-finished-sprints`,
  `invalid-input`, `request-not-allowed`, `internal-error`.

## What it keeps between calls

The project's team list, each team's areas and each team's working days are
kept in memory for 10 minutes; nothing is written to disk, and closing the
assistant forgets them. Everything else, including the admin switch and the
workflow mapping, is read fresh on every call. A team or area change in
Azure DevOps can take up to 10 minutes to show; restart the assistant to
see it at once.

## Not in this version

- **Aging zones and trend** (the p50/p85 bands and the per-sprint trend).

## About this repository

This repository holds the documentation for the connector. The package is
published on npm as
[`@baytekdev/agile-analytics-mcp`](https://www.npmjs.com/package/@baytekdev/agile-analytics-mcp)
and is part of [Agile Analytics for Azure DevOps](https://ado-analytics.baytekdev.com/),
available on the
[Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Baytek.agile-analytics).
The source code is not published here.

Questions and problems: open an issue in this repository, or write to
license@baytekdev.com.
