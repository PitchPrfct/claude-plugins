---
name: ten-dlc-registration
description: Use when a PitchPrfct account needs to register or fix its 10DLC setup (brand, 10DLC campaign, phone numbers attached), or when texts are blocked, filtered or held because registration is missing, pending or declined. Walks brand, then campaign, then attach phone numbers, and explains statuses and vetting.
---

# 10DLC registration

US carriers deliver business texts from a local (10-digit) number only when the
sender is registered. Registration happens in three steps, always in this order:

1. **Brand**: the legal business (or a sole proprietor).
2. **10DLC campaign**: what the brand will text about. It is reviewed, then
   registered with the carrier.
3. **Attach phone numbers** to the approved campaign.

A number can send only when all three are done. Tool names are the PitchPrfct
MCP tools, possibly shown with a server prefix.

## Start by reading what exists

- `ten_dlc_brands_list` shows the account's brands and their status.
  `ten_dlc_brands_get_by_id` reads one brand by its provider brand id.
- `ten_dlc_campaigns_list_by_brand` lists a brand's campaigns, refreshed from the
  carrier.
- `phone_numbers_list` lists enabled numbers.
- `ten_dlc_reference_get_provider_status` shows which providers the account has
  brands on, and whether an upgrade is recommended.

Report what is done and what is missing before you change anything. Statuses
change over hours or days. Re-read them with these tools and never assume a step
went through.

## Two ways to register

- **Guided auto-flow (recommended for new businesses without a compliant
  website):** `ten_dlc_generations_create` with the business details starts a
  run. `ten_dlc_generations_set_domain` secures a domain and builds the
  compliance website. That step can cost money, so get the user's explicit
  approval first. `ten_dlc_generations_get_for_user` follows progress,
  `ten_dlc_generations_retry` resumes a failed run, and
  `ten_dlc_generations_remove` deletes one before a domain is bought.
- **Direct registration** (the business already has a compliant website): the
  steps below.

## Step 1: brand

- Get the allowed values first:
  `ten_dlc_reference_get_enums` with `entityTypes`, `verticals` and
  `brandRelationships`.
- `ten_dlc_brands_create`. Use exactly the legal name and EIN as they appear on
  IRS records. A mismatch is the most common reason a brand fails. Required
  fields depend on the entity type:
  - PRIVATE_PROFIT, NON_PROFIT, GOVERNMENT: companyName, ein, brandRelationship,
    vertical.
  - PUBLIC_PROFIT: also stockSymbol and stockExchange.
  - SOLE_PROPRIETOR: firstName, lastName, mobilePhone (no EIN).
- If the EIN or legal name is already registered to another account, creation
  is refused. Don't retry with altered details; tell the user.
- An unverified brand can be corrected and sent again with
  `ten_dlc_brands_resubmit`. Only the display name (DBA) can be renamed later,
  with `ten_dlc_brands_update_dba`. The EIN and legal name cannot.
- **Vetting** raises the brand's daily carrier cap. `ten_dlc_brands_request_vetting`
  (STANDARD or ENHANCED) is a paid order, so confirm the cost with the user first.
  `ten_dlc_brands_get_vetting_history` shows past scores.
  `ten_dlc_brands_get_capabilities` shows per-carrier limits.

## Step 2: 10DLC campaign

- `ten_dlc_reference_get_enums` with `usecases` picks the use case. Choose the one
  that matches what the business will actually send (marketing, customer care,
  account notifications…). Traffic outside the registered use case gets filtered.
- `ten_dlc_campaigns_create` needs `brandId` (the provider brand id), `usecase`,
  `subUsecases`, `description` and at least one sample message. Strong
  submissions also include:
  - a `messageFlow` that says exactly how people opt in (web form with a consent
    checkbox, keyword, in person), with a link to it when one exists;
  - samples that name the brand and include "Reply STOP to opt out";
  - opt-out, help and opt-in keywords and replies;
  - `privacyPolicyLink` and `termsAndConditionsLink` on the brand's website. The
    privacy policy must say that mobile numbers are not shared for marketing.
- A new campaign starts **in review**: an internal review checks it before it
  reaches the carrier. `ten_dlc_campaigns_edit` changes it while it is in review.
  A declined campaign can be fixed and sent again with `ten_dlc_campaigns_resubmit`.
  `ten_dlc_campaigns_remove` deletes one. Deleting is irreversible, so confirm first.
- `ten_dlc_campaigns_get` reads one campaign by its internal id.

## Step 3: attach phone numbers

- Once the campaign is approved, `ten_dlc_campaigns_attach_phone` with the
  provider campaign id and the internal ids of numbers from `phone_numbers_list`.
- Attach every number that will send for this brand. A number that isn't attached
  will see its texts blocked or filtered.

## After registration

- Re-check with `ten_dlc_campaigns_list_by_brand` until the numbers show as
  attached, then hand over to the `compliant-sending` skill.
- If the carrier holds texts above the daily cap, they are delivered the next day.
  Vetting is how to raise the cap. Never promise a specific cap or score. Read it
  from the tools.

## Never

- Guess or "clean up" a legal name or EIN, or invent opt-in evidence.
- Order vetting or a domain without the user's explicit approval.
- Tell the user a brand or campaign is approved before a tool shows it.
- Name the messaging provider. Say "the carrier".
