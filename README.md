# Unless plugin for Claude

[Unless](https://unless.com) is an AI platform for customer support and self-service: AI agents, a help center, a content library and insights. This plugin connects Claude to your Unless account and adds skills for common admin work.

## Skills

### Answer unanswered questions

Turns the questions your Unless AI agent could not answer into documents for your knowledge base.

1. Claude fetches the questions your AI could not answer in the past 30 days (or another period you choose), grouped by similarity and sorted by how often they were asked.
2. It filters out noise and questions your content library already answers.
3. It drafts an answer for each question from what you tell it, your existing content library, the conversations the question came from, and any other sources available in your session.
4. You review every draft and approve, edit or skip it. Nothing is saved without your approval, and Claude asks you instead of guessing when it cannot find an answer.
5. Approved answers are saved as documents in Unless. Unless trains your AI on them and adds them to your content library automatically.

Try it by asking Claude: "Which questions could our AI not answer last month? Help me answer them."

### Connect your API

Connects your own API to your Unless AI agent, so it can look things up or act for your logged-in customers.

1. Claude reads your API reference: an OpenAPI or Swagger file, your published API documentation, or the route code itself when there is no reference.
2. You agree on the endpoints, hosts and methods the AI may use. Only reading data is allowed unless you choose otherwise.
3. When your API needs to know who the customer is, Claude sets up secure identification in your code: a backend endpoint that signs the identify JWT with the customer's own short-lived token, and the frontend calls that pass it to Unless and keep it fresh. You review every change. Without access to your code, Claude writes the steps down for your developer instead.
4. Claude writes the API reference and description, you review them, and they are saved as one "Connect your API" procedure in Unless.

Try it by asking Claude: "Connect our API to the Unless AI so it can answer questions about a customer's orders."

### Tickets to knowledge

Turns the answers your support team already gave in old tickets into documents for your knowledge base. Claude reads tickets from the helpdesk you connected in Claude (or an export), groups the questions that keep coming back, skips what your content library already covers, and drafts anonymous answers. You approve every document before it is saved. Your tickets are never sent to Unless.

Try it by asking Claude: "Go through our solved Zendesk tickets from the last six months and turn the recurring questions into knowledge."

### Fix a wrong answer

Finds out why your AI gave a wrong answer and fixes the cause. Start from a conversation link or from the answers visitors downvoted. Claude checks the sources the AI used, names the cause (missing, outdated or conflicting content, the wrong audience, or an instruction) and proposes the smallest fix. Nothing changes without your approval.

Try it by asking Claude: "Why did our AI give this answer, and how do we fix it?" with a link to the conversation.

### Quality control

Tests your AI against control questions with known correct answers. Claude writes control questions from what visitors really ask and what your content library says, runs a quality control report, explains each failed answer and proposes a fix. Available on the Enterprise, Flex, Fixed and Plus plans.

Try it by asking Claude: "Set up quality control for our AI and run a report."

### Account review

Reviews how your workspace is set up and performing, including your AI maturity score, outcomes, unanswered and downvoted questions, sources and settings. Claude writes a ranked, step-by-step plan of the improvements that would help most, with the numbers behind each one and what it means for you. It only reads and changes nothing.

Try it by asking Claude: "Review our Unless account and tell me what to improve first."

### Update docs from product changes

Keeps your knowledge base in sync with your product. Give Claude release notes, a pull request or a range of commits. It finds the content the change makes outdated or leaves out and drafts updated and new documents for your approval. For content that comes from your website or files, it tells you exactly what to change at the source.

Try it by asking Claude: "We just released version 4.2. Which help content is now outdated?"

## Setup

Install the plugin and connect the Unless connector when Claude asks. You log in with your normal Unless account, including SSO and multi-factor authentication. You never create or paste an API key.

Claude acts as you, with exactly your role and permissions in Unless. You need permission to manage AI training data to save documents, to manage procedures to connect your API, and to manage the AI configuration and quality control for the skills that change those.

## What the Unless tools can do

The Unless MCP server gives Claude the tools below. Read tools change nothing. Tools that change something are marked as destructive in the server, and the skills ask for your approval before using them. Each tool only works with your own permissions in Unless.

**Read:** workspaces, insights, AI maturity, training data, the content library and its change log, conversations, procedures, audiences, components, tasks, the AI configuration, help center publish status and quality control questions and reports.

**Publish and go live:**

- Publish the help center, which deploys your FAQs, categories and settings to your live help center site.
- Create and update components (personalizations), which change what visitors see on your website.
- Create and update documents and training data sources. The AI uses them in its answers to your visitors right away.
- Accept suggested content library changes, which writes them to the library.
- Change the AI configuration: custom rules, language, product name, answer length, PII filtering and feedback replies.

**Change or discard existing content:**

- Update and overwrite existing documents, procedures, components, audiences, help center FAQs and categories and control questions.
- Roll a content library page back to an earlier version, or force a page to be imported again.
- Discard suggested content library changes.
- Create tasks, mark them as done or deleted, and assign them to a teammate.

The tools cannot delete documents, components, FAQs or categories. That is only possible in the Unless dashboard.

**Change access:**

- Create and update audiences. Audiences decide which visitors see a personalization and which content the AI may use for which visitor, so changing one changes who gets access to what.
- Create and update procedures. A procedure lets the AI send requests to your own API for your logged-in customers, including requests that change data when you allow methods other than reading.

**Send:**

- Assigning a task to a teammate sends them an email from Unless.
- Procedures let the AI send requests to your own API, as described above.
- A website source makes Unless crawl that website.

The tools send no other email, and never message your visitors directly.

**Pay:** none. The tools cannot buy anything, make payments or change your subscription.

## Data and privacy

The plugin only talks to the Unless MCP server at `https://mcp.unless.com/mcp`. Through it, Claude reads your workspaces, insights, AI maturity, training data, content library, conversation transcripts, procedures, audiences, tasks and quality control reports. It creates and updates documents, procedures and control questions, updates the custom rules of your AI, and starts quality control reports, always after your approval. Some skills also read tickets or code from your own session. Of that, the plugin only sends Unless the content you approve, such as an anonymous document or the API reference of a procedure, never the tickets or code themselves. The plugin sends no data anywhere else and stores nothing itself. See the [Unless privacy statement](https://unless.com/en/legal/visitors/privacy-statement/), the [data processing agreement](https://unless.com/en/legal/customers/data-processing-agreement/) and the [guide on connecting AI apps](https://docs.unless.com/guides/connect-ai-apps/).

## Support

Questions or problems: [support@unless.com](mailto:support@unless.com).
