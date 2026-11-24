
# Gmail Support Agent (n8n)

> A Gmail support agent that watches your inbox, uses an OpenAI chat model as its brain, remembers conversation context, checks your FAQ sheet, and creates a draft reply automatically.

---

## Overview

This n8n workflow turns your Gmail inbox into a first-line support assistant. When a customer emails a question, the agent looks up the answer in your FAQ sheet and saves a ready-to-review reply as a Gmail draft. Nothing is sent automatically, so you stay in control.

```
Gmail Trigger → AI Agent → Create a draft (Gmail)
                  │
                  ├── Chat Model: OpenAI Chat Model
                  ├── Memory:     Simple Memory
                  └── Tool:       Get row(s) in sheet (Google Sheets)
```

---

## What It Does

| Capability | How it's done | Node |
|---|---|---|
| **Watches your inbox** | Fires on every new incoming email and passes it to the agent | Gmail Trigger |
| **Uses an OpenAI chat model as the brain** | Understands the email, decides when to look up the FAQ, and writes the reply | OpenAI Chat Model |
| **Remembers conversation context** | Keeps recent messages so the agent can handle follow-ups in the same conversation | Simple Memory |
| **Checks your FAQ sheet** | Reads the Question/Answer rows and answers only from them | Get row(s) in sheet (Google Sheets) |
| **Creates a draft reply automatically** | Saves the agent's answer as a Gmail draft for you to review and send | Create a draft (Gmail) |

---

## Knowledge Base

File: `FAQ_Knowledge.xlsx` (upload to Google Sheets)

| Column | Description |
|---|---|
| `Question` | A common customer question |
| `Answer` | The approved answer |

It currently holds **14 FAQs** for a $10 lifetime-access course, covering:

- Course contents, lifetime access, certificate
- Refund policy (7-day full refund)
- Payment methods, receipts and invoices
- Beginner-friendliness, mobile access, software requirements
- Team pricing (20% off for 5+ seats)
- Password reset, missing enrollment email, changing account email
- Free preview (first module)

To teach the agent something new, add a row. No workflow changes are needed.

---

## Setup

1. **Upload the FAQ sheet:** import `FAQ_Knowledge.xlsx` into Google Sheets.
2. **Add credentials in n8n:**
   - Gmail OAuth2 (trigger and draft node)
   - Google Sheets OAuth2
   - OpenAI API key (API billing enabled; a ChatGPT subscription alone is not enough)
3. **Configure the Sheets tool:** select your FAQ document and `Sheet1`.
4. **Set the agent's system prompt** (see below).
5. **Configure Create a draft:** map the agent output to the message body, and use the sender, subject, and thread ID from the trigger so the draft replies in the original thread.
6. **Memory session key:** use the Gmail thread ID or sender address so different customers don't share context.
7. **Activate** the workflow.

### Suggested system prompt

```text
You are a customer support assistant. For every incoming email, use the
Google Sheets tool to read the FAQ knowledge base. Answer ONLY using
information found there. Write a short, friendly, professional reply.

If the question is not covered by the FAQ, respond with exactly:
NO_MATCH
```

---

## Known Issue: `NO_MATCH` Drafts

When an email doesn't match any FAQ (for example, a promotional email), the agent returns `NO_MATCH`, and the workflow still creates a draft containing that text.

**Fix:** add an IF node so drafts are only created for real answers.

```
Gmail Trigger → AI Agent → IF (output ≠ NO_MATCH) → Create a draft
                                    └─ else → label or notify a human
```

Also consider filtering the trigger (for example `-category:promotions`) so newsletters and marketing emails never reach the agent.

---

## Notes and Limitations

- Drafts are never sent automatically. Review each one first.
- Answers are only as good as the FAQ sheet, so keep it up to date.
- Simple Memory is held in n8n's process and resets on restart.
- Make sure the trigger doesn't fire on your own sent mail, to avoid loops.
- Works with any tool-calling OpenAI chat model (for example GPT-4o-mini for low cost). The model node can be swapped for another provider without changing the rest of the workflow.

---

## Files

| File | Purpose |
|---|---|
| `FAQ_Knowledge.xlsx` | FAQ knowledge base (Question / Answer) |
| n8n workflow JSON | Export from n8n and add it to this repo |
