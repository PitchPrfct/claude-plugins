---
name: agent-builder
description: Use when the user wants an AI agent on PitchPrfct that answers contacts by SMS on its own, such as building, editing, testing or publishing an appointment-booking agent, attaching it to phone numbers, workflows or campaigns, or following and taking over the conversations it handles. Covers readiness checks, the setup interview, writing the persona, instructions and qualifying questions, the proposal shown before anything is created, test conversations, a confirmed go-live and what to watch afterwards.
---

# Agent builder

An AI agent texts real contacts without a person typing. Once it is published
and has a channel (a phone number, a workflow or an SMS campaign), it answers
the contacts who text that number or reply to that workflow or campaign: it
asks its qualifying questions and books a time on the business calendar. Only
appointment-booking agents can be built today. If the user wants another kind,
say so plainly.

An agent never sends the first text of a conversation. A campaign, a workflow or
the contact does that (see the `campaign-setup` skill), and the agent picks up
from the contact's reply. Tool names are the PitchPrfct MCP tools, possibly
shown with a server prefix. The texts an agent sends come from the account's
numbers, so the sender rules in the `compliant-sending` skill apply to them.

The order below is the one `agents_get_build_guide` returns: its steps, the
seven brief items with their preset answers, every setting in plain words with
its default, and the writing rules. Read it once at the start; it is the same
text for every account and always matches the tools you have.

## 1. Check readiness first

Read before you ask anything, and never ask the user for an id.

- `agents_get_setup_options` shows which calendars are connected, the active
  Calendly event types, the phone numbers, workflows and SMS campaigns an agent
  can answer on with the agent that holds each today, the suggested time zones,
  the reply languages and the limit of every field. Its `summary` says it in
  one line you can read to the user. A list named in `unavailable` could not be
  read: treat it as unknown, not empty, and call again.
- A calendar is connected by a person in the app, never through a tool. When
  none is connected, give the user the `connectCalendarUrl` link and carry on
  with the draft: the agent can be built now but not published until a calendar
  is connected.
- An empty `phoneNumbers` list means there is nothing to attach yet
  (`phone_numbers_list` shows every number with its details). A number also
  needs an approved 10DLC campaign for its texts to deliver; for a gap, use the
  `ten-dlc-registration` skill.
- `agents_list` shows the agents that exist. If one already does this job,
  offer to edit it (`agents_get`, then `agents_update`) instead of creating a
  second one.
- A tool that answers 403 means either AI agents are not available on this
  account, or the connection lacks the access the step needs. Building needs
  Full access. Attaching channels and publishing need an account admin. A
  connection made before AI agents were added must be reconnected to get them.
  Say which, and stop.

## 2. Interview the user

Ask one thing at a time, in this order, with the option you recommend first.
Skip whatever the setup options or an earlier answer already settle. These are
the seven `brief` items of the build guide; say in a line why each matters when
it helps the user choose.

1. **Purpose.** What the appointment is for, and how long it takes. Offer three
   or four appointment types that fit the business, and lengths of 15, 30, 45
   or 60 minutes. The title and length go on every booking and size the slots
   the agent offers. Suggest the agent's internal name from the purpose.
2. **Persona.** How the agent should come across. Offer a few styles, such as a
   friendly front-desk coordinator, professional and brief, or warm and chatty.
   The user picks a style; you write the text.
3. **Questions.** What the business needs to know before it books someone.
   Propose two to four questions yourself, each with the rule a good answer
   meets (location: in or near Tampa; need: health insurance), and say which
   ones must be answered before a time is offered. No questions means the agent
   books everyone who is interested. Ask about follow-ups too: a nudge goes out
   only about a question still unanswered, and stops as soon as the contact
   replies. A sensible default is one nudge after 30 minutes.
4. **Calendar.** Google Calendar or Calendly. Offer the connected one first. A
   Calendly agent books exactly one event type, so ask which, by name.
5. **Hours.** The days and hours appointments can be booked, and the business
   time zone. Offer the presets of the build guide, Monday to Friday: business
   hours 9:00–17:00 (the usual choice), extended 8:00–20:00, mornings
   8:00–12:00, afternoons 12:00–18:00. Take the time zone from the account and
   ask only when it has none. A wrong time zone offers times at the wrong hour.
