---
name: campaign-setup
description: Use when the user wants to text many contacts at once on PitchPrfct, such as launching, scheduling, pausing or resuming an SMS or workflow campaign, building or changing a workflow, choosing its audience, or enrolling contacts in a workflow. Covers readiness checks, audience building, drip pacing, credit estimates and safe launch.
---

# Campaign setup

A campaign sends to a whole audience and costs credits per text. It is the right
tool for anything beyond a handful of one-off messages. Never loop single sends
instead. Tool names are the PitchPrfct MCP tools, possibly shown with a server
prefix. Every message rule in the `compliant-sending` skill applies to each text
a campaign sends. To have an AI agent answer the replies a campaign or
workflow gets, use the `agent-builder` skill.

## 1. Check readiness first (blocking)

Before you design anything, check that the account can send. If something is
missing, stop and say what it is:

- An approved 10DLC brand and campaign with the sending numbers attached
  (`ten_dlc_brands_list`, `ten_dlc_campaigns_list_by_brand`). If not, use the
  `ten-dlc-registration` skill.
- At least one enabled number (`phone_numbers_list`). An empty list means it
  cannot send.
- An active subscription (`billing_api_get_status`).
- Credits (`credits_balance_api_get_balance`). Starting or resuming a campaign
  needs at least 50 SMS credits. Beyond that, budget about one credit per
  160-character segment per contact (an MMS costs 3).

## 2. Pick the campaign type

- **SMS campaign** (`type: "sms"`): one message to everyone. Needs `message` and,
  for a single sender, `phoneNumber` (E.164, an enabled number).
- **Workflow campaign** (`type: "workflow"`): enrolls the audience in a
  multi-step workflow (`workflowId`). Find it with `workflows_list` and check its
  steps with `workflows_get`. For a few contacts, `workflows_enroll` or
  `workflows_bulk_enroll` enroll them directly without a campaign.

### Building or changing a workflow

- `workflows_create` makes a workflow with its trigger and steps. It is a draft
  unless you send `status: "active"`.
- Change one step at a time with `workflows_insert_step`,
  `workflows_update_step`, `workflows_remove_step` and `workflows_reorder_steps`.
  Other steps keep their `nodeId`.
- To turn a draft on, call `workflows_update` with `status: "active"`. A
  published workflow can start texting contacts at once, so confirm with the
  user first.
- `workflows_pause` stops a published workflow. `workflows_resume` starts a
  paused one again. Both finish in the background.

### First-message wording

The first text a contact gets (the first text step of a workflow, or the
message of an SMS campaign) must name the business and include opt-out wording
("Reply STOP to opt out"). Personalise with merge fields such as `{firstName}`.
`templates_list` holds the account's approved copy. Use `compliance_check` to
test the text before you save it.

The server checks the first message when it goes live for the first time (a
draft workflow is published, a campaign that never started is started or
scheduled) and when the first message of a published or paused one changes. It
refuses with MESSAGE_NOT_COMPLIANT, listing what to fix. Fix the text and try
again. A resume is not checked. If `compliance_check` answers 404, the server
check is not on for this account yet: check the text yourself before you save.

## 3. Build the audience

- Find contacts with `contacts_list`. It filters by `listId`, `tags`, `search`
  and created date, and pages with `page` and `take`. Lists come from
  `contact_lists_list` and tags from `tags_list`.
- Leave out opted-out contacts. `contact_opt_outs_list` shows them.
- `campaigns_add_contacts` takes up to 10,000 contact UUIDs per call. If any
  contact opted out in the last 30 days, the **whole request** is refused with
  400 HIGH_RISK_CONTACTS. Remove those contacts and call again. Don't force them in.
  The response reports how many were added and how many were filtered (already
  enrolled, opted out…).
- Watch for overlap. A contact already in an active workflow or campaign may get
  two messages in one day. Check `campaigns_list` and `workflows_list_enrollments`
  and ask the user how to handle duplicates.

## 4. Create, pace and schedule

`campaigns_create` takes `name`, `type`, and `message` + `phoneNumber` (SMS) or
`workflowId` (workflow). Optional:

- `scheduledStart` (ISO 8601). Schedule the start inside **8:00–21:00 in the
  recipients' local time**. With a nationwide audience, start when every time zone
  is inside that window.
- Drip pacing: `dripBatchSize` (default 10) texts every `dripInterval`
  `dripIntervalUnit` (minutes, hours or days). Work out how long the run takes:
  audience ÷ batch size × interval. Keep it inside the window, or the drip will
  still be texting in the evening.
- New brands have low daily carrier caps. Texts above the cap are held and
  delivered the next day. Pace large sends over several days rather than
  bursting.

Contacts are added after creation (step 3). Use `campaigns_update` to change a
draft. Once a campaign is running, only its message can change, and a scheduled
campaign can't be edited in the minute before it starts.

## 5. Confirm, then launch

Before `campaigns_start`, show the user: how many people it will reach, the exact
first message, the sender, the start time and time zone, the drip pace with its
estimated finish time, and the estimated credit cost. Get an explicit yes.

`campaigns_start` can refuse with:

- 402: not enough credits.
- 403: no subscription, onboarding not finished, or the account is paused. Tell
  the user; don't retry.
- 422 MESSAGE_NOT_COMPLIANT: the first message fails the compliance check. The
  error lists each issue and how to fix it. Fix the text with the user, then
  start again. 422 BRAND_REQUIRED means the account has no 10DLC brand yet.
- 503 COMPLIANCE_UNAVAILABLE: the check could not run and nothing started.
  Retry in a few minutes.

## 6. Run it

- `campaigns_get` shows progress. `campaigns_list_contacts` with `status`
  (pending, processed, succeeded, failed) shows each recipient.
- `campaigns_pause` stops it right away (for example near 21:00, or after a
  spike in opt-outs). `campaigns_resume` continues it, and needs the same
  credit and subscription checks as starting.
- `campaigns_delete` is permanent and removes the campaign's data. Confirm first.

## Never

- Start or resume a campaign, or publish or resume a workflow, without the
  user's explicit confirmation of the audience size, text and cost.
- Re-add contacts the platform filtered or refused as high risk.
- Schedule sends outside 8:00–21:00 in the recipients' local time.
