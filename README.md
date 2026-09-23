# TypeSafe Agent Skills

Agent skills for building with [TypeSafe](https://typesafe.ai): typed decisions and probabilities from System One models.

You can [read SKILL.md on GitHub](https://github.com/typesafe-ai/skills/blob/main/skills/typesafe-ai/SKILL.md) or fetch its [raw Markdown](https://raw.githubusercontent.com/typesafe-ai/skills/main/skills/typesafe-ai/SKILL.md).

## Install

### Claude Code plugin

Run in your terminal:

```bash
claude plugin marketplace add typesafe-ai/skills
claude plugin install typesafe@typesafe-ai
```

### Other agents via skills.sh

```bash
npx skills add typesafe-ai/skills --skill typesafe-ai
```

Select your agent when prompted. Installation is project-local by default; add `-g` to install globally.

### Cursor

Opening this repo loads both skills through `.cursor/skills/` (symlinks to `skills/jev` and `skills/typesafe-ai`). In Agent chat, type `/jev`.

Brian's personal install target is [BriWalsh/cursor-user-skills](https://github.com/BriWalsh/cursor-user-skills): copy `skills/jev` and `skills/typesafe-ai` to that repo's root. [brwalsh `.cursor/install-skills.sh`](https://github.com/BriWalsh/brwalsh/blob/main/.cursor/install-skills.sh) rsyncs that root into `~/.cursor/skills` and `~/.agents/skills`. There is no existing Jev-to-Cursor pattern to extend. See [BRIAN-HOWTO.md](BRIAN-HOWTO.md).

For any other project:

```bash
npx skills add liatrio-labs/jev-skills --skill jev --skill typesafe-ai --agent cursor -y
```

`npx skills add` writes Cursor project skills to `.agents/skills/` and global skills to `~/.cursor/skills/`.

See the [installation guide](https://docs.typesafe.ai/agent-skill#installation) for a prompt to copy to your agent, manual installation, and updates.

## Use

Ask your agent, for example:

> Use TypeSafe to route incoming support tickets by department, with human review for uncertain decisions.

In Claude Code, you can explicitly invoke the plugin skill with `/typesafe:typesafe-ai`.

| Skill | Purpose |
|---|---|
| [jev](skills/jev/SKILL.md) | Call Jev to classify, route, score, or decide. Reads `TYPESAFE_API_KEY` from the environment. |
| [typesafe-ai](skills/typesafe-ai/SKILL.md) | Design TypeSafe workflows, find current docs and cookbooks, and compose typed judgments in code |

## License

[MIT](LICENSE).
