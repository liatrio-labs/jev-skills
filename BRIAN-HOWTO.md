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

Anvil confirmed there is no existing Jev-to-Cursor pattern to extend. Your machine already has a skills home. Use that.

**Preferred target:** [BriWalsh/cursor-user-skills](https://github.com/BriWalsh/cursor-user-skills). Production skills sit one folder per skill at that repo's root (`eyes/SKILL.md`, `eod-drafter/SKILL.md`, `follow-up-radar/SKILL.md`). [BriWalsh/brwalsh](https://github.com/BriWalsh/brwalsh) `.cursor/install-skills.sh` clones that repo and rsyncs the root, excluding `.git` and `README.md`, into both `~/.cursor/skills` and `~/.agents/skills`. Cloud Agents discover skills only from disk at startup, so the folders have to be on `main` before the next agent starts. Do not run `bots/docs/promote-to-user-skills.sh` for this. That script only promotes the ops kit.

This agent's token cannot read `cursor-user-skills` (private; clone returns "repository not found"). The steps below follow the public layout in [bots/docs/install-notes.md](https://github.com/BriWalsh/brwalsh/blob/main/bots/docs/install-notes.md) and [`.cursor/install-skills.sh`](https://github.com/BriWalsh/brwalsh/blob/main/.cursor/install-skills.sh).

Run this on a machine that can read the private repo. It copies `typesafe-ai` and `jev` to the repo root, commits, and pushes `cursor/add-jev-and-typesafe-ai`. It does not put a key in the commit. After this how-to is on `main`, you can drop the `--branch` line and clone `main` instead.

```bash
git clone --branch cursor/brian-jev-howto-1e46 --depth 1 \
  https://github.com/liatrio-labs/jev-skills.git
git clone git@github.com:BriWalsh/cursor-user-skills.git
cd cursor-user-skills
git checkout -b cursor/add-jev-and-typesafe-ai
if [ -e jev ] || [ -e typesafe-ai ]; then
  echo "jev or typesafe-ai already exists. Stop and look before replacing."
  exit 1
fi
cp -R ../jev-skills/skills/typesafe-ai .
cp -R ../jev-skills/skills/jev .
test -f typesafe-ai/SKILL.md && test -f jev/SKILL.md
git add typesafe-ai jev
git commit -m "Add typesafe-ai and jev Cursor skills"
git push -u origin cursor/add-jev-and-typesafe-ai
echo "Open: https://github.com/BriWalsh/cursor-user-skills/compare/main...cursor/add-jev-and-typesafe-ai?expand=1"
```

`install-skills.sh` clones `main` with `--depth 1`, so Cloud Agents pick the skills up only after that PR is merged. The folder name must match `name` in `SKILL.md`.

On your laptop, before or after merge, install the same tree into the directories Cursor reads. From the `cursor-user-skills` clone that already contains `jev/` and `typesafe-ai/`:

   ```bash
   for dest in "$HOME/.cursor/skills" "$HOME/.agents/skills"; do
     mkdir -p "$dest"
     rsync -a --exclude '.git' --exclude 'README.md' ./ "$dest"/
   done
   find "$HOME/.cursor/skills/jev" "$HOME/.cursor/skills/typesafe-ai" -name SKILL.md
   ```

   This rsync does not pass `--delete`. The Cloud Agent script does. Run that script only from a full `main` checkout, or a partial tree will remove the ops-kit skills.

   After the folders are on `main`, `brwalsh`'s `.cursor/install-skills.sh` does this same rsync on each Cloud Agent VM. It needs a `GH_TOKEN` that can read the private repo, or the Cursor GitHub app already granted that access. The script strips the token from the clone remote after a successful authenticated clone.

Open a new Agent chat. Type `/jev`. Under Customize, then Skills, you should see `jev` and `typesafe-ai`.

Opening **this** repo also loads both skills, because `.cursor/skills/jev` and `.cursor/skills/typesafe-ai` are symlinks. That only covers work inside `jev-skills`. The user-skills repo is what your other projects and Cloud Agents use.

Other projects can still install from here without your private repo:

```bash
npx skills add liatrio-labs/jev-skills --skill jev --skill typesafe-ai --agent cursor -y
```

That writes `.agents/skills/` in the current project. `-g` writes `~/.cursor/skills/`. Until this branch merges, run `npx skills add .` from a local clone. Use one copy of `typesafe-ai`. The upstream package is `npx skills add typesafe-ai/skills --skill typesafe-ai --agent cursor -y`, and a second copy drifts.

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

6. Install the skills into [BriWalsh/cursor-user-skills](https://github.com/BriWalsh/cursor-user-skills) using the section above, then rsync them into `~/.cursor/skills`. Opening this repo is only a way to try `/jev` while you review the how-to.
7. In Agent chat, with `TYPESAFE_API_KEY` exported in the environment Cursor can see, ask:

   > Use the jev skill. Classify this note, route it to one team, and score urgency. Do not print the API key.

   The agent should call `POST https://api.typesafe.ai/v1/systemone`. If it starts designing a larger workflow, it should also follow `typesafe-ai`.

Direct API price, from the [models page](https://docs.typesafe.ai/models.md): input is `$0.042` per million tokens and output tokens are free. That is the TypeSafe API, not the Gateway promo in Path A.

### Path B blocker

If [console.typesafe.ai](https://console.typesafe.ai/) still puts `@liatrio.com` on a waitlist, you cannot create a key. Kevin's no-waitlist signup was one Liatrio email on 2026-09-21, not a guarantee for every account. Stop there and tell Innovation. Do not borrow someone else's key.

A `401` after you created a key means the shell Cursor is using does not have `TYPESAFE_API_KEY`, or the header is wrong. Export it again in that shell and open a new Agent chat. Do not put the key in `mcp.json` or in the repo.

## Which one tonight

Do Path B if you want Cursor to call Jev while you write code. Do Path A if you want Robert's inbox bot. Do both only if you have Vercel access and a TypeSafe account. Keep the two keys apart.
