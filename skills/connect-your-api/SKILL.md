---
name: connect-your-api
description: Connect a company's own API to its Unless AI agent, so the AI can look things up or act for logged-in customers, and set up the secure user identification that passes each customer's credentials along. Use when the user wants the AI to use their API, connect their backend or product, answer questions from live account data, turn an OpenAPI or Swagger spec into a procedure, make the AI agentic, or set up identifying users, an identify JWT or secure traits for Unless.
---

# Connect your API

Create or update one "Connect your API" procedure in Unless: an API reference the AI reads to compose its own requests, the hosts and methods it may use, and how each request is authenticated. When the API needs to know who the customer is, also set up secure identification in the user's code, so every request carries that customer's own short-lived credentials.

All Unless tools below come from the Unless connector. If they are not available, tell the user to connect Unless (`https://mcp.unless.com/mcp`) and stop.

Never save the procedure or change code without the user's approval. Show what you are about to write first.

## 1. Pick the workspace

Call `list_workspaces`. With one workspace, use it and name it. With more than one, ask the user which one to use. Every later call needs its `websiteId`.

Call `get_procedures` and look for procedures with `command: documented-api`. When one already exists for this API, update it: keep its `actionId` and send the whole procedure back with your changes, because the write replaces it. Ask before adding a second one for the same API.

Large responses are cut off. When the `get_procedures` response ends with `[Response truncated ...]`, never write a procedure back from it, because its `docs` may be incomplete. Rebuild `docs` from the API reference (step 2) and take the other fields from what the user confirms, or ask the user to make the change in the dashboard.

## 2. Find the API reference

Use the best source available, in this order:

1. An OpenAPI or Swagger file in the user's code or a URL to one.
2. Published API documentation the user points you to.
3. The route handlers in the user's code, when there is no reference at all. Read the routes, their parameters and their responses, and never describe an endpoint you did not find.

When you cannot find any of these, ask the user where the API is documented.

## 3. Agree on the scope

Propose, and let the user confirm or change:

- **Endpoints**: the ones a customer support assistant would reach for, such as looking up an order, an invoice, a subscription or account settings. Leave out admin, internal, billing management, webhook and health endpoints.
- **Methods**: `GET` only. Add `POST`, `PUT`, `PATCH` or `DELETE` only when the user wants the AI to change data, and name the endpoints that would allow it.
- **Hosts**: the bare hostnames the requests go to, such as `api.example.com`. At most 5, with no scheme, port, path or IP address.
- **Base path**: the prefix every path shares, such as `/v2`, or none.

## 4. Decide how requests are authenticated

Ask how the API authenticates a user, then pick the `auth` type:

- **`none`**: the API is public. Skip step 5.
- **`bearer`**: the API takes `Authorization: Bearer <token>`. The token comes from a secure trait, `authToken` by default.
- **`headers`**: the API wants one or more other headers, such as `x-api-key` and `x-customer-id`. Each header takes its value from a secure trait, or a value that is not a secret, such as an API version.

Secure traits are the `traits` of the identify JWT that the user's backend signs (step 5). Unless never stores a credential on the procedure, so a secret API key of the company itself does not belong in a trait either: the visitor's browser holds the JWT and can decode it. The token in a trait must belong to the logged-in customer, be accepted by the API, and expire within an hour. Ask whether their backend can issue such a token. When it cannot, say so plainly: the API needs a per-user token before the AI can call it safely.

## 5. Set up secure identification

Unless reads the traits from a JWT that the user's backend signs and the page passes to the Unless snippet. Fetch `https://docs.unless.com/getting-started/identifying-users.md` and follow it. It contains the full guide and the JavaScript API pages it uses.

First check what is already there. Search the user's code for `TxtOptions`, `Txt.initialize`, `Txt.setJwt`, `Txt.onJwtExpired` and a backend that signs a JWT with the Unless API key. Reuse what exists and only fill the gaps.

When the user's code is available in this session, implement it, following the code style and libraries the project already uses:

- **Backend**: an endpoint for the logged-in user that returns an HS256 JWT signed with the account's Unless API key, with `sub` set to a stable user ID, `traits` holding exactly the trait names the procedure's `auth` reads, and `exp` at most one hour ahead, in seconds. Read the API key from the project's existing secret or environment setup. It must never reach the browser, so never put it in frontend code, a client bundle or a committed file.
- **Frontend**: set `window.TxtOptions` with `customIdentifier` and `jwt` before the Unless snippet loads. When the user only logs in after the page has loaded, as in most single page apps, set `autoInitialize: false` and call `Txt.initialize({ identifier, jwt })` once the user is known, with the same ID as `sub`.
- **Refreshing**: call `Txt.setJwt(newJwt)` before the token expires, and define `Txt.onJwtExpired` to return a fresh JWT from the backend endpoint, as the fallback after a device has been asleep.

Show the full diff and let the user review it. When the code is not available, write a short handover for their developer instead: the steps above, the exact trait names, and the docs link.

## 6. Write the reference and the description

**`docs`**: write the API reference for the AI, not for a person. Use one `## METHOD /path` heading per endpoint, with paths relative to the base path. Under each heading say what the endpoint is for, its parameters, which are required and what values they accept, and add a one-line example. Never invent an endpoint, a parameter or a value. Stay under 64,000 characters. When the API does not fit, keep the endpoints a support assistant needs most.

**`description`**: the AI is only offered the procedure when it has one, and the dashboard edits it as four sections. Use exactly these headings, in this order, under 2,000 characters in total:

```markdown
### What does it do?
What the AI can do with this API, in one or two sentences.

### When is it needed?
The kinds of customer questions that should make the AI use it.

### Custom rules
Limits worth respecting, such as data it cannot reach or when to ask the customer for a value instead of guessing. Leave this section out when there are none.

### Example questions
Two or three example customer questions, one per line.
```

Write the description for an operator reading a settings page: no HTTP, endpoints, paths, parameters or JSON.

## 7. Save the procedure

Show the user the name, description, hosts, methods, base path, auth and the list of endpoints in `docs`. Ask whether to save it live, or switched off until they have tested it. Then call `upsert_procedure`:

```json
{
  "websiteId": "<websiteId>",
  "body": {
    "actionId": "<the existing actionId, or a new UUID you generate>",
    "arguments": "<the same actionId>",
    "name": "Connect your API",
    "command": "documented-api",
    "description": "<the four sections>",
    "docs": "<the reference>",
    "hosts": ["api.example.com"],
    "methods": ["GET"],
    "basePath": "/v2",
    "auth": { "type": "bearer", "secureProfileKey": "authToken" },
    "smartAction": true,
    "enabled": true,
    "verifiedCustomersOnly": true,
    "lifecycleStage": "support"
  }
}
```

- `arguments` must equal `actionId`. Generate the UUID yourself when creating, so both carry the same value.
- `smartAction` must be `true`, or the AI is never offered the procedure.
- Set `verifiedCustomersOnly` to `true` whenever `auth` is not `none`, so customers who are not identified are asked to log in instead. Also set it when `methods` allows changing data.
- Use the API's own name for `name` when there is more than one connected API.
- On an update, send back every field of the existing procedure with your changes applied.

When the save fails with a `400`, read the field named in `issues[].path`, fix it, and try again.

## 8. Wrap up

Tell the user:

- that the procedure is saved, whether it is live, and that they can review and edit it in the Unless dashboard under procedures;
- which code changes were made, or that the handover for their developer is ready;
- how to test it: log in to their product as a test customer, open the Unless AI and ask one of the example questions. A request that fails with 401 or 403 usually means a trait is missing from the JWT or has a different name than the procedure's `auth` reads.
