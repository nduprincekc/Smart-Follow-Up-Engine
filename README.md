# 🚀 Smart Follow-Up Engine

An n8n workflow that automatically generates and delivers personalized, AI-written follow-up messages for sales leads — enriched from both HubSpot and Monday.com, and pushed to Slack, Telegram, and Microsoft Teams.

No more forgotten follow-ups. No more generic templates. Every lead gets a timely, context-aware nudge.

---

## ✨ What It Does

1. **Trigger** — runs on a schedule (or can be adapted to trigger on a new CRM event).
2. **Contact Enrichment** — looks up the same contact in parallel from:
   - **HubSpot** (via email search)
   - **Monday.com** (via `getByColumnValue` on the Contacts board)
3. **Merge & Normalize** — combines both sources into a single clean profile (`Name`, `Email`), gracefully falling back to whichever source has data.
4. **AI Message Generation** — an OpenAI-powered agent (`gpt-4o-mini`) writes a short, professional, personalized follow-up referencing the contact's last interaction.
5. **Multi-Channel Delivery** — the generated message is sent simultaneously to:
   - 💬 Slack
   - 📲 Telegram
   - 👥 Microsoft Teams (optional, disabled by default)

---

## 🧱 Workflow Architecture

```
Schedule Trigger
      │
Set Sample Data (contact_name, context, Email)
      │
      ├──> HubSpot Contact Lookup ──┐
      │                              ├──> Merge (combine by email) ──> Edit Fields (normalize Name/Email) ──> AI Agent (Generate Follow-up)
      └──> Monday Contact Fetch ────┘                                                                                │
                                                                                                    ┌──────────────────┼──────────────────┐
                                                                                                    ▼                  ▼                  ▼
                                                                                              Slack Reminder   Telegram Reminder   Teams Reminder
```

---

## 🔧 Requirements

- Self-hosted or cloud [n8n](https://n8n.io) instance
- **HubSpot** account + Private App token with contact read access
- **Monday.com** account + API token, with a "Contacts" board containing an email column
- **OpenAI API key** (model used: `gpt-4o-mini`, swappable)
- **Slack** app/bot token with `chat:write` scope
- **Telegram** bot (created via [@BotFather](https://t.me/BotFather))
- *(Optional)* **Microsoft Teams** app registration, if enabling the Teams node

---

## ⚙️ Setup

1. Import `27 - Smart Follow-Up Engine.json` into your n8n instance.
2. Configure credentials for each node:
   - `🔗 HubSpot Contact Lookup` → HubSpot App Token
   - `📋 Monday Contact Fetch` → Monday.com API credential
   - `🤖 AI Language Model` → OpenAI API credential
   - `💬 Slack Reminder` → Slack credential + target channel ID
   - `💬 Telegram Reminder` → Telegram bot credential + chat ID
   - `💬 Teams Reminder` → Microsoft Teams credential (if enabling)
3. In `📋 Monday Contact Fetch`, replace `boardId` and `columnId` with your actual board/column IDs.
4. In `💬 Slack Reminder`, set your real Slack channel ID.
5. In `💬 Telegram Reminder`, set the correct chat ID.
   - ⚠️ Telegram bots can only message chats that have first messaged the bot — have each recipient send `/start` to the bot at least once.
6. Update the signature block inside the `🤖 Generate Follow-up Message` prompt with your own business details.
7. Replace `📝 Set Sample Data` with a real trigger source (CRM webhook, form submission, etc.) once you're ready to move past testing.

---

## 🐞 Common Issues

| Symptom | Cause | Fix |
|---|---|---|
| Merge node returns no output | Merge node's output isn't wired to the next step | Confirm `Merge → Edit Fields` connection exists |
| Merged fields come out blank or garbled | Field names from HubSpot/Monday don't match, or are being concatenated instead of matched | Normalize field names in a Set node before merging; use fallback (`||`) logic, not concatenation |
| Telegram: `Bad Request: chat not found` | Recipient hasn't started a conversation with the bot yet, or the bot token doesn't match the bot you messaged | Send `/start` to the exact bot linked to your n8n credential; verify via `getMe` / `getUpdates` on the Telegram Bot API |
| AI message references missing data | Prompt points to a field that got dropped by an upstream Set node | Confirm the field exists on the item feeding the AI Agent node |

---

## 🛠️ Built With

- [n8n](https://n8n.io) — workflow automation
- [HubSpot API](https://developers.hubspot.com/)
- [Monday.com API](https://developer.monday.com/)
- [OpenAI API](https://platform.openai.com/) (`gpt-4o-mini`)
- Slack, Telegram, and Microsoft Teams APIs

---

## 📄 License

Internal/client project — adapt as needed for your own use case.

---

Built by **KC** · [KCEMMA HUB](https://github.com/nduprincekc)