6. **Channels.** Which numbers, workflows or SMS campaigns the agent answers
   on. Offer the ones from `agents_get_setup_options` by name, and mark each one
   another agent already holds (`usedByAgent`): a channel belongs to one agent
   at a time, so tell the user before you plan to move one.
7. **Engagement.** Whether the agent answers every new contact, or only when
   the first message reads as positive. Always ask; never assume. Positive-only
   leaves neutral and negative first messages to a person. Once a conversation
   is running, the agent answers every reply either way.

After the seventh item, make **one** offer, and only one: "Want to check the
advanced settings?", with "No, use the defaults" as the recommended answer. The
advanced settings are optional: the wrap-up and off-topic instructions, the
hand-off message, ending idle conversations (and its tag), the time zone and
the reply language. Never ask them one by one in the interview, and never hold
the agent back for them. On a yes, list them with their defaults from the build
guide and ask only the ones the user picks; on a no, leave them out of the call
and the defaults apply.

## 3. Write the persona, instructions and questions

Write these yourself from the answers. Don't ask the user to write them.

- **Persona** (`persona`): who the agent is to the contact, in one or two
  sentences: its role and the business it represents. No questions, steps or
  rules in it.
- **Tone** (`tone`): a few words, such as "friendly, quick, professional".
- **Description** (`description`): one line for the user's team. The agent
  never reads it.
- **Instructions** (`instructions`): the agent's playbook, written to the agent
  in the second person. It holds what no other setting covers:
  - the flow, starting from the contact's reply: the order the agent works in,
    what makes someone a good fit, what it does when they qualify, what it does
    when they don't, how it handles "not now" or a question about price;
  - facts about the business the agent may share, and topics it must avoid or
    never promise.

  Refer to another setting in one line ("Ask the qualifying questions set for
  this agent", "When they qualify, move to booking") and never restate what it
  holds. Keep it under 6,000 characters; the agent reads only the first 8,000.
- **Questions** (`qualifyingQuestions`): each short and answerable in one text,
  in asking order. Set `required` only for a deal-breaker, put the rule in
  `qualificationCriteria` (leave it out when any answer qualifies), and set
  `followUpEnabled` only when follow-ups are on (`followUp`). Don't ask for
  what a booking already collects, such as the name or the time.

Leave these out of the instructions, because the platform or another field
already handles them, and a second copy can contradict the real setting:

- the qualifying questions and their rules, and the follow-up nudges;
- the calendar, the appointment title and length, booking hours, how times are
  offered and confirmed;
- the hand-off to a person and its message (`handoff`), the off-topic redirect
  (`offTopicInstructions`) and the wrap-up (`wrapUpInstructions`);
- the persona and tone, the reply language and the time zone;
- an opening step ("send the first text", "introduce the business"): the
  agent only ever replies, and the platform decides whether a greeting is due;
- opt-out words such as STOP, quiet hours, consent, carrier rules and SMS
  style. The platform enforces them.

Use only facts the user or the tools gave you. Where a fact is missing, write
the rule without it or leave a clearly marked gap such as "[add your service
area]", and tell the user about the gap. Name contact details in words ("the
contact's first name"); the agent never fills in placeholders in braces.

## 4. Show the proposal before you create

`agents_preview` takes the same fields as `agents_create`, plus the channel ids
the agent would get (`phoneNumberIds`, `workflowIds`, `campaignIds`) and, if
you have them, `rationale` (why this setup suits the user, a few short lines)
and `unsupported` (what they asked for that an agent cannot do). It creates
nothing.

- Show the user `text` as it is: the whole setup in plain words, the questions,
  the advanced settings in one line when they are the defaults, what they
  should know first ("Good to know") and the closing line that nothing exists
  yet. Don't paraphrase it into field names or ids.
- `warnings` says what still blocks publishing and names a channel that would
  move from another agent. Read them out.
- When the status says the setup cannot be created, `issues` names each field
  and what is wrong. Fix them and preview again.
