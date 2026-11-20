---
name: tickets-to-knowledge
description: Turn the answers buried in old support tickets into documents in the Unless knowledge base. Use when the user wants to learn from their helpdesk history, mine Zendesk, Freshdesk, TOPdesk, Salesforce or HubSpot tickets, turn tickets into FAQs or articles, find undocumented knowledge, or fill knowledge gaps from what their support team already answered.
---

# Tickets to knowledge

Read support tickets from the helpdesk the user has connected in this session, find the questions their team answers again and again, check what the Unless content library already covers, and save the missing answers as documents after the user approves each one. The tickets themselves never go to Unless, only the approved, anonymous answers.

All Unless tools below come from the Unless connector. If they are not available, tell the user to connect Unless (`https://mcp.unless.com/mcp`) and stop.

## 1. Pick the workspace

Call `list_workspaces`. With one workspace, use it and name it. With more than one, ask the user which one to use. Every later call needs its `websiteId`.

## 2. Get the tickets

Use what the user has available in this session, in this order:

1. A helpdesk connector, such as Zendesk, Freshdesk, HubSpot or Salesforce.
2. An export file, such as a CSV or JSON export of tickets.
3. Tickets the user pastes.

When none of these is available, ask the user to connect their helpdesk or export the tickets, and stop.

Agree on the scope before reading: the period (default the past 6 months), and optionally a group, product or tag. Only use tickets that are solved and contain an answer from the support team. Skip spam, auto-replies, bounced mail and tickets without a reply.

With more than a few hundred tickets, work in batches of about 200 and keep a running list of topics, so you never need every ticket in context at once.

## 3. Find the recurring questions

Group tickets that ask the same thing, even when the wording differs. For each group, note:

- the question as a customer would ask it,
- how many tickets asked it,
- the answer the support team gave, and whether the answers agree,
- one or two ticket IDs, so the user can check the source.

Drop groups that cannot become a general answer: questions about one customer's own account, order or data, bug reports that were fixed since, complaints, and one-off cases. When the team gave conflicting answers, keep the group but flag it, because the user has to decide which answer is right.

Sort the groups by ticket count and show the list. With more than 15, propose working through the top 15 first and let the user choose.

## 4. Check what the content library already covers

For each group, call `search_content_library` with the question in plain language. Read the top hits with `get_content_library_page` to see what they actually say. Decide per group:

- **Covered**: the content library already answers it correctly. Skip it and name the page.
- **Outdated or incomplete**: a page covers it, but the tickets show a newer or fuller answer. Plan an update (step 6).
- **Missing**: nothing covers it. Plan a new document.

Tell the user how many groups were already covered, in one line.

## 5. Draft the documents

Write one document per group. When the questions in the tickets come in more than one language, ask once whether to write every document in one main language (the Unless AI translates when it answers) or each in the language of its tickets.

- Use the question as the title.
- Answer the question in the first sentence, then add the steps or details that matter. Use markdown headings, lists and tables for longer answers.
- Generalize the answer. Remove every personal detail from the tickets: names, email addresses, phone numbers, account or order numbers, and anything else that points to a customer or employee.
- Never invent facts. Prices, limits, dates, policies and product behavior must come from the tickets, the content library or the user. When tickets disagree or are old, ask the user which answer is current.
- Stay under 20,000 characters for the title and body together.

Some answers depend on who is asking, for example accountants versus business owners. Call `get_audiences` once and, when an existing audience matches, propose restricting the document to it with `restrictedAudiences`. Never create audiences in this skill.

## 6. Review every draft with the user

Never save a document the user has not approved. Go through the drafts one at a time, or in small batches when the user prefers that. For each one show:

- the title and how many tickets asked it,
- the draft body,
- whether it is new or an update of an existing page, and which page,
- the audience restriction, if any,
- anything that needs the user's input.

Ask the user to approve, edit or skip it.

## 7. Save the approved documents

Save each approved document as soon as it is approved, with `upsert_document`:

```json
{
  "websiteId": "<websiteId>",
  "body": {
    "document": {
      "title": "<the question>",
      "body": "<the approved answer in markdown>",
      "restrictedAudiences": ["<audienceId>"]
    }
  }
}
```

Leave out `restrictedAudiences` when the document is for everyone.

**Updating an existing page**: the page's `sections` name the source each part came from. When a section's `trainingDataId` belongs to a document (`get_training_data` with that `trainingDataId` returns `type: faq`, with the title in `question` and the body in `answer`), update that document by sending its `trainingDataId` with the full new title and body. Only trust a section's `trainingDataId` when its `sourceInherited` is false. When it is true, the ID is the page's first source, not necessarily the one that wrote the section, so confirm with the user which source holds the text. When the content comes from a website or a file, Unless cannot change it from here: tell the user which page and source URL to fix, or offer to save a new document with the correct answer instead.

When a save fails, show the error and continue with the next document.

## 8. Wrap up

End with a short summary: how many ticket groups were found, how many were already covered, how many documents were created or updated, which were skipped, and which still need an answer from someone. Tell the user that Unless trains the AI on the new documents and adds them to the content library automatically.
