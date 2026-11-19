# Agile Analytics local AI connector

Ask Claude, or another assistant that supports the Model Context Protocol
(MCP), about your team's Agile Analytics numbers: sprint summary, cycle
time, flow efficiency, work in progress, aging and forecast. The connector
runs on your own computer, reads Azure DevOps with your own access, and
answers with the same numbers the Agile Analytics extension shows, because it
runs the extension's own metric code.

It works during the trial and on every paid plan, for the people who hold a
seat (on Team and Business, the assigned people; on a trial, on Premium and
on the older plans, everyone in the organisation). It answers only when the
extension is installed in your organisation, the organisation is on its
trial or has a valid licence, and a workflow mapping is saved for the
project. Nothing is sent to Baytek.

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

- **Claude Desktop, on every system:** it does not pass your environment
  variables to connectors, so put the token in the server entry's **`"env"`
  block** as `AZURE_DEVOPS_PAT` (shown in the example below), and keep that
  config file readable by your user account only.
- **Windows, other assistants (Claude Code, VS Code):** set it as a **user
  environment variable named `AZURE_DEVOPS_PAT`** (Start → "Edit environment
  variables for your account" → New), which keeps it out of config files.
  Restart your assistant afterwards so it sees the new variable.
- **macOS and Linux:** desktop assistants do not read your shell profile, so
  put the token in the server entry's **`"env"` block** as
  `AZURE_DEVOPS_PAT` (shown in the examples below), and keep that config file
  readable by your user account only (`chmod 600 <file>`).

If the connector answers `azure-cli-sign-in-required` (`settings-unreadable`
before 6.76.1) although the variable is set, your assistant is not passing it
to the connector: use the `"env"` block.

The token stays in the connector's memory. It is never printed, logged or
written anywhere.

## 2. Add the connector to your assistant

The connector starts with:

```
npx -y @baytekdev/agile-analytics-mcp
```

**Claude Desktop** (Settings → Desktop app → Extensions → Edit config, under
Local MCP servers; in older versions Settings → Developer → Edit Config. It
opens `claude_desktop_config.json`: add the `mcpServers` section, keeping
anything already in the file, save, then quit Claude Desktop completely, from
the tray icon too, and open it again):

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

Claude Desktop needs the token in the entry on Windows too: it does not pass
your environment variables to the connector. Leave the `"env"` line out only
if you sign in with the Azure CLI.

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

**First time?** Ask "check the Agile Analytics connector status for
organisation contoso" (the `status` tool, below): it says what is still
missing, one check at a time.

### Remembering your organisation, project and team

If you mostly ask about one team, set these once in the server entry's
`"env"` block (next to the token) and leave them out of your questions:

```json
"env": {
  "AZURE_DEVOPS_PAT": "<your token>",
  "AGILE_ANALYTICS_ORG": "contoso",
  "AGILE_ANALYTICS_PROJECT": "Contoso Web",
  "AGILE_ANALYTICS_TEAM": "Platform"
}
```

Each one is optional, and a name you do give in a question still wins. The
connector reads them when it starts, so restart your assistant after a
change. A value that is not a valid name answers `invalid-input`. Leaving
them unset (or empty) changes nothing.

## Signing in with the Azure CLI instead

If you would rather not create a token, the connector can use the Azure CLI:
run `az login` once, and leave `AZURE_DEVOPS_PAT` unset. This works for
organisations connected to Microsoft Entra, with an account that is a member
of the organisation; Microsoft-account-only organisations cannot use it and
need a token. To pick the Entra tenant, add
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
  last N **finished** sprints, never the running one. `from` and `to` are
  the dates those sprints cover (the earliest start and the latest end), so
  your assistant can say the period instead of naming sprints by id.
- `wip`: the running sprint's items in progress right now.
- `aging`: open work in the current sprint, carried-over items from the
  previous sprint included, per item its stage, days in that stage and days
  in flow. `weekendBasis` (`business-days` or `calendar-days`) says which
  day basis the ages use: your saved Aging choice in the extension, else the
  same default the Aging view uses.
- `forecast` (`mode` `howMany` with `windowDays`, default 30; or `when`
  with `backlog`): the Monte Carlo view's default forecast.
- `status` (`org`; `project` and `team` optional): what the connector still
  needs, without stopping at the first problem. It runs the same checks
  every tool runs, in the same order, and gives each one a fixed word:
  `signIn` (`ok` / `sign-in-required`), `settings` (`ok` /
  `settings-unreadable`), `adminSwitch` (`on` / `off`), `licence` (`trial` /
  `licensed` / `licence-inactive`), `seat` (`ok` / `no-seat`),
  `workflowMapping` (`saved` / `workflow-not-mapped`, checked only with a
  project and team) and `plan` (`open` / `assigned-seats`). A check after
  one that failed is `not-checked`; a check that could not run says why with
  an error code (for example `team-not-found`). It also gives the
  connector's version and the organisation it looked at, and nothing else:
  no names, no e-mail addresses, nothing about the licence but those words.

Every answer also carries `scope`: `includesSubTeams` (the extension's
"Include the sub-teams' work" applies to this team and is on) and
`excludedTeamCount` (how many other teams' items this answer leaves out),
so it counts the same items as the extension does.

On Premium (a plan open to the whole organisation, trials and the older
plans included), every answer of the six tools also carries
`openInAgileAnalytics`: a link that opens the matching view of the
extension for that team, built only from the organisation, project and
team you gave and fixed words, for example
`https://dev.azure.com/contoso/Contoso%20Web/_apps/hub/Baytek.agile-analytics.agile-analytics-hub#view=cycle-time&team=Platform`.
On Team and Business the answers are the same without the link. When the
plan cannot be read, the link is added.

Every tool is marked read-only for your assistant (it changes nothing and
reads Azure DevOps), has a short title, and publishes the shape of its answer
(an output schema). Each answer comes twice in one reply: as JSON text and as
structured content of that shape.

## Ready-made commands (Premium)

Three commands your assistant can offer in its prompt or slash-command
menu: **Sprint review** (`sprint_review`), **Stand-up** (`stand_up`) and
**Forecast** (`forecast`). Each takes an optional organisation, project and
team (the remembered ones by default) and gives your assistant a short task:
which tools to call and how to present the answer (numbers and their dates
first, sample sizes, work item ids to open, never invented titles or names,
and a word of caution when a number rests on few sprints). With no
organisation known, the task starts by asking you for it.

The commands are part of the **Premium** plan, that is a plan open to the
whole organisation (trials and the older plans count too). On Team and
Business the command answers that it is part of Premium; the six questions
and the `status` check still work on every plan for everyone who holds a
seat: ask them in your own words. When the plan cannot be read, the
command is offered anyway.

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
  the settings, so it cannot see other values.
- **Your seat:** the plan's access rule the extension last stored and, when
  it limits seats, the organisation's access list (who is assigned, who is
  an extension admin, and who asked for access) and your own Azure DevOps
  identity, to see whether you hold a seat. They are used on your computer
  for a yes or no. None of it is put in an answer.
- **Licence state:** whether your organisation is on its trial or has a
  valid licence, from what the extension last stored in Azure DevOps: the
  plan, when the licence was last validated, the trial dates and when the
  extension last reached the licence server. **Never** the activation token,
  the licence's id or the organisation name on it.
- **Never** guesses a workflow mapping: no saved mapping means the answer
  `workflow-not-mapped`.

Answers contain numbers, dates, fixed words, ids and work item ids only,
never a title, a name or a comment. The exceptions are what you typed
yourself: the link back to the extension (organisation, project and team)
and, in the `status` answer, the organisation name and the connector's
version.

## What leaves your computer

Requests to `dev.azure.com` and `extmgmt.dev.azure.com`, made with your own
access. **Nothing is sent to Baytek**: no telemetry, no usage counts, and
the licence is checked on your computer from what is stored in your Azure
DevOps organisation, not by asking Baytek. The answers go to your assistant, and what your assistant
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

- `azure-cli-sign-in-required`: Azure DevOps did not accept your sign-in, or
  there is none. Check that `AZURE_DEVOPS_PAT` is set, has not expired and has
  the three scopes (see step 1). On the Azure CLI route, run `az login` again;
  if the organisation is not connected to Microsoft Entra, or your account is
  not a member of it, the Azure CLI route cannot work there, so use a token.
- `workflow-not-mapped`: save a workflow mapping for this project in the
  extension first.
- `disabled-by-admin`: an admin turned the connector off.
- `no-seat`: your organisation's plan gives Agile Analytics to assigned
  people only (Team and Business), and you are not one of them. Ask an
  extension admin to assign you a seat in Configuration → Access Control. On
  a trial, on Premium and on the older plans everyone in the organisation can
  use the connector. Two things to check first: open Agile Analytics in Azure
  DevOps once and ask again (after a plan change the connector sees the new
  plan only once the extension has stored it), and make sure the token or
  the Azure CLI sign-in belongs to the same account you use in the browser.
- `licence-inactive`: the organisation's trial has ended and it has no valid
  licence, or its licence has not been confirmed for 30 days. That happens
  when nobody has opened the Agile Analytics hub page for 30 days (a
  dashboard widget does not count), or when the stored licence needs
  attention. Open Agile Analytics in Azure DevOps: a valid licence is
  confirmed again when the hub opens, and the trial-ended screen shows how to
  add a licence key. If the answer stays the same, an admin opens
  Configuration → License and follows the message there, or writes to
  license@baytekdev.com.
- `settings-unreadable`: the extension's settings could not be read (is the
  extension installed, and does the token have Extension Data: Read?).
