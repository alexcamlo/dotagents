# dotagents

Shared agents, contracts, commands, and skills for `~/.agents`.

## Layout

```text
.
├── AGENTS.md
├── agents/
├── commands/
├── contracts/
└── skills/
```

## Install

Point your local `~/.agents` entries at this repo:

```bash
ln -sfn "$PWD/AGENTS.md" ~/.agents/AGENTS.md
ln -sfn "$PWD/agents" ~/.agents/agents
ln -sfn "$PWD/commands" ~/.agents/commands
ln -sfn "$PWD/contracts" ~/.agents/contracts
ln -sfn "$PWD/skills" ~/.agents/skills
```

Your agent harness can keep reading from `~/.agents/...` while the actual content lives in this repo.

## Adding Skills

If your tooling installs skills into `~/.agents/skills`, and that path is symlinked to this repo, new skills will land here automatically.

Example:

```bash
npx skills add some-skill
git status
```

## Workflow

Edit either location:

```bash
nvim ~/.agents/skills/tdd/SKILL.md
# or
nvim ~/path/to/dotagents/skills/tdd/SKILL.md
```

Then review and commit from this repo:

```bash
git status
```

## Notes

- This repo is the source of truth for shared agent content.
- In my own setup, `~/.dotfiles` consumes this repo as a submodule and uses chezmoi to wire `~/.agents` to it.
