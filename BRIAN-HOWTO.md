# Brian: use Jev from Cursor tonight

Two separate ways to use Jev. They do not share a key.

| Path | What you get | Credential |
| --- | --- | --- |
| A. Grok bot | Robert Kelly's inbox-triage bot, calling Jev through Vercel AI Gateway | A key the Vercel connector creates on your Vercel account |
| B. TypeSafe console | Jev inside Cursor and in your own code | `TYPESAFE_API_KEY` from [console.typesafe.ai/keys](https://console.typesafe.ai/keys) |

Jev is not a chat or coding model. It takes a state plus typed questions and returns a Choice, a Score, or a Noul (a 0–1 yes probability). Cursor stays the coding agent. The skill in this repo tells Cursor when to call Jev.

Nothing in this repo contains an API key. Do not paste a key into chat, Slack, or a commit.

## What this repo is

[liatrio-labs/jev-skills](https://github.com/liatrio-labs/jev-skills) is agent skills for the TypeSafe System One API. It is not an application and it does not call Jev by itself.

| Path | Role |
| --- | --- |
| [skills/typesafe-ai/SKILL.md](skills/typesafe-ai/SKILL.md) | Design skill (TypeSafe AI, v0.5.7). How to shape questions, read the live docs, and compose judgments. |
| [skills/jev/SKILL.md](skills/jev/SKILL.md) | Cursor call skill. When to classify, route, score, or decide, and how to send the HTTP request. |
| `.cursor/skills/jev` | Symlink to `skills/jev`. Cursor loads this when the repo is open. |
| `.cursor/skills/typesafe-ai` | Symlink to `skills/typesafe-ai`. Same auto-load. |
| [.env.example](.env.example) | Names the variables. Values stay empty. |

### Environment variables

This repository requires none. A live call needs a key you create yourself.

| Variable | Required | Where it is defined | Default |
| --- | --- | --- | --- |
| `TYPESAFE_API_KEY` | Yes, for Path B calls | [Python SDK constants](https://docs.typesafe.ai/sdk/python/api/constants.md) and [JavaScript `ENV`](https://docs.typesafe.ai/sdk/javascript/api/variables/ENV.md) | none |
| `TYPESAFE_BASE_URL` | No | same | `https://api.typesafe.ai` |
| `TYPESAFE_DEFAULT_MODEL` | No | same | `jev-latest` |
| `TYPESAFE_LOG_LEVEL` | No | same | `warn` |

Path A does not use these names. Robert's bot uses the Vercel connector's key. The sibling demo [liatrio/jev-demo-jburns24](https://github.com/liatrio/jev-demo-jburns24) also uses a Gateway key in `API_KEY` and the model id `typesafe-ai/jev`. That is not `TYPESAFE_API_KEY`.

### Install the skills into Cursor

Cursor discovers skills in `.cursor/skills/` and `.agents/skills/` inside a project, and in `~/.cursor/skills/` for you personally. It does not auto-load a top-level `skills/` directory. This repo symlinks both skills into `.cursor/skills/` so opening the repo is enough.

After this branch is on your machine:

1. Clone or pull, then open the folder in Cursor.
2. Open a new Agent chat.
3. Type `/jev` or ask it to classify, route, score, or decide with Jev.
4. Confirm both skills under Customize, then Skills. You should see `jev` and `typesafe-ai`.

To install them into a different project, from that project:

```bash
npx skills add liatrio-labs/jev-skills --skill jev --skill typesafe-ai --agent cursor -y
```

That writes `.agents/skills/`, which Cursor loads for that project. For every project on your machine:

```bash
npx skills add liatrio-labs/jev-skills --skill jev --skill typesafe-ai --agent cursor -g -y
```

Global installs go to `~/.cursor/skills/`. Until this branch is merged, run the same commands with `npx skills add .` from a local clone instead of `liatrio-labs/jev-skills`.

The upstream TypeSafe repo is the same design skill, without the Liatrio how-to:

```bash
npx skills add typesafe-ai/skills --skill typesafe-ai --agent cursor -y
```

Use one install of `typesafe-ai`. Two copies drift.

Manual copy, if you would rather not run the installer: copy `skills/jev` and `skills/typesafe-ai` into `~/.cursor/skills/`. Each folder must contain `SKILL.md`, and the folder name must match the `name` in the frontmatter (`jev`, `typesafe-ai`).

## Path A — Grok bot and the Vercel connector

This is the personal inbox path. Robert Kelly posted it in [#discuss-grok-bot](https://liatrio.slack.com/archives/C0BQV8H8DM5/p1790026581746899) on 2026-09-21:

> I refactored a template for Jev to default to Vercel AI Gateway - still free this week.

He uses that bot to improve inbox triage. Adding his template does not give you his Vercel login or his key.

1. Open the [Grok Bot](https://docs.x.ai/grok-bot/bots) app and sign in with the account that should own the bot.
2. Open Robert's template: [https://x.ai/bot/Sr5InZGTkNbcDys2XcG4l](https://x.ai/bot/Sr5InZGTkNbcDys2XcG4l).
3. Click **Add to Grok Bot**. That creates a copy on your account. It does not copy his computer, logins, conversation history, or API keys. The page says the bot was created by a third party; adding it accepts those terms. You need the Grok Bot app to finish.
4. Add the Vercel connector and have it create a key. That is Robert's step, and it requires Vercel access. His screenshots are the two replies in that Slack thread. The general plugin flow is: in Grok Bot, open Plugins in the sidebar, search for Vercel, add it, and finish Authorize in the browser. Confirm Vercel shows under Installed. See [Connect plugins](https://cursor.com/help/grok-bot/connect-plugins.md).
5. Start a conversation with the copied bot and point it at inbox triage, which is what Robert is doing with it.
6. Check whether Gateway calls are still free before you rely on them. "Still free this week" was Robert's note on 2026-09-21. Tonight is 2026-09-23. The connector or the Vercel dashboard is the place to confirm. Do not assume the promo is still on.

Gregg Coppen confirmed on 2026-09-22 that Jev is available on the Vercel AI Gateway. That supports this path. It does not replace the connector step.

### Path A blocker

You need a Vercel account that is allowed to create the connector's key. If you are not on the Liatrio Vercel team, stop and ask whoever admins that team. If the Vercel plugin says **Disabled by team admin**, a Cursor team admin has to enable that marketplace plugin. That is separate from Vercel org access.

## Path B — TypeSafe console, API key, Cursor

This is the API and code path. Josh Guice's TekTok in [#liatrio-innovation](https://liatrio.slack.com/archives/C097EC3N4SC/p1790032341432719) points at the docs and the console. Kevin Jaris replied on 2026-09-21 that he signed up with his Liatrio email and there was no waitlist. Josh said that looked like it had changed on 2026-09-20.

1. Read [System One](https://docs.typesafe.ai/concepts/system-one). Ten minutes is enough to see Choice, Score, and Noul. The launch post is [Introducing System One Models & Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev).
2. Open [https://console.typesafe.ai/](https://console.typesafe.ai/) and sign up or sign in with your `@liatrio.com` email.
3. If the console lets you in, open the [Playground](https://console.typesafe.ai/playground) and run one question before writing code.
4. Create a key at [https://console.typesafe.ai/keys](https://console.typesafe.ai/keys). Store it only in your shell or in a gitignored `.env`. This repo's [.env.example](.env.example) shows the variable name and nothing else.

   ```bash
   export TYPESAFE_API_KEY='the key from the console'
   ```

5. Smoke-test the API. This is the official quickstart shape. It reads the variable and does not embed a key:

   ```bash
   curl -sS -X POST https://api.typesafe.ai/v1/systemone \
     -H "Authorization: Bearer $TYPESAFE_API_KEY" \
     -H "Content-Type: application/json" \
     -d @- <<'EOF'
   {
     "state": "Hi, I've been trying to connect my Stripe account for 3 days and the integration keeps failing. I'm losing sales. Please help ASAP.",
     "model": "jev-latest",
     "questions": {
       "urgency": {
         "type": "noul",
         "instructions": "Does this message express urgency?"
       }
     }
   }
   EOF
   ```

   A good response has `answers.urgency.noul` between 0 and 1, and a `model` id such as `jev-1.13.0`. `401` means the key was missing or rejected.

6. Install the skills using the section above. Open this repo in Cursor, or run `npx skills add` against it.
7. In Agent chat, with `TYPESAFE_API_KEY` exported in the environment Cursor can see, ask:

   > Use the jev skill. Classify this note, route it to one team, and score urgency. Do not print the API key.

   The agent should call `POST https://api.typesafe.ai/v1/systemone`. If it starts designing a larger workflow, it should also follow `typesafe-ai`.

Direct API price, from the [models page](https://docs.typesafe.ai/models.md): input is `$0.042` per million tokens and output tokens are free. That is the TypeSafe API, not the Gateway promo in Path A.

### Path B blocker

If [console.typesafe.ai](https://console.typesafe.ai/) still puts `@liatrio.com` on a waitlist, you cannot create a key. Kevin's no-waitlist signup was one Liatrio email on 2026-09-21, not a guarantee for every account. Stop there and tell Innovation. Do not borrow someone else's key.

A `401` after you created a key means the shell Cursor is using does not have `TYPESAFE_API_KEY`, or the header is wrong. Export it again in that shell and open a new Agent chat. Do not put the key in `mcp.json` or in the repo.

## Which one tonight

Do Path B if you want Cursor to call Jev while you write code. Do Path A if you want Robert's inbox bot. Do both only if you have Vercel access and a TypeSafe account. Keep the two keys apart.