- Wait for a yes, or for changes. After changes, preview again.

## 5. Create the draft

Once the user agrees, pass `createArguments` from the preview to
`agents_create` unchanged. (Without a preview, `agents_create` needs only
`name`, and takes `persona`, `tone`, `instructions`, `qualifyingQuestions`,
`calendarProvider`, `calendlyEventTypeId`, `appointmentTitle`,
`durationMinutes`, `bookingHours` with `start`, `end`, `days` and `timezone`,
`followUp` and `requirePositiveSentiment` in the same call.) The result is a
draft: nothing texts anybody yet. Read these back to the user:

- `summary.headline`: what happened and where the agent stands, in a sentence
  ("Created … as a draft. It is not published and answers nobody yet."). Say
  it; don't replace it with "done". `summary.sections` is the saved setup as
  label and value rows, and `appUrl` opens the agent in the app.
- `nextSteps`: what is left, in the order to do it, each with the tool to use
  or a link for what only a person can do in the app (connecting a calendar).
  Tell the user what comes next from this list.
- `warnings`: caveats about the text, such as instructions longer than the
  agent reads or a question repeated in the instructions. Fix them with
  `agents_update`.
- `incomplete`: fields that were not saved. Send them again with
  `agents_update`.
- `readiness.gaps`: everything that still blocks publishing, such as a calendar
  that is not connected or no channel yet. Each gap says how it is closed in
  `fixWith` (a tool and its field, or a person in the app at `appUrl`).

To change a draft later, send only the fields that change to `agents_update`.
A list replaces the whole list, so send every question, not only the new one,
and keep each question's `id`. `changedFields` says what the call changed, and
`summary.headline` says which of the changes apply at once and which take
effect after a publish. Tell the user that, in those words.

## 6. Add knowledge (optional)

When the agent should answer from a price list, an FAQ or a policy, put it in
a knowledge collection rather than in the instructions.

- `agent_knowledge_create_collection` makes a collection for this agent, and
  `agent_knowledge_create_source` adds content to it: plain text (`content`),
  question-and-answer pairs (`pairs`) or a table (`columns` and `rows`), picked
  with `type`. Files are uploaded in the app.
- Each source is indexed before the call answers. A status of ready means the
  agent can find it; failed means it was saved but not indexed, and `error`
  says why. `agent_knowledge_update_source` corrects a source,
  `agent_knowledge_list_collections` and `agent_knowledge_list_sources` show
  what exists.
- An agent searches every collection it owns unless its
  `knowledgeCollectionIds` names a list. When the result says
  `publishRequired`, the new collection reaches contacts after the next
  publish.
- `agent_knowledge_delete_collection` and `agent_knowledge_delete_source`
  cannot be undone, and the agent stops finding that content at once, also in
  running conversations. Confirm first.

Only store content the user supplied.

## 7. Test it

`agent_test_sessions_create` opens a test conversation with the draft, played
as one of the account's contacts (`contactUuid`, from `contacts_list`; ask the
user which contact to use). `agent_test_sessions_send` sends one message as
that contact would text it and returns the agent's reply.
`agent_test_sessions_reset` starts over and returns a new id to use. Tell the
user plainly before you start: each test message spends credits like a live
reply, and no SMS is sent, so the contact receives nothing.

Play a short conversation: an interested reply, an answer that fails a rule, a
question about price, a "not now". Show the user the replies. If the agent
restates a setting, invents a fact or misses a step, fix the instructions or
questions with `agents_update`, reset and test again. A draft that cannot run
yet is refused with 409 AGENT_NOT_READY, and `gaps` names what is missing. A
null reply with `skippedReason` means a rule stopped the agent, for example
because that contact opted out; pick another contact.

## 8. Confirm before go-live

Attaching a channel and publishing make the agent text real contacts on its
own. Before either call, show the user the whole setup from `agents_get`: its
`summary.sections` rows give it in order (the name, persona and tone, the
instructions, each question with its rule and whether it is required, the
follow-up timing, the calendar, the appointment title and length, the booking
hours and time zone, whether it answers every first message or only positive
ones), and name the exact numbers, workflows and campaigns it will answer on.
Name any channel that will be taken from another agent. Get an explicit yes.

