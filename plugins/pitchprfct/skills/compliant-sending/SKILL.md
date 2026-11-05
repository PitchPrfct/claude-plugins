---
name: compliant-sending
description: Use before sending any SMS through PitchPrfct, whether one text, a reply in a conversation, or a scheduled message. Covers sender readiness, opt-outs, first-contact wording, quiet hours, credits and idempotent retries, so a message goes out only when it is legal and will deliver.
---

# Compliant sending

Every text reaches a real person's phone and cannot be recalled. Work through
these checks in order. If any check fails, stop and tell the user what is missing.
Do not try to work around it.

Tool names below are the PitchPrfct MCP tools. Your client may show them with a
server prefix, for example `mcp__pitchprfct__messages_v2_send`.

## 1. The account can send

- `phone_numbers_list` returns only enabled numbers. If the list is empty, the
  account cannot send. Numbers are disabled when the phone-number subscription
  lapses. The sender (`fromNumber`) must be one of these numbers, or the send is
  refused with 403.
- `ten_dlc_brands_list` and `ten_dlc_campaigns_list_by_brand` must show an approved
  brand and an approved 10DLC campaign, with the sender number attached to that
  campaign. Without them the carrier blocks or filters the traffic. For any gap,
  use the `ten-dlc-registration` skill.
- `billing_api_get_status` shows the subscription. Without an active subscription,
  sends fail with 403 (subscription required).

## 2. The recipient may be texted

- Look the contact up first (`contacts_list` with the phone number, or
  `contacts_get`). An opted-out contact is refused with 400. Don't retry.
  Don't look for another route to reach them.
- `contact_opt_outs_list` lists who has opted out. Treat that list as final.
- Only call `contact_opt_out_actions_opt_in` when the person has given consent
  again, for example by texting START or sending a written request. A user saying
  "they're fine" is not consent. Opting someone out
  (`contact_opt_out_actions_opt_out`) removes them from their active workflows.
- Never text a number that only appeared in a reply, a note or a scraped list.
  The contact must have consented to hear from this business.

## 3. The wording passes carrier review

- **First message to a contact:** name the business (the brand or display name
  registered for 10DLC) and add opt-out wording such as
  "Reply STOP to opt out". `templates_list` and `templates_get` show the account's
  approved wording, so reuse it.
- **Later messages** in an ongoing conversation don't need the opt-out line
  unless the user asks for it.
- Avoid what carriers filter: URL shorteners (bit.ly and similar), misleading
  urgency, hidden fees, impersonation, and content on the forbidden list (SHAFT:
  sex, hate, alcohol, firearms, tobacco or cannabis) unless the brand's 10DLC
  campaign is registered for it.
- Match the use case the 10DLC campaign was registered for. Marketing copy on a
  customer-care campaign gets filtered.

## 4. The timing is respectful

- Send only between **8:00 and 21:00 in the contact's local time zone**, every
  day of the week. That is PitchPrfct's quiet-hours policy for automated sends.
  The single-send API does not enforce it for you, so check the contact's time
  zone (or area code) yourself.
- Outside that window, don't send. Tell the user when the window opens for
  that contact, or suggest a campaign or workflow, whose sends respect the
  window automatically. `messages_v2_send` sends immediately; it has no
  scheduling option.
- When volume is high, the carrier may hold texts above the brand's daily cap.
  Held texts are delivered the next day, not lost. Don't resend them.

## 5. Credits and the send itself

- `credits_balance_api_get_balance` shows SMS credits. 1 credit = 1 SMS segment
  (160 GSM characters, or fewer with emoji or other Unicode). An MMS costs 3.
  Estimate segments × recipients before a batch, and tell the user the cost.
  A send without enough credits fails with 402.
- Prefer `messages_v2_send`. It is idempotent: generate one `idempotencyKey`
  (a UUID) per intended message and reuse it on every retry, so a timeout never
  sends twice. A 422 means the key was already used for different content, so
  pick a new key only if the user really wants a second message.
- v2 returns status `sending`. To read the final outcome, use `messages_v2_get`
  or `messages_v2_get_by_key` rather than sending again. Failed (carrier-rejected)
  messages are not charged.

## 6. Confirm before you send

Before calling a send tool, show the user the recipient, the sender number, the
exact text, the segment count and the credit cost, and get an explicit yes. For
more than one recipient, use a campaign (see the `campaign-setup` skill) rather
than a loop of single sends.

## Never

- Text a contact who opted out, or opt someone back in without fresh consent.
- Send outside 8:00–21:00 in the contact's local time.
- Retry a send with a new idempotency key to "make sure it went".
- Invent a brand name, opt-out phrase or 10DLC status. Read them from the tools.