- `throttled`: Azure DevOps is limiting how fast your account can read, and
  was still busy after the connector asked again. Wait a minute and ask
  again.
- `timed-out`: Azure DevOps took too long to answer. Ask again, or ask for
  fewer sprints.
- `forbidden`: your Azure DevOps account cannot read this project or team.
  Ask a project admin for access, or check the token's scopes.
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

## Developing

This package lives in `mcp/` of the Agile Analytics extension repository and
imports the extension's metric code by path (`mcp/src/core/index.ts`), never
a copy, so its numbers cannot drift. **Rule: no partial numbers.** Where the
connector cannot read what the extension reads, the number is left out, not
approximated.

- **Build:** `npx webpack --config mcp/webpack.config.js` writes
  `mcp/dist/server.js` (the bin) and `mcp/dist/cli.js` (hand testing). Every
  dependency is bundled, so the package has none at run time.
- **Gate:** `npm run check` builds the bundle, fails if it contains the
  extension SDK, the licence server client or telemetry code, a Baytek
  address, or `fetch(`
  outside `mcp/src/data/adoClient.ts`, then runs `npm pack --dry-run` in
  `mcp/` and fails unless the package holds exactly `package.json`,
  `README.md`, `LICENSE` and `dist/*.js`, under 2 MB.
