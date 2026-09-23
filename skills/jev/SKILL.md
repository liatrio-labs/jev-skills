---
name: jev
license: MIT
description: >
  Call TypeSafe Jev when code needs a typed classify, route, score, or decide
  step. Jev is a System One model: it returns Choice, Score, or Noul answers
  with probabilities, not generated text. Use when a prompt-and-parse JSON
  decision, a brittle keyword rule, or an "if this message is about X" branch
  should become a structured judgment. Do not use Jev to write code, chat, or
  replace the coding model. Requires TYPESAFE_API_KEY already in the
  environment. Never invent, paste, or commit an API key.
---

# Call Jev for classify, route, score, decide

Jev answers typed questions about a **state**. Your code owns the workflow.
Jev does not write replies, code, or explanations. If the task is to edit
files or talk through a design, stay in the coding agent. Call Jev only for
the judgment.

The live TypeSafe docs are the source of truth for request fields. Read the
page you need before writing a call. Start at the
[docs index](https://docs.typesafe.ai/llms.txt). Append `.md` to a docs path
when you want Markdown. For workflow design beyond a single call, also read the `typesafe-ai` skill
(`skills/typesafe-ai/SKILL.md` in this repo).

## When to call

| Job | Primitive | What you get |
| --- | --- | --- |
| Classify or route to one of a fixed set | [Choice](https://docs.typesafe.ai/primitives/choice.md) | `choice`, per-option `probabilities`, `confidence` |
| Score a degree (urgency, risk, quality) | [Score](https://docs.typesafe.ai/primitives/score.md) | `score`, level `probabilities`, `confidence` |
| Decide whether a condition holds | [Noul](https://docs.typesafe.ai/primitives/noul.md) | `noul` from 0 to 1. No separate confidence |

Ask independent questions in one request. They run in parallel and cannot see
each other's answers. Split a broad judgment into narrow questions, then
combine the numbers in code. Include a no-match option when nothing may fit.

Use the probabilities. For Choice and Score, `confidence` measures how
concentrated the distribution is, not whether the workflow is allowed to act.
A Noul near 0.5 means yes and no are similarly likely. Escalate uncertain
cases to a person or a reasoning model. Evaluate any threshold on the user's
data. Do not copy a cookbook threshold as a universal rule.

## How to call

1. Confirm `TYPESAFE_API_KEY` is set in the environment. If it is missing, stop
   and tell the user to create a key at
   [console.typesafe.ai/keys](https://console.typesafe.ai/keys) and export it.
   Do not invent a key, ask them to paste it into chat, print it, or write it
   into a file that will be committed.
2. Read the current [HTTP API](https://docs.typesafe.ai/api.md) or the SDK page
   you will use ([Python](https://docs.typesafe.ai/sdk/python.md),
   [JavaScript](https://docs.typesafe.ai/sdk/javascript.md)).
3. Send one `POST` to `https://api.typesafe.ai/v1/systemone` with
   `Authorization: Bearer $TYPESAFE_API_KEY`. Set `model` to `jev-latest`
   unless the user pinned a version. Put the text or JSON under `state`. Put
   each judgment under `questions`.
4. Branch on the typed fields. Do not parse free-form model prose. There is
   none.

Optional environment variables, from the official SDKs. Explicit client
options override them.

| Variable | Role | Default |
| --- | --- | --- |
| `TYPESAFE_API_KEY` | Required API key | none |
| `TYPESAFE_BASE_URL` | API root | `https://api.typesafe.ai` |
| `TYPESAFE_DEFAULT_MODEL` | Model when omitted | `jev-latest` |
| `TYPESAFE_LOG_LEVEL` | SDK log level | `warn` |

A Vercel AI Gateway key (`API_KEY`, model id `typesafe-ai/jev`) is a different
credential. Do not send it as `TYPESAFE_API_KEY`.

### Minimal request

```bash
curl -X POST https://api.typesafe.ai/v1/systemone \
  -H "Authorization: Bearer $TYPESAFE_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- <<'EOF'
{
  "state": "Help! My payouts have been failing for 3 days.",
  "model": "jev-latest",
  "questions": {
    "department": {
      "type": "choice",
      "instructions": "Which team should handle this?",
      "criteria": {
        "billing": "Payments, invoicing, refunds",
        "technical": "Bugs, outages, integrations",
        "other": "Does not fit billing or technical"
      }
    },
    "frustration": {
      "type": "score",
      "instructions": "How frustrated is the customer?",
      "criteria": ["Calm", "Frustrated", "Very angry"]
    },
    "is_urgent": {
      "type": "noul",
      "instructions": "The message conveys urgency or time-sensitivity"
    }
  }
}
EOF
```

The Python and JavaScript clients read `TYPESAFE_API_KEY` when you construct
`TypeSafeClient` without an api key argument. Prefer an SDK in application
code. Keep the key on the server.

## After the call

- `401`: the key is missing or rejected. Do not retry with a guessed key.
- `422`: the body failed validation. Read the error fields and the API page.
- `429` or `529`: back off and retry. The SDKs do this by default.
- Log the response `model` id when you care which version answered.
- Do not treat a typed answer as proof the content is true. Check it against
  the state and the consequence of acting.
