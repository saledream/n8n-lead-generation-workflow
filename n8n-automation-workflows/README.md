# n8n Automation Workflows

Practical [n8n](https://n8n.io) workflows I built and use. Each workflow lives in its own folder with an importable `workflow.json` and notes.

| Workflow | What it does |
|---|---|
| [Contact Form Lead Follow-up](contact-form-lead-followup/) | Logs a new lead, writes a personalised reply with an LLM, emails it, and alerts the owner on Telegram |

---

## Contact Form Lead Follow-up

Turns a website contact-form submission into an instant, personalised response with no manual work.

### Flow

```
Webhook (POST) → Google Sheets (append lead) → LLM chain (write reply) → Gmail (send) → Telegram (notify owner)
                                                     ↑
                                         OpenRouter chat model
```

1. **Contact Form Webhook** receives a `POST` at `/contact-form-lead`.
2. **Save Lead to Sheet** appends Name, Email, Message and a timestamp to a Google Sheet, so the lead is stored even if a later step fails.
3. **Write Follow-up Email** (LLM chain via OpenRouter) drafts a short, warm reply that refers to the specific points in the lead's message.
4. **Send Follow-up Email** sends it through Gmail with the subject `Thanks for contacting us, <name>`.
5. **Notify on Telegram** tells the owner a new lead arrived and that the follow-up was sent.

### Expected payload

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "message": "I'd like a quote for..."
}
```

Field names are accepted in either case (`name` / `Name`, etc.), so most form tools work without changes.

### Design choices

- **Lead is saved before the LLM runs.** A model or email failure never loses the lead.
- **Constrained prompt.** The system prompt fixes length (80–120 words), plain text, no subject line, and a set sign-off, so replies stay consistent without review.
- **Swappable model.** The workflow uses an OpenRouter chat model node; change the model in one place.
- **Owner alert.** The Telegram message gives the owner the lead details immediately, so they can step in for high-value leads.

### Setup

1. In n8n choose **Import from file** and select `contact-form-lead-followup/workflow.json`.
2. Create credentials and attach them to the nodes:
   - Google Sheets (OAuth2)
   - Gmail (OAuth2)
   - OpenRouter API key
   - Telegram bot token (create a bot with [@BotFather](https://t.me/BotFather))
3. Replace the placeholders:
   - `YOUR_GOOGLE_SHEET_ID`: ID of your sheet (the long string in its URL). The sheet needs the columns `Timestamp`, `Name`, `Email`, `Message`, on a tab called `Form Responses 1` (or change the tab name in the node).
   - `YOUR_TELEGRAM_CHAT_ID`: your chat ID with the bot.
   - `YOUR_CREDENTIAL_ID`: pick your own credentials in each node.
4. Choose the model in the **OpenRouter Chat Model** node.
5. Activate the workflow and point your form's webhook at the **production** URL.

### Test it

```bash
curl -X POST "https://YOUR_N8N_HOST/webhook/contact-form-lead" \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"you@example.com","message":"Hi, I want to know about your pricing."}'
```

### Possible improvements

- Validate the email address and reject spam before the LLM step.
- Add an error workflow that alerts on Telegram when a step fails.
- Add a webhook secret header check.

## About

Built by [Saleamlak Msgan](https://github.com/saledream). I also build AI-assisted SOC automation with n8n and MCP servers on top of Wazuh and Active Directory.
