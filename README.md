# dotclaude

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Personal Claude Code plugins for planning and working with Claude.

These plugins are built for [Claude Code](https://code.claude.com/docs) only. They rely on Claude Code-specific features, such as plugin skills and the `AskUserQuestion` tool, and are not designed for other agents or for the Claude apps.

## Plugins

| Plugin | Skill / command | What it does |
| --- | --- | --- |
| `workflow` | `/workflow:ask-first` | Asks focused clarifying questions with a recommended answer for each, then drafts a plan for approval before any code is written. |

## Requirements

- [Claude Code](https://code.claude.com/docs/en/setup) with plugin support. The documentation does not state a single minimum version for plugins, so keep Claude Code current with `claude update`.
- `git`, which Claude Code uses to clone the marketplace from GitHub.

## Installation

### From GitHub

Run these inside a Claude Code session:

```text
/plugin marketplace add zhoulinhua0-star/dotclaude
/plugin install workflow@dotclaude
```

In a session, `/plugin install` opens the plugin's details. Choose an install scope there to finish. To install from your shell instead:

```bash
claude plugin marketplace add zhoulinhua0-star/dotclaude
claude plugin install workflow@dotclaude
```

### From a local clone (for development)

```bash
git clone https://github.com/zhoulinhua0-star/dotclaude.git
cd dotclaude
claude plugin marketplace add ./
claude plugin install workflow@dotclaude
```

A marketplace added from a local path loads its plugins in place, so your edits take effect at the next session start, or immediately after running `/reload-plugins` in a session.

To try a plugin for a single session without installing it:

```bash
claude --plugin-dir ./plugins/workflow
```

### Updating

Plugins in this marketplace don't set a `version`, so every commit pushed to `main` is a new release. Auto-update is off by default for third-party marketplaces. Turn it on once per machine: run `/plugin`, open the **Marketplaces** tab, select `dotclaude`, and choose **Enable auto-update**.

With auto-update on, Claude Code fetches new commits in the background after each session starts. The running session keeps the copy it loaded and shows `Plugin updated`. Run `/reload-plugins` to apply the update, or start a new session.

To update manually instead:

```bash
claude plugin marketplace update dotclaude
claude plugin update workflow@dotclaude
```

### Uninstalling

```bash
claude plugin uninstall workflow@dotclaude
```

To remove the marketplace and every plugin installed from it:

```bash
claude plugin marketplace remove dotclaude
```

## Usage

Invoke `ask-first` with a description of the task:

```text
/workflow:ask-first Add a "What's new" panel that shows release notes after an update
```

If no other installed skill is named `ask-first`, the short form `/ask-first` also works.

What happens next:

1. Claude reads the code and docs relevant to the task, such as the existing onboarding tips, the settings model and how versions are tracked.
2. A question dialog opens. Each question has 2 to 4 options, and the recommended one is listed first with its reasoning, for example: "Show once per version (Recommended): matches the existing one-time tip in `Onboarding.tsx`".
3. You pick an option for each question, type your own answer, or accept the recommendations.
4. Claude presents a short plan with your choices, the defaults it applied, the files it will change and how it will verify the result. Then it waits.
5. Nothing is implemented until you approve the plan.

The skill runs only when you invoke it. Claude never starts it on its own.

## How `ask-first` works

- **Research first.** Claude reads the relevant code before asking, so it doesn't ask what the code already answers. When a question depends on the current state, it cites what it found.
- **Only questions that matter.** It asks only about decisions that change the plan. Anything a sensible default covers is decided for you and listed in the plan, where you can still override it.
- **As many questions as the task needs.** There is no fixed count. Claude asks every question that would change the plan and nothing else. Each dialog holds up to 4 questions, and Claude fills each one before opening the next, so you see as few dialogs as possible. Only questions that depend on your earlier answers wait for a later dialog. If the questions keep multiplying, Claude suggests narrowing or splitting the task instead.
- **Recommendations with reasons.** Every question comes with a clear recommendation and the trade-off of each option. If you choose differently, Claude follows your choice without arguing.
- **Scope asked separately.** If the scope is unclear, or the task touches something you said not to change, scope gets its own question.
- **Text fallback.** Where the dialog isn't available, such as with `claude -p` or inside a subagent, questions are asked in text with lettered options and a suggestion for each. You can reply with letters only, or say "take your suggestions for the rest".
- **Your language.** Questions and the plan are written in the language you use.
- **Preview for UI work.** For interface changes, Claude builds an interactive preview of the options before finalizing the plan.
- **Approval before code.** Claude writes no code until you approve the plan. In plan mode, it submits the plan through the plan approval dialog.

## Repository layout

```text
dotclaude/
├── .claude-plugin/
│   └── marketplace.json        # marketplace "dotclaude", lists each plugin
├── plugins/
│   └── workflow/
│       ├── .claude-plugin/
│       │   └── plugin.json     # plugin manifest: name, description, author, license
│       └── skills/
│           └── ask-first/
│               └── SKILL.md    # the ask-first skill
├── README.md
├── LICENSE
└── .gitignore
```

## Adding a new skill or plugin

### Add a skill to an existing plugin

1. Create `plugins/<plugin>/skills/<skill-name>/SKILL.md`. The directory name becomes the command, `/<plugin>:<skill-name>`.
2. Start the file with YAML frontmatter. Write a `description` that says what the skill does and when to use it. Add `disable-model-invocation: true` if the skill should run only when you type its command. See the [skills documentation](https://code.claude.com/docs/en/skills) for every field.
3. Test it locally, then commit and push (see [Versioning](#versioning)).

### Add a new plugin

1. Create the plugin directory with a manifest and at least one component:

   ```text
   plugins/<plugin>/
   ├── .claude-plugin/
   │   └── plugin.json
   └── skills/
       └── <skill-name>/
           └── SKILL.md
   ```

2. Write `plugin.json`. Copy `plugins/workflow/.claude-plugin/plugin.json` and change `name`, `description` and `keywords`. Don't add a `version` field (see [Versioning](#versioning)).
3. Register the plugin in `.claude-plugin/marketplace.json` by adding an entry to `plugins`:

   ```json
   {
     "name": "<plugin>",
     "source": "./plugins/<plugin>",
     "description": "One line shown in the /plugin browser."
   }
   ```

   The entry `name` must match the `name` in `plugin.json`, and `source` must start with `./` and be relative to the repository root. Don't set `version` in the entry either.

4. Add a row to the [Plugins](#plugins) table in this README.

### Versioning

Plugins here deliberately omit `version` in both `plugin.json` and `marketplace.json`. Claude Code then uses the git commit as the version, so every push to `main` reaches machines that installed from GitHub, with no version bump to forget. This has two consequences:

- **Push only finished work.** Every commit on `main` is picked up by the next update, including unfinished ones.
- **Validate without `--strict`.** `claude plugin validate` warns when `version` is missing, and `--strict` turns that warning into a failure.

If a plugin is ever shared more widely and needs controlled releases, add `version` to its `plugin.json` and bump it on every release. Users then stay on their cached copy until the string changes.

Local development is unaffected either way, because a marketplace added from a local path always loads the current files.

### Test locally before pushing

```bash
# Validate the marketplace and every plugin it lists
claude plugin validate .

# Register this clone as a marketplace (one time) and install the plugin
claude plugin marketplace add ./
claude plugin install <plugin>@dotclaude

# Check that the plugin loaded and lists the new skill
claude plugin details <plugin>
```

Then start `claude`, run `/reload-plugins` if a session is already open, and invoke the new skill once. To exercise the plugin without touching your installed setup, use `claude --plugin-dir ./plugins/<plugin>` instead.

## License

Released under the [MIT License](LICENSE).
