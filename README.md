# Unless plugin for Claude

[Unless](https://unless.com) is an AI platform for customer support and self-service: AI agents, a help center, a content library and insights. This plugin connects Claude to your Unless account and adds skills for common admin work.

## Skills

### Answer unanswered questions

Turns the questions your Unless AI agent could not answer into Q&A content.

1. Claude fetches the questions your AI could not answer in the past 30 days (or another period you choose), grouped by similarity and sorted by how often they were asked.
2. It filters out noise and questions an existing Q&A already covers.
3. It drafts an answer for each question from what you tell it, your existing content library, the conversations the question came from, and any other sources available in your session.
4. You review every draft and approve, edit or skip it. Nothing is saved without your approval, and Claude asks you instead of guessing when it cannot find an answer.
5. Approved pairs are saved as Q&A sources in Unless. Unless trains your AI on them and adds them to your content library automatically.

Try it by asking Claude: "Which questions could our AI not answer last month? Help me answer them."

## Setup

Install the plugin and connect the Unless connector when Claude asks. You log in with your normal Unless account, including SSO and multi-factor authentication. You never create or paste an API key.

Claude acts as you, with exactly your role and permissions in Unless. You need permission to manage AI training data to save Q&As.

## Data and privacy

The plugin only talks to the Unless MCP server at `https://mcp.unless.com/mcp`. Through it, Claude reads your workspaces, insights, training data, content library and conversation transcripts, and creates Q&A training data sources. The plugin sends no data anywhere else and stores nothing itself. See the [Unless privacy statement](https://unless.com/en/legal/visitors/privacy-statement/), the [data processing agreement](https://unless.com/en/legal/customers/data-processing-agreement/) and the [guide on connecting AI apps](https://docs.unless.com/guides/connect-ai-apps/).

## Support

Questions or problems: [support@unless.com](mailto:support@unless.com).
