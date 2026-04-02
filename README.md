# xhs-cli-skill

Claude Skill for Xiaohongshu CLI operations — browse notes, search, download images, and manage interactions from terminal.

## Installation

```bash
cd ~/.claude
npx skills add -g https://github.com/Lionad-Morotar/xhs-cli-skill
```

## Usage

```sh
/xhs-cli {your command}
```

If your IDE doesn't support SlashCommands, add a prefix to your prompt:

```plaintext
使用 xhs-cli 技能，{your command}
```

This ensures the AI follows the documented patterns. Without the prefix, skill triggering may be inconsistent depending on how well your prompt matches the skill description keywords.

## Features

- Note information & metadata extraction
- Image download
- Search notes and users
- Hot rankings and trending feeds
- Favorites and collections management
- Social interactions (like, collect, follow)
- Structured output (YAML/JSON) for automation

## Prerequisites

Install xiaohongshu-cli:

```bash
uv tool install xiaohongshu-cli
```

Then login:

```bash
xhs login
```

## License

MIT

---

thanks: https://github.com/jackwener/xiaohongshu-cli
