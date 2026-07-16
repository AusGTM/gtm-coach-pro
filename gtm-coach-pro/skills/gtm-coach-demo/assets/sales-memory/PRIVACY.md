# Privacy & data handling

> **This is a synthetic demo bank.** Every company, person, deal, and quote in this
> `sales-memory/` is **fictional**, created for demonstrating GTM Coach Pro. It contains no
> real customer data.

In real use, this folder stores summaries/transcripts of **real customer conversations**
locally, and the following applies:

- **Local only.** Data stays in `sales-memory/` on your machine and is never sent anywhere
  except the MCP tools you already connected.
- **Recording-consent reminder.** Many jurisdictions require all-party consent to record calls
  (e.g. two-party-consent US states, GDPR in the EU). You are responsible for having captured
  calls lawfully; GTM Coach only reads what your connected tool already has.
- **Redaction option.** GTM Coach can redact personal data (emails, phone numbers, personal
  names of non-buying-committee individuals) on request; set `redaction: on` in `config.json`.
- **Deletion.** Deleting the `sales-memory/` folder removes all stored memory.
