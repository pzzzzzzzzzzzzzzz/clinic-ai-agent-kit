# 3 Working AI Agents for n8n — Free Sample

Three complete n8n workflows you can import and run today. Every prompt and threshold is an
editable node — nothing is hardcoded inside a script.

| # | Workflow | File | What it does |
|---|---|---|---|
| 1 | **Lead Qualifier** | `01-lead-qualifier.json` | Scores an inbound enquiry against criteria you set, routes qualified leads to your sheet/CRM, and tells you *why* it scored that way |
| 2 | **Ticket Triage** | `02-ticket-triage.json` | Sorts every ticket into `urgent` / `normal` / `spam`, drafts a reply, and flags what a human must see (urgent always pings out — not just a label) |
| 3 | **Invoice Chaser** | `03-invoice-chaser.json` | Runs daily, sends polite escalating reminders for overdue invoices, and stops after N reminders so it never becomes the annoying email |

---

## Install in about 2 minutes

### You need
- An n8n instance — [n8n Cloud](https://app.n8n.io) (free tier is fine) or self-hosted (`npm install n8n`)
- One OpenAI-compatible API key (works with OpenAI, and with any endpoint that speaks the same API)

### Steps
1. **Import.** In n8n: *Workflows → Import from File* → pick `01-lead-qualifier.json` (repeat for the other two).
   *Or* import the single package `shipped-automations.n8np` if you have it — it contains all three and
   runs a pre-flight check before import, so a broken file tells you immediately instead of half-way.
2. **Add your API key.** Open the *Call model* node → Credentials → create a new **Header Auth** credential:
   - Name: `Authorization`
   - Value: `Bearer sk-...`
   Pick it in both workflow 1 and workflow 2 (workflow 3 doesn't call a model).
3. **Turn it on.** Imported workflows arrive **inactive** — click **Active** on each one, or hit the
   *Active* toggle in the workflow list.
4. **Send it a test.**
   - Lead Qualifier: `POST https://your-n8n/webhook/lead-qualifier`
     ```json
     { "id": "req-001", "name": "Ann", "email": "ann@example.com", "company": "Acme", "message": "We need lead scoring wired to HubSpot", "source": "website_form" }
     ```
   - Ticket Triage: `POST https://your-n8n/webhook/ticket-triage`
     ```json
     { "id": "tkt-001", "subject": "Card declined", "body": "I tried to pay 3 times and it keeps failing", "from": "bob@example.com" }
     ```

---

## The part that matters: everything is editable

Two nodes per workflow are yours to edit, and **nothing else needs to change when you edit them**:

| Node | What to change there |
|---|---|
| **`Config (edit me)`** | model name, endpoint URL, score threshold, reminder days, reminder cap, sender email, webhook URLs, sheet name — plain fields, no code |
| **`Prompt (edit me)`** | the actual instructions and scoring criteria the model uses — a text block at the top of one node |

The node after the prompt (`Build request`) only formats; the node after the model (`Parse response (safe)`)
only parses. That's the whole point of the layout: **you can change what the agent says without touching code.**

### How the "safe" part works
Every model call uses `temperature: 0` and `response_format: json_object`. The parse step is wrapped in
`try/catch` — if the model returns malformed JSON, the workflow **degrades instead of crashing**:
you get `status: "error"` with the raw excerpt, rather than a dead run.

### Ticket Triage routing
`urgent` → escalates through your webhook (Slack/email/anything) **and** sets `needs_human: true`
`normal` → writes a **draft** only, never auto-sends (auto-sending customer email is how you get a complaint)
`spam` → filed away

### Invoice Chaser setup
1. Put your invoices in a Google Sheet with a header row:
   `invoice_id | client_name | client_email | amount | currency | due_date | reminder_count`
2. Publish the sheet to the web (*File → Share → Publish to web*) and paste the sheet ID into `Config`.
3. Set an email API key credential (*Header Auth*: `Authorization` = `Bearer …`) — the sample targets a
   Resend-style endpoint; change the URL and body fields if your provider differs.
4. Every reminder is **BCC'd to you** — that paper trail is the point, and it's why you can prove delivery.

Reminder tone escalates: friendly → second reminder → final. It stops at `maxReminders` and never nags past it.

---

## Common questions

**Can I use a different model?** Yes. Any OpenAI-compatible endpoint: change `model` and `baseUrl` in
`Config (edit me)`. If your endpoint doesn't support `response_format`, set `useJsonMode` to `false`.

**Can I change the language?** Yes — rewrite the prompt in `Prompt (edit me)`. The output JSON keys
stay in English so the downstream nodes keep working.

**Can I use Make or Zapier instead of n8n?** Yes. Every workflow is just HTTP calls and a schedule:
point the webhook at your form, and the outbound webhooks at Make/Zapier. You'd rebuild the node
graph, but none of the logic is n8n-specific.

**Do I need to keep this on my account?** Yes — these run on **your** n8n instance, in your account,
with your keys. Nothing is hosted for you, and you can delete it any time.

**What about the n8n licence?** These workflows are free for you to use, modify, and run on your own
instance, including commercially. n8n itself is fair-code licensed (Sustainable Use License) and is not
being redistributed here — you're getting a working configuration, not a copy of n8n.
Please don't redistribute these files as a paid template; if you want one rebuilt around your own stack,
that's a service, and it's a different thing.

---

## Support

Questions about the node setup? Reply to wherever you got this — usually the fastest route.
For a version rebuilt around **your** CRM, **your** inbox, **your** rules — that's a separate paid
engagement, and the free sample stays free either way.

Files: `01-lead-qualifier.json` · `02-ticket-triage.json` · `03-invoice-chaser.json` · `CREDENTIALS.md`