## 9. Attach channels, then publish

- `agents_set_channels` first: a channel is one of the things publishing
  checks. Each list you send (`phoneNumberIds`, `workflowIds`, `campaignIds`)
  replaces that kind of channel, a list you leave out is kept. A channel
  another agent holds is refused with 409 CHANNEL_CONFLICT and the list of who
  holds what. Send `takeOver` only after the user agreed to move it.
- `agents_publish` then publishes the draft. If it answers 409
  AGENT_NOT_READY, nothing was published: `gaps` lists what is missing and how
  each is closed. Fix each gap (a calendar is connected in the app at
  `connectCalendarUrl`) and publish again. 409 DRAFT_CONFLICT means the draft changed meanwhile, so
  read the agent again; 409 ROLLOUT_CONFLICT means an earlier publish is still
  being applied, so publish again in a moment.
- Both calls answer with `summary.headline`: after the channels, whether the
  agent answers there yet; after the publish, that it now answers new
  conversations and on how many channels. Read it to the user.
- The agent is live when `answering` is true in `agents_get`: published,
  switched on and attached to at least one channel. Say it is live only then.
- Later edits change the draft. A live agent keeps answering with its
  published version until `agents_publish` runs again; `enabled`, the reply
  hours, the name and the channels apply at once. Conversations in progress
  finish on the version they started with unless `applyToLiveSessions` is sent.
- `agents_list_versions` and `agents_restore_version` bring an older version
  back into the draft. `agents_archive` retires an agent and frees its
  channels; confirm first.
- `agent_settings_update` holds two account-wide switches: the default agent
  for conversations no channel routes, and pause-all (`agentsPaused`), which
  stops every agent at once. Change either only when the user asks.

## 10. After go-live

- `agent_sessions_list` shows the conversations an agent handled. Filter with
  `agentId`, `status` or `needsAttention`. `agent_sessions_get` gives one, with
  why it escalated or ended, its outcome and the answers to the questions.
  `conversations_get_messages` reads the texts.
- A session that needs attention is waiting on a person. Tell the user which
  contact and why. Once someone has dealt with it,
  `agent_sessions_acknowledge` clears the flag.
- To take a conversation over, `conversation_agent_pause` stops the agent on
  that one conversation and cancels the replies and follow-ups it had waiting.
  `conversation_agent_resume` hands it back, and `conversation_agent_get` shows
  the current state. Neither sends anything.
- `agents_start_session` hands one existing conversation to a published agent
  (`conversationId` or `contactUuid`). The agent does not open a conversation,
  but when the newest message is a text from the contact that nobody answered,
  it replies at once, so this call can send an SMS. It also takes the contact
  out of the workflows that would keep texting them. Use it for one
  conversation the user names, after they agree. 409 THREAD_BUSY means another
  agent holds the conversation; send `replace` only if the user wants it moved.
- For a CRM, subscribe a webhook with `webhooks_api_create` to the four agent
  events: session started, lead qualified, escalated and session ended. Each
  carries the contact's uuid, name and phone, the reason code and the
  qualification answers. An escalation delivers both the escalated and the
  session-ended event for the same session. The appointment-booked event
  carries `agentId` and `sessionId` when an agent made the booking, which links
  a booking to its session.

## Never

- Create an agent the user has not seen as a proposal (`agents_preview`) and
  agreed to.
- Ask the advanced settings one by one. They are optional: one offer, once.
- Publish an agent or attach a channel without the user's explicit yes to the
  full setup.
- Say an agent is live before `agents_publish` succeeded and a channel is
  attached, or say a calendar is connected when the tools don't show it.
- Invent business facts: prices, addresses, hours, guarantees, staff names or
  policies.
- Put opt-out wording, quiet hours or other compliance rules into the
  instructions.
- Take a number, workflow or campaign from another agent (`takeOver`) without
  telling the user which agent loses it.
- Start sessions in bulk. `agents_start_session` is for one conversation at a
  time, each one named by the user.
- Delete a knowledge collection or source without confirming.
