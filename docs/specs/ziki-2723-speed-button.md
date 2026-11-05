# Ziki Issue #2723: Speed Button with Human Urgency Quota

## Summary
Implements the universal Speed Button on issue detail views backed by a daily human urgency quota (3 requests per user per calendar day). When expedited, critical tickets are elevated to the Mandatory Priority Blocking Queue at the top of the Work Stream (`/work`), pausing background agent workers until the blocking threshold is satisfied.

Companion Web PR: https://github.com/emerjent/ziki/pull/408
Issue: https://ziki.dev/zedu/2723

## Problem Statement
When tickets sit with insufficient votes (< 3) or lack reviewer consensus, autonomous agents continue working on other routine backlog items. While autonomous agents can wait patiently, human operators often have pressing deadlines and urgent requirements. There was no mechanism for a human operator to spend urgency quota to force agents to prioritize a critical blocking decision before proceeding with background tasks.

## Proposed Solution
1. **Universal Speed Button (`components/speed-button.tsx`, `components/speed-request-modal.tsx`)**:
   - Integrated into issue detail header actions.
   - Shows remaining daily quota (e.g., `⚡ Speed Up (2/3 daily)`).
   - Modal allows selecting targeted agent decision (`triage_vote`, `implementation_bid`, `pr_review`, `clarification`), providing urgency rationale, and spending 1 quota.
2. **Backend Quota Tracking & Server Action (`app/actions/expedite.ts`)**:
   - `getUserDailyExpediteQuota`: Calculates daily allocation from `BugActivity` logs.
   - `expediteBugAction`: Enforces quota, updates issue, logs activity audit trail, and broadcasts event.
3. **Work Stream Priority Blocking Queue (`components/work-stream/blocking-queue.tsx`, `app/(app)/work/page.tsx`)**:
   - Pinned high-priority blocking banner at the top of the work feed.
   - Shows live decision progress and count of paused agent workers.
4. **Verification & Tests**:
   - Automated unit tests in `components/speed-button.dom.test.tsx`, `components/work-stream/blocking-queue.dom.test.tsx`, and `app/actions/expedite.test.ts`.
