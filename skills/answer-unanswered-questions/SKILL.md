---
name: answer-unanswered-questions
description: Turn the questions an Unless AI agent could not answer into Q&A content. Use when the user wants to fill knowledge gaps, close content gaps, answer unanswered or missed questions, improve the AI's answer rate, or asks what visitors asked that the AI could not answer.
---

# Answer unanswered questions

Fetch the questions the Unless AI could not answer, draft an answer for each one, review every draft with the user, and save the approved ones as Q&A sources in Unless. Unless trains the AI on each new Q&A and places it in the content library on its own, so this skill never edits the content library directly.

All tools below come from the Unless connector. If they are not available, tell the user to connect Unless (`https://mcp.unless.com/mcp`) and stop.

## 1. Pick the workspace

Call `list_workspaces`. With one workspace, use it and name it. With more than one, ask the user which one to use. Every later call needs its `websiteId`.

## 2. Fetch the unanswered questions

Default to the past 30 days unless the user asked for another period. Compute `startTime` and `endTime` as Unix timestamps in milliseconds.

Call `get_insights_grouped_questions` with `websiteId`, `startTime`, `endTime` and `type: commonlyUnansweredQuestions`, so only the unanswered list comes back. Each entry is a cluster of similar questions: `message` is the most common phrasing and `frequency` is how often it was asked.

Then drop questions that are not worth an answer: greetings, tests, gibberish, abuse, requests for a human, pasted prompts or documents, questions unrelated to the business, questions about one visitor's own account or order that no general answer can cover, and follow-ups that only make sense inside their conversation ("and where can I find that?"). When a follow-up hides a real question, rephrase it as a standalone question instead of dropping it. Use the `title` of its `conversations`, which summarizes what the conversation was about, and do not open transcripts in this step. Questions from the same conversation that ask the same thing become one question. Mention how many you dropped and why, in one line.

Call `get_training_data` with the `websiteId` and look at the sources with `type: faq`. Skip unanswered questions that an existing Q&A already covers, and say which ones you skipped.

If nothing is left, tell the user and stop.

Show the remaining questions as a short numbered list, sorted by frequency, with the frequency next to each. With more than 10, propose working through the top 10 first and let the user choose.

## 3. Draft the answers

When the questions come in more than one language, ask once whether to write every Q&A in one main language (the Unless AI translates when it answers) or each in the language of its question. Draft one answer per question. Use these sources, in this order:

1. What the user told you in this conversation.
2. The workspace's own content: `get_content_library_tree` and `get_content_library_page` show what the knowledge base already says about related topics. Stay consistent with it.
3. The conversations the question came from: each entry lists up to 10 `conversations`. Transcripts are large, so only open them for the questions you are drafting now, and only when the question and its conversation `title` leave the meaning unclear. Then read one, at most two, with `get_conversations` and a `conversationId`.
4. Other sources available in this session, such as files, code or documentation the user has open, or web search on the company's own public site.

Never invent facts. Prices, limits, dates, policies, legal terms, contact details and product behavior must come from a source you actually read. When you cannot find the answer, do not guess: mark the draft as needing input and ask the user what the answer is.

Write answers that stand on their own, because the AI will reuse them for many visitors:

- Answer the question in the first sentence, then add the details that matter.
- Keep it short. The question and the answer together must stay under 1900 characters.
- No greetings, no "as an AI", no references to this conversation or to a specific visitor.
- Plain text or simple Markdown lists. Links only to pages you have verified.

## 4. Review every draft with the user

Never save a Q&A the user has not approved. Go through the drafts one at a time, or in small batches when the user prefers that. For each one show:

- the question (you may reword it into a clean, general question),
- how often it was asked,
- the draft answer,
- where the answer came from, or that it needs input.

Ask the user to approve, edit, or skip it. Apply edits and show the result again when the change is more than a small correction.

## 5. Save the approved Q&As

Save each approved pair as soon as it is approved with `ingest_training_data`:

```json
{
  "websiteId": "<websiteId>",
  "body": {
    "trainingData": {
      "type": "faq",
      "method": "single",
      "name": "<the question>",
      "question": "<the question>",
      "answer": "<the approved answer>",
      "isPublic": false,
      "url": "",
      "include": [],
      "exclude": []
    }
  }
}
```

Leave out `trainingDataId`, a new source is created. `isPublic` is required but Unless always stores Q&As with `false`, so don't ask the user about it. When a save fails, show the error and continue with the next question.

## 6. Wrap up

End with a short summary: how many Q&As were saved, skipped and left open, and the questions that still need an answer from someone else. Tell the user that Unless trains the AI on the new Q&As and adds them to the content library automatically, and that resulting content library changes can show up as tasks to review in the Unless dashboard.
