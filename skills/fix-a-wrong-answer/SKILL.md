---
name: fix-a-wrong-answer
description: Find out why the Unless AI gave a wrong, outdated or unhelpful answer and fix the cause. Use when the user shares a conversation that went wrong, asks why the AI said something, wants to correct an answer, or wants to work through the answers visitors downvoted.
---

# Fix a wrong answer

Start from a conversation or from the answers visitors downvoted, find out why the AI answered the way it did, and fix the cause after the user approves: the content, the AI's instructions, or the procedure involved.

All tools below come from the Unless connector. If they are not available, tell the user to connect Unless (`https://mcp.unless.com/mcp`) and stop.

## 1. Pick the workspace

Call `list_workspaces`. With one workspace, use it and name it. With more than one, ask the user which one to use. Every later call needs its `websiteId`.

## 2. Find the answers to fix

**From a conversation**: when the user gives a conversation ID, or a dashboard link to a conversation (it ends in `<sessionId>/<conversationId>`, so take the last path segment), call `get_conversations` with `conversationId`. Never pass a dashboard link as `conversationUrl`: that parameter is the transcript's storage key. Ask which answer was wrong when that is not obvious, and what the right answer would have been, if they know.

**From downvoted answers**: when the user has no specific conversation, call `get_insights_grouped_questions` with `type: commonlyDownvotedQuestions` for the past 30 days, or the period the user asked for. Show the list sorted by frequency and let the user pick which to work on. For each picked question, open one of its `conversations` with `get_conversations` to see the actual answer.

## 3. Diagnose the cause

In the transcript, each AI message lists the sources it was given in `metadata`: the `title`, the `source` and whether the AI marked it `isUsedForAnswer`. Read the sources the answer used, and search for what the right answer should have been with `search_content_library`. Read the relevant pages with `get_content_library_page`.

Call `get_ai_configuration` and read `customRules`, so you know the instructions the AI follows on every answer. When the answer involved a procedure, read it with `get_procedures`.

Name the cause, with the evidence:

- **Missing content**: nothing in the content library answers the question, so the AI guessed or used a loosely related source.
- **Wrong or outdated content**: a source says something that is no longer true.
- **Conflicting content**: two sources disagree, and the AI picked the wrong one.
- **Wrong audience**: the right answer exists, but for another kind of customer, or the content is restricted from this visitor.
- **Instructions**: `customRules` or the configuration made the AI answer this way, for example a tone, a length or a rule that does not fit this case.
- **Procedure**: a procedure ran when it should not have, did not run, or returned something the AI misread.
- **Question was unclear**: the AI answered a reasonable reading of an ambiguous question. Often no fix is needed, or only a clarifying sentence in the content.

When you cannot tell the right answer from the content, ask the user. Never invent it.

## 4. Propose the fix

Explain the cause in plain language and propose the smallest fix that solves it:

- **Content in a document**: when the page section's `trainingDataId` belongs to a document (`get_training_data` with that `trainingDataId` returns `type: faq`, with the title in `question` and the body in `answer`), propose the corrected text. Only trust a section's `trainingDataId` when its `sourceInherited` is false. When it is true, the ID is the page's first source, not necessarily the one that wrote the section, so confirm with the user which source holds the text.
- **Content from a website or file**: Unless cannot change it from here. Name the page and the source URL the user should fix, and Unless picks it up on the next scan. When that takes too long, offer a document with the correct answer in the meantime, and say that the source still has to be fixed so the two do not conflict.
- **Missing content**: a new document, with the question as the title.
- **Wrong audience**: a document restricted to the right audience with `restrictedAudiences`, using an audience from `get_audiences`.
- **Instructions**: the exact change to `customRules`. Add or reword one rule, and keep every other rule as it is.
- **Procedure**: the change to its description or settings. Procedures are complex, so describe the change and let the user decide whether to apply it here or in the dashboard.

Show the fix and ask the user to approve, edit or skip it.

## 5. Apply the approved fix

- **Documents**: call `upsert_document`. Send the `trainingDataId` to update an existing document, with the full new title and body, or leave it out to create one:

  ```json
  {
    "websiteId": "<websiteId>",
    "body": {
      "document": {
        "trainingDataId": "<only when updating>",
        "title": "<the question or topic>",
        "body": "<the corrected answer in markdown>"
      }
    }
  }
  ```

  Answer the question in the first sentence, keep it general enough for every visitor, and stay under 20,000 characters.
- **Instructions**: call `update_ai_configuration` with only `customRules`, holding the complete new text.
- **Procedures**: call `upsert_procedure` with the whole procedure and your change applied, because the write replaces it.

When a save fails, show the error and continue.

## 6. Wrap up

Summarize per answer: the cause, the fix that was applied, and anything the user still has to do, such as fixing a website source. Tell the user that Unless retrains the AI on changed documents automatically, and suggest asking the same question again once that is done to check it.

When the account has quality control, offer to add the question with the correct answer as a control question with `upsert_quality_control_question`, so a future change cannot break it again unnoticed.
