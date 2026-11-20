---
name: account-review
description: Review how an Unless workspace is set up and performing, and write a ranked, step-by-step plan to improve it. Use when the user asks how their AI agent is doing, what to improve next, for an account review, health check, audit or improvement plan, how to raise their AI maturity score, or which Unless features they are not using yet.
---

# Account review

Read how a workspace performs and how it is set up, then write a short plan of the improvements that would help most, each explained in terms of this workspace. This skill only reads; it changes nothing.

All tools below come from the Unless connector. If they are not available, tell the user to connect Unless (`https://mcp.unless.com/mcp`) and stop.

## 1. Pick the workspace

Call `list_workspaces`. With one workspace, use it and name it. With more than one, ask the user which one to review. When several workspaces look like they serve the same product, note it: running them separately often splits the content and the reporting.

Default to the past 30 days, compared with the 30 days before, unless the user asked for another period. The insights totals and per-day tools take `startTime` and `endTime` as `YYYY-MM-DD` days. The grouped questions and common sources tools take Unix timestamps in milliseconds.

## 2. Gather the data

Call these, and keep going when one fails or returns nothing:

- `get_insights_ai_maturity`: the overall score and the features behind it. Each feature says whether it is `implemented`, whether it can be measured, and whether the account `skipped` it. A cell's `blockingPrerequisite` explains a zero that comes from a missing deployment rather than missing work. `deeplink` points at the dashboard page for the feature.
- `get_insights_outcomes` and `get_insights_outcomes_per_day` for both periods: the conversions the AI and components contributed to, and the trend. When `status` says the numbers are not ready, mention it instead of reading them as zero.
- `get_insights_grouped_questions` for the current period: the most asked, unanswered and downvoted questions and how often each came up.
- `get_insights_common_sources`: the sources the AI relies on most. A source titled `<Source is removed>` is still cited in old answers.
- `get_ai_configuration`: languages, `customRules`, response length and privacy settings.
- `get_training_data`: the knowledge sources. Look for sources that failed or stopped scanning (`lastScanTimestamp` far in the past), websites where `indexedUrlsCount` is far below `totalUrlsCount`, and how much content is documents (`type: faq`).
- `get_procedures`: what the AI can do beyond answering, and whether procedures are enabled and have a description.
- `get_quality_control_reports`: whether answer quality is measured, and the latest grade.
- `get_tasks` with the open state: work waiting for someone in the dashboard.

## 3. Find what matters most

Look for the improvements with the biggest effect on this workspace, backed by the data:

- **Knowledge gaps**: a high share of unanswered questions, or a few unanswered questions asked very often.
- **Wrong answers**: downvoted questions, or a low or falling quality control grade.
- **Broken or stale sources**: failed scans, websites that are barely indexed, removed sources that are still cited.
- **Unused capabilities**: maturity features that are not implemented and not skipped, such as the content library, the help center, lifecycle stages, procedures or secure identification. Name the ones whose prerequisites are already in place first.
- **Configuration**: missing languages visitors use, `customRules` that contradict each other, quality control that was never set up.
- **Trends**: outcomes or usage dropping compared with the previous period.

Leave out anything the account `skipped` on purpose, unless the data shows it now hurts them.

## 4. Write the plan

Rank the improvements by expected effect against effort, and keep at most 7. For each one, write:

- **What to do**, in one sentence.
- **Why**, with the numbers from this workspace, for example "41 of the 230 questions last month went unanswered, 12 of them about invoices".
- **What it means for you**: the concrete result for this workspace, in plain language, without Unless jargon.
- **Effort**: small, medium or large.
- **How**: the dashboard `deeplink` from the maturity data, or the skill that does it in Claude:
  - unanswered questions: the `answer-unanswered-questions` skill
  - wrong or downvoted answers: the `fix-a-wrong-answer` skill
  - measuring answer quality: the `quality-control` skill
  - knowledge in old tickets: the `tickets-to-knowledge` skill
  - letting the AI act through an API: the `connect-your-api` skill
  - docs behind on product changes: the `update-docs-from-changes` skill

Open with three lines: the maturity score, the trend in outcomes, and the single most important improvement. Do not pad the plan with generic advice that the data does not support.

Offer to save the review as a markdown file, so the user can share it with their team.

## 5. Next step

Ask which improvement the user wants to start with. When a skill covers it, start that skill.
