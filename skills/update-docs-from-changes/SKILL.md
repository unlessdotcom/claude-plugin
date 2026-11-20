---
name: update-docs-from-changes
description: Keep the Unless knowledge base in sync with product changes. Use when the user shipped a release, merged a pull request, has release notes or a changelog, or asks which help content a code change makes outdated, and wants the AI's documentation updated or new features documented.
---

# Update docs from product changes

Take a product change, find the content in the Unless content library that it makes outdated or leaves out, and update or add documents after the user approves each one, so the AI does not keep answering from the old behavior.

All Unless tools below come from the Unless connector. If they are not available, tell the user to connect Unless (`https://mcp.unless.com/mcp`) and stop.

## 1. Pick the workspace

Call `list_workspaces`. With one workspace, use it and name it. With more than one, ask the user which one to use. Every later call needs its `websiteId`.

## 2. Get the change

Use what the user points to, in this order of preference:

1. Release notes or a changelog.
2. A pull request, merged branch or range of commits in the code available in this session, such as `git log` and `git diff` between two tags.
3. A description from the user.

Ask whether the change is already live for customers. Documenting a feature before it ships makes the AI promise something visitors cannot use yet, so only continue for changes that are live, or ask the user to come back when they are.

## 3. List what changed for customers

Turn the change into a short list of things a customer would notice: new features, changed behavior, renamed or moved screens and settings, removed features, new limits or prices. Ignore refactors, internal tooling, tests and fixes that do not change what customers see or do.

Read the code when the release notes are vague, but describe each change in product language, never in code terms. Show the list and let the user correct it before going on.

## 4. Find the affected content

For each change, call `search_content_library` with a few queries: the feature name, the old name or behavior, and the questions a customer would ask about it. Read the hits with `get_content_library_page`. Decide per change:

- **Outdated**: a page describes the old behavior, an old name or a removed feature.
- **Missing**: a new feature, setting or limit that no page covers.
- **Fine**: the content still matches, or the change does not affect what customers ask about.

Show the result per change, with the pages involved.

## 5. Draft the updates

Write the content in the language the existing content library uses, unless the user asks otherwise.

- **Outdated content in a document**: the page's `sections` name the source each part came from. When a section's `trainingDataId` belongs to a document (`get_training_data` with that `trainingDataId` returns `type: faq`, with the title in `question` and the body in `answer`), draft the full updated body of that document, changing only what the product change affects. Only trust a section's `trainingDataId` when its `sourceInherited` is false. When it is true, the ID is the page's first source, not necessarily the one that wrote the section, so confirm with the user which source holds the text.
- **Outdated content from a website or file**: Unless cannot change it from here. Name the page, the source URL and the exact text to change, so the user can fix it where that content is maintained. Unless picks it up on the next scan.
- **Missing content**: draft a new document. Use a title a customer would search for, start with what the feature does and who it is for, then how to use it. Use markdown headings, lists and tables for longer documents, and stay under 20,000 characters for the title and body together.

Never invent behavior. Every statement must come from the change, the code or the user.

## 6. Review every draft with the user

Never save anything the user has not approved. For each draft show whether it is an update or new, the page it replaces, and the new text, with the changed parts easy to spot. Ask the user to approve, edit or skip it.

## 7. Save the approved documents

Save each approved document with `upsert_document`. Send the `trainingDataId` to update an existing document, or leave it out to create a new one:

```json
{
  "websiteId": "<websiteId>",
  "body": {
    "document": {
      "trainingDataId": "<only when updating>",
      "title": "<title>",
      "body": "<the approved content in markdown>"
    }
  }
}
```

When a save fails, show the error and continue with the next one.

## 8. Wrap up

Summarize: the changes that were checked, the documents updated and created, and the website or file sources the user still has to fix themselves. Tell the user that Unless retrains the AI on the changed documents automatically.