- **Version:** `node scripts/bump-version.mjs --target all` bumps
  `mcp/package.json` with the extension manifests.
- **Hand testing:** `node mcp/dist/cli.js sprint_summary --org <org>
--project "<project>" --team "<team>"` prints one tool's JSON;
  `node mcp/test/live/liveClient.mjs --org <org> --project "<project>"
--team "<team>"` starts the server and calls every tool (skipped unless
  `AZURE_DEVOPS_PAT` is set). `--extension agile-analytics-v2` targets the
  private V2 test listing.
- **Tests:** `npx vitest run mcp/test` (part of the full suite; needs
  `node scripts/setup-test-fixtures.mjs`). The parity tests prove each tool
  equals the extension on the repo's scenario captures and hand-built
  histories.
- **The admin switch** is the org document `agile-analytics:mcp-settings:v1`
  (`{"disabled": true}` = off, no document = on), written only by
  `McpConnectorSwitch` in the extension and only read here.

How each tool matches the extension:

- `sprint_summary`: delivered and cycle time from `buildSprintReport`;
  commitment and carryover from Sprint Summary's own
  `buildVisibleSprintMetricsFromBaseline` on the item pool and Day-1 ASOF
  baseline that `getSprintScopeWorkItems` builds, with the same WIQL ladder
  (`mcp/src/data/sprintScope.ts`).
- `cycle_time`: the Cycle Time view's `loadPointsForTeam` sample and
  `findCycleDays`, nearest-rank `percentile`. The view also offers a
  1-sprint window; the connector keeps 3, 6 and 12.
- `flow_efficiency`: the Flow Efficiency view's `aggregateAcrossMappings`.
- `wip`: `isInProgressAtInstant` on the running sprint, as `getWip` does.
- `aging`: the Aging view's `agingViewsWithCarryover`; the weekend basis
  follows `useAgingWeekendPreference.ts` (`mcp/src/settings/agingWeekends.ts`).
- `forecast`: the Monte Carlo view's default path (12 sprints of
  throughput, the running sprint dropped, the seeded simulation).
- Which items count: the extension's `subTeamScopeRules.ts` and
  `parentScopeFilter.ts`, applied by `mcp/src/data/subTeamScope.ts`. Two
  limits: the connector cannot see Baytek's backend kill switch for the
  sub-teams feature, so it assumes on; and where the extension carries on
  after a failed team read, the connector answers `ado-request-failed`.
- The read cache: `mcp/src/data/readCache.ts` (10 minutes, at most 500
  answers, successful reads only).
