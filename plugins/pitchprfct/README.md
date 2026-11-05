# PitchPrfct for Claude

Connect Claude to your PitchPrfct account. Claude can then look up contacts, send compliant SMS, run
campaigns and workflows, and walk you through 10DLC registration. You sign in with your own
PitchPrfct account. There is no API key to paste.

## What the plugin contains

- **The `pitchprfct` MCP server.** This is the PitchPrfct Developer API as tools: contacts, lists,
  tags, messages, conversations, campaigns, workflows, templates, webhooks, phone numbers, credits
  and 10DLC. The plugin holds only the server address, not the server code.
- **Three skills.** A skill is a set of written instructions that tells Claude how to use the tools
  safely. Skills do not run code and do not send data.
  - **compliant-sending**: use before you send any SMS, whether one text, a reply in a conversation
    or a scheduled message. Claude checks sender readiness, opt-outs, first-contact wording, quiet
    hours (8:00 to 21:00 in the contact's local time), credits and safe retries. It asks you to
    confirm the recipient, the sender, the exact text and the cost before it sends.
  - **ten-dlc-registration**: use when your account must register or fix its 10DLC setup, or when
    texts are blocked, filtered or held. Claude goes through the brand, then the 10DLC campaign, then
    attaching phone numbers. It explains statuses and vetting, and asks before it orders anything
    that costs money.
  - **campaign-setup**: use when you want to text many contacts at once. Claude checks that the
    account is ready, builds the audience, sets the drip pace, estimates the credit cost, and starts
    the campaign only after you confirm it.

The plugin has no hooks, no scripts, no commands and no stored credentials. Apart from the
manifest, the server address and the three skill files, it holds only its icon.

## What the plugin connects to

The plugin connects to one host, `app.pitchprfct.com`, which PitchPrfct runs:

- **Tool calls** go to `https://app.pitchprfct.com/api/mcp`.
- **Sign-in** opens in your browser on the same host the first time Claude uses a PitchPrfct tool.
  This is an OAuth sign-in. You choose the PitchPrfct account that Claude works on and the access
  level. Claude receives a token for that account. It never receives your password or an API key.

The plugin sends nothing to any other destination. It collects no analytics.

## What data moves, and what Claude can do

When Claude calls a tool, Claude sends the request to the PitchPrfct server and receives the
result. The tools read and change data only in the PitchPrfct account you chose at sign-in, and only
within the access level you granted:

- **Read-only**: see contacts, conversations, campaigns, workflows and reports. Claude cannot send,
  change or delete anything.
- **Full**: everything Read-only allows, plus send messages (this uses credits), edit contacts, and
  start campaigns and workflows.

Your role in the account can limit you to Read-only. An account admin can change your role.

Text messages that Claude sends through PitchPrfct go to real phones and use your credits. You
can disconnect Claude at any time in Integrations & Apps in PitchPrfct.

PitchPrfct handles your data as its privacy policy describes (link below).

## Requirements

- A PitchPrfct account with the Developer API enabled. If it is not enabled, ask an account owner to
  enable it.
- To send texts: enabled phone numbers, an approved 10DLC brand and campaign, an active
  subscription and enough credits. The skills check these for you.

## Install

From the plugin directory: open the directory in Claude, find **PitchPrfct**, and add it.

From Claude Code:

```
/plugin marketplace add PitchPrfct/claude-plugins
/plugin install pitchprfct@pitchprfct-plugins
```

Then ask Claude to do something with your PitchPrfct account. Your browser opens the PitchPrfct
sign-in. Approve access for the account you want Claude to work on.

## Links

- API and MCP reference: https://app.pitchprfct.com/api/docs
- Privacy policy: https://www.pitchprfct.com/privacy
- Terms: https://www.pitchprfct.com/terms
- Support: https://www.pitchprfct.com/contact

## License

MIT. See the LICENSE file in this folder.
