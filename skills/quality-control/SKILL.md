---
name: quality-control
description: Test the Unless AI against a set of control questions with known correct answers, run a quality report, and fix what fails. Use when the user wants to test or measure answer quality or correctness, write control or test questions, run a quality control report, check the AI before or after a content change, or find out where the AI answers wrong.
---

# Quality control

Build a set of control questions with the answers the AI should give, run a quality control report that asks the AI every question and grades each answer, and explain the failures with a fix for each one.

All tools below come from the Unless connector. If they are not available, tell the user to connect Unless (`https://mcp.unless.com/mcp`) and stop.

Quality control reports are only available on the Enterprise, Flex, Fixed and Plus plans. When `generate_quality_control_report` answers `403`, tell the user their plan does not include it, and stop after saving the questions.

## 1. Pick the workspace

Call `list_workspaces`. With one workspace, use it and name it. With more than one, ask the user which one to use, listing each workspace with its `description` when it has one. When the request makes one workspace the clear fit, suggest it. Every later call needs its `websiteId`.

## 2. Check what is already there

Call `get_quality_control_questions` and `get_quality_control_reports`. Tell the user how many control questions exist and the grade and pass rate of the latest report, if any. Then ask what they want:

- **Add questions**, when there are none or too few to say much (fewer than about 20).
- **Run a report** on the current questions.
- **Review the latest report** without running a new one.

## 3. Write control questions

A good control set covers what visitors really ask, with answers the user has confirmed. Collect candidates from:

1. `get_insights_grouped_questions` with `type: commonlyAskedQuestions` for the past 90 days: the questions that matter most.
2. The same with `type: commonlyUnansweredQuestions`, but only questions the content library answers by now.
3. The content library itself: `get_content_library_tree` and `get_content_library_page`, for important topics no question covers yet.
4. Questions the user names, such as ones that went wrong before.

Skip candidates that duplicate an existing control question, that depend on one visitor's own account, or that have no correct answer in the content.

For each candidate, write the expected answer from what the content library says: find it with `search_content_library` and read the page with `get_content_library_page`. Keep expected answers short and factual, because the grader compares the AI's answer against them, and include the facts that must be in a correct answer. Never invent an expected answer. When the content does not say, ask the user.

Answers that depend on who is asking can be tested for a specific audience by sending `audienceId`, from `get_audiences`. Pick the audience by its name and `description`.

Show the questions with their expected answers in batches of about 10 and let the user approve, edit or skip each one. Save each approved one with `upsert_quality_control_question`:

```json
{
  "websiteId": "<websiteId>",
  "body": {
    "question": "<the question as a visitor would ask it>",
    "answer": "<the expected answer>"
  }
}
```

## 4. Run the report

Every control question costs one AI call. Tell the user how many questions will run, and start the report with `generate_quality_control_report`.

Grading runs in the background. Poll `get_quality_control_reports` every 30 seconds or so until the report with the returned `reportId` has `processed` equal to `totalQuestionsInReport`. The report is missing from the list until its first question has been scored. Questions that fail to score are dropped, so treat a report whose `timestamp` has not moved for a few minutes as finished.

## 5. Explain the results

Call `get_quality_control_reports` with the `reportId` to get the result per question. Each has a `grade` out of 10, the `controlAnswer`, the `generatedAnswer` and the grader's `explanation`. A question passes at 6 or above.

Report the overall grade and pass rate, compared with the previous report when there is one. Then go through the failed questions, worst first. For each one, find out why it failed:

- **Missing content**: the AI did not answer (`questionWasUnanswered`) or answered from an unrelated source. Check with `search_content_library`.
- **Wrong or outdated content**: the content library says something different from the expected answer. Ask the user which one is right: sometimes the control question is what is outdated.
- **Conflicting content**: two pages disagree.
- **Instructions**: the answer is right but its form is wrong, for example too long or in the wrong tone. Check `customRules` with `get_ai_configuration`.
- **Grading**: the generated answer is actually correct, and the expected answer is too narrow or worded too differently. Propose rewording the control question's answer.

## 6. Fix what failed

Propose a fix per failed question and apply it after the user approves:

- **Content in a document**: when the page section's `trainingDataId` belongs to a document (`get_training_data` returns `type: faq`, with the title in `question` and the body in `answer`), update it with `upsert_document`, sending that `trainingDataId` with the full new title and body. Only trust a section's `trainingDataId` when its `sourceInherited` is false. When it is true, the ID is the page's first source, not necessarily the one that wrote the section, so confirm with the user which source holds the text.
- **Missing content**: create a document with `upsert_document`, with the question as the title and an answer that starts with the answer itself.
- **Content from a website or file**: Unless cannot change it from here. Name the page and source URL the user should fix.
- **Instructions**: update `customRules` with `update_ai_configuration`, keeping every other rule as it is.
- **Control question**: update it with `upsert_quality_control_question`. Send the report row's `qualityControlQuestionId` as `id` (not its `questionId`), with the question, the answer and any `segmentId` or `audienceId` from `get_quality_control_questions`, because the write replaces the whole question.

## 7. Wrap up

Summarize: the grade and pass rate, the fixes that were applied, and what the user still has to do. Unless retrains the AI on changed documents automatically. Offer to run the report again once that is done, to confirm the fixes.
