# PitchPrfct plugins for Claude Code

The official [PitchPrfct](https://www.pitchprfct.com) plugin marketplace. The `pitchprfct`
plugin connects Claude to your PitchPrfct account. Claude can then look up contacts, send compliant
SMS, run campaigns and workflows, and walk you through 10DLC registration.

## Install

In Claude Code:

```
/plugin marketplace add PitchPrfct/claude-plugins
/plugin install pitchprfct@pitchprfct-plugins
```

The first time Claude uses a PitchPrfct tool, your browser opens the PitchPrfct sign-in. Approve
access for the account you want Claude to work on. There is no API key to paste.

## What's included

| Component | What it does |
| --- | --- |
| `pitchprfct` MCP server | The PitchPrfct Developer API as tools: contacts, lists, tags, messages, conversations, campaigns, workflows, templates, webhooks, phone numbers, credits and 10DLC. Endpoint: `https://app.pitchprfct.com/api/mcp` (OAuth). |
| `compliant-sending` skill | Checks before any text: sender readiness, opt-outs, first-contact wording, quiet hours (8:00–21:00 in the contact's local time), credits and idempotent retries. |
| `ten-dlc-registration` skill | Brand, then 10DLC campaign, then attaching phone numbers, including sole-proprietor OTP, vetting, and fixing declined registrations. |
| `campaign-setup` skill | Readiness checks, building an audience, drip pacing, cost estimates and a confirmed launch for SMS and workflow campaigns. |

## Documentation

- API and MCP reference: https://app.pitchprfct.com/api/docs
- Privacy policy: https://www.pitchprfct.com/privacy
- Terms: https://www.pitchprfct.com/terms

## Support

This repository is a read-only mirror, published automatically from PitchPrfct's main codebase.
Pull requests here are not merged. To report a problem or ask for help, open an issue in this
repository or contact us at https://www.pitchprfct.com/contact.

## License

The `pitchprfct` plugin is released under the MIT License. See `plugins/pitchprfct/LICENSE`.
