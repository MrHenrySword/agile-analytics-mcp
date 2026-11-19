# What data leaves your Azure DevOps organisation

A short, plain account for security and privacy reviewers. The Privacy Policy
lists every item in full: https://ado-analytics.baytekdev.com/privacy/

## The short version

- Every analytics view runs **in your browser**, against the Azure DevOps
  REST API, using your existing Azure DevOps session.
- Sprint contents, work item titles, pull requests, code, comments and
  attachments: **none of it is sent to a Baytek server.**
- The extension asks for **four read-only permissions** and cannot change
  your work items.
- The local AI connector in this repository runs on your own computer and
  sends **nothing to Baytek**.

## Permissions the extension asks for

`vso.project`, `vso.work`, `vso.graph` and `vso.build`. All four are read
scopes. There is no `vso.code`, no `vso.analytics` and no write scope, so the
extension cannot modify work items, repositories, pipelines or permissions.

It reads Azure DevOps through the signed-in person's own session. You do not
create or hand over a personal access token to use the extension:
https://ado-analytics.baytekdev.com/solutions/azure-devops-analytics-without-pat/

## What stays in your organisation

Azure DevOps work item content, and any AI adoption credentials you enter.
Baytek never sees either.

## What Baytek's backend receives

- **Licence records:** organisation name, plan, expiry, activation tokens.
- **An install and heartbeat ping:** organisation name, extension version,
  event type, timestamp.
- **Pseudonymous usage events:** which view was opened, which feature was
  used, configuration completion, trial-day milestones. They are linked to
  your organisation and your Azure DevOps user GUID, never your name or
  email, and are used to count active users and enforce seat limits. An
  extension admin can turn them off for the whole organisation under
  Configuration, Privacy.
- **On Team and Business plans, who holds a seat:** as opaque Azure DevOps
  identity descriptors and role, never names or email addresses.
- **One administrator sign-in email:** the Azure DevOps sign-in email of the
  first organisation administrator to open the hub is recorded once per
  organisation, the first time they open it, and used for trial reminders and
  service notices, each with a one-click opt-out link. You can ask us to
  delete it at any time.
- **A short-lived access token, for one check:** to confirm which
  organisation a request comes from, the extension sends a short-lived Azure
  DevOps access token that our server uses for one check and then discards.
  Our licence server also records the IP address and browser of licence
  requests, for security and rate limiting.
- **Industry Benchmarks, only if you opt in:** that feature's aggregate
  metric data.

None of this is work item content.

## AGAN AI, the built-in assistant

If you send AGAN AI a question, or ask it for a stakeholder summary, these
reach Baytek and its AI provider, Anthropic:

- the question (a summary has none) and the earlier messages of that
  conversation;
- a short list of counts and codes about your setup (including Setup advisor
  numbers such as your WIP limit), with item types and states sent by
  position, not by name;
- on the views that show them, the numbers on that view, with the same
  numbers for the previous sprint or period where the view holds them.

Never work item titles, IDs, state names or people's names. No transcript is
kept; Baytek keeps only a monthly total of AI usage per organisation. An
extension admin can turn AI answers off under Configuration, Access Control,
and each person can switch AGAN AI on or off in the account menu.

## Optional integrations

When you connect GitHub Copilot for AI Metrics, or set up webhook
notifications, requests go from your browser directly to that source
(`api.github.com`) or to your webhook address (Microsoft Teams or Slack),
never through Baytek.

## The local AI connector

It runs on your own computer, reads Azure DevOps with your own access, and
answers your assistant from there. It makes no request to a Baytek server.
What your assistant does with an answer follows your assistant's own terms.

Guide and roll-out notes for security teams:
https://ado-analytics.baytekdev.com/docs/local-ai-connector/

## More

- Security overview: https://ado-analytics.baytekdev.com/security/
- Privacy Policy: https://ado-analytics.baytekdev.com/privacy/
- Report a vulnerability: see [SECURITY.md](../SECURITY.md)
- Questions: support@baytekdev.com
