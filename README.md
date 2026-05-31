# ink-tui-skill

Agent skill for building terminal user interfaces with [Ink](https://github.com/vadimdemedes/ink), React, and TypeScript.

## Install

```bash
npx skills add <your-github-username>/ink-tui-skill
```

After the repository is published, replace `<your-github-username>` with your GitHub owner name.

## What It Covers

- Ink components such as `Box`, `Text`, `Newline`, `Spacer`, and `Static`
- Keyboard input with `useInput()`
- App shutdown with `useApp()`
- Focus management patterns for multi-panel TUIs
- CLI wiring with `meow`, `render()`, and `cli.tsx`
- Layout patterns for dashboards, sidebars, and inline popups
- Testing with `ink-testing-library`

## Repository Layout

```text
ink-tui-skill/
├── SKILL.md
├── README.md
├── LICENSE
└── references/
    └── ink-reference.md
```

## Usage Notes

- Keep `SKILL.md` concise so agents can discover and load it quickly.
- Put long-form guidance and examples in `references/`.
- The repository must be public for `npx skills add <owner>/ink-tui-skill` to work for other users.
- The skill should appear on [skills.sh](https://www.skills.sh/) after installs begin to accumulate.

## skills.sh Badge

Update the owner name after publishing:

```md
[![skills.sh](https://skills.sh/b/<your-github-username>/ink-tui-skill)](https://skills.sh/<your-github-username>/ink-tui-skill)
```
