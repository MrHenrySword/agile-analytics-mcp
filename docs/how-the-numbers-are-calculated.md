# How the numbers are calculated

Agile Analytics for Azure DevOps works out every number from your own Azure
Boards data: work items, their state changes and their sprint (iteration)
history. This page gives the rule behind each of the main numbers in plain
words. The website has the full guides; each section links to one.

The local AI connector in this repository runs the extension's own metric
code, so your assistant gets the same numbers by the same rules.

## The one setting everything depends on: Workflow Mapping

Every team names its board states differently. Workflow Mapping says which of
your states mean new, queued, in progress, test, review and done. The
extension reads your board and maps it for you on first install, and an
admin can change it under Configuration.

Cycle time, flow efficiency, throughput, commitment reliability and carryover
all follow that mapping. If a number looks wrong, check the mapping first.

Guide: https://ado-analytics.baytekdev.com/docs/workflow-mapping/

## Flow

| Number | Rule |
| --- | --- |
| **Cycle time** | From the first move into an In Progress state to the first move into a Done state, in calendar days with decimals. Shown with p50, p70, p85 and p95 lines. Quote a percentile, not an average. |
| **Lead time** | From the day the item was created to the first move into a Done state, so it includes time waiting in the backlog. |
| **Throughput** | How many items finished. For a sprint, the sprint's items that are in a state mapped as Done. |
| **Work in progress (WIP)** | Items started but not finished. Compared with one WIP limit per team. Items in the Removed state do not count. |
| **Work item age** | How long each unfinished item has been in flow so far, shown against the team's own p50 and p85. |
| **Flow efficiency** | Touch time divided by cycle time. Time in states mapped as In Progress, Test or Review counts as active; time in queued or waiting states counts as wait. A state whose name signals a queue (for example "ready for", blocked, waiting, on hold) counts as wait even when it is mapped as active. |

A running sprint is never part of an average, a trend or a forecast. Those
use finished sprints only.

Guide: https://ado-analytics.baytekdev.com/blog/azure-devops-metrics/

## Sprints

**Planned Items (the commitment).** What the sprint holds at the end of its
first day, measured in UTC. A planning meeting held on the first day still
counts. A work item also counts as planned when its own Azure DevOps history
shows it was already in the sprint at the end of day 1, for example a story
planned the day before the sprint starts.

**Scope Change.** Work added to the sprint after day 1, and work removed from
it after day 1. An item that goes in and comes straight out again is neutral.

**Commitment reliability.** The share of planned items that were done by the
end of the sprint. Work finished up to 24 hours after the last sprint day
still counts as done, so a late close in another time zone is not held
against the team. Removed items count on neither side.

**Carryover.** An item is carried over when all three are true:

1. It sat in the sprint at the end of at least one sprint day. An item pulled
   in by mistake and taken out again the same day never counts.
2. It had not reached a state your Workflow Mapping calls Done by the time
   the sprint ended, or by the time it left the sprint.
3. It was still in the sprint when the sprint ended, or it was moved straight
   into the next sprint. Moving it anywhere else, such as back to the
   backlog, makes it removed scope instead.

The carryover percentage is carryover items divided by every item that was in
the sprint at the end of at least one day. It can never go over 100%.

Guides: https://ado-analytics.baytekdev.com/docs/carryover/ and
https://ado-analytics.baytekdev.com/blog/how-to-see-scope-change-during-a-sprint-azure-devops/

## Forecasts

Monte Carlo forecasting resamples the throughput of your finished sprints in
10,000 trials. It answers "how many items by this date?" or "when will this
many items be finished?" as a range, at 50%, 70%, 85% and 95% confidence,
not as a single date.

## Where to read more

- Documentation: https://ado-analytics.baytekdev.com/docs/
- Every view the extension ships with: https://ado-analytics.baytekdev.com/features/
- Release notes: https://ado-analytics.baytekdev.com/release-notes/

If a number on your screen does not match the rule on this page, tell us:
https://ado-analytics.baytekdev.com/support/ or support@baytekdev.com.
