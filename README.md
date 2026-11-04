# Rise Bar Skill

An AI coding assistant skill for installing, configuring, developing, and troubleshooting [Quickshell Rise Bar](https://github.com/guiestrela/quickshell-rise-review) on **Hyprland with Omarchy**.

This repository contains the skill and its supporting references. The Rise Bar application lives in the separate repository linked above.

## What the skill does

- Guides installation and updates through the Omarchy plugin manager, including fonts, CAVA, AI usage collectors, and the recovery hook.
- Helps configure and maintain both V1 and V2 bar variants, widgets, vertical layouts, panels, and integrations.
- Explains the shared QML architecture so changes preserve variant behavior and avoid duplicated implementations.
- Guides crash and hang diagnosis using process logs, journals, core dumps, and isolated probes.
- Supports code reviews, QML warning fixes, and relevant regression checks.
- Preserves user settings, saved state, local changes, and other enabled Omarchy plugins.

## Hyprland and Omarchy support

The skill targets Rise running as a plugin inside Omarchy’s existing `omarchy-shell` process on Hyprland. It includes guidance for monitor handling, bar placement, vertical geometry, and session-aware validation.

The current installation workflow requires Omarchy’s Quickshell plugin system. A standalone Hyprland installation without Omarchy needs its own integration work; this skill does not provide a standalone installer.

## Compatibility and requirements

No minimum Omarchy, Hyprland, Qt, or Quickshell version has been established by this package. The target is the integrated Omarchy plugin architecture; standalone Hyprland is unsupported by this installer. References to API documentation versions are not runtime support claims.

| Task | Required environment/tools | Optional or conditional |
| --- | --- | --- |
| Read the skill | Agent with access to Markdown and bundled references | MCP text delivery can replace local skill files |
| Run the bar | Linux, Hyprland/Wayland, Omarchy shell with Quickshell plugin support, matching Qt/QML runtime | Provider CLIs and credentials only for selected integrations |
| Install/update | Omarchy package/plugin commands, shell, network for remote packages/source, fontconfig | CAVA with PipeWire for waveform; Python 3 and user systemd for AI collectors |
| Develop/review | Actual Rise checkout and tools required by the affected code | Node.js, Python, Qt/QML tools, ImageMagick as required by the chosen tests; ShellCheck is optional in the broad runner |
| Validate desktop | Access to the user's graphical session and active installation | Headless fixtures cover only their simulated behavior |

Verified agent behavior: Copilot CLI **1.0.105** discovered, activated, and read the skill in a temporary project on 2026-11-04. Claude Code could not run the test because it was not logged in. Cursor and Gemini CLI are not installed (only tool-manager launchers are present). Their behavior and Copilot IDE integration remain unverified. This package does not establish a tested desktop version combination.

## Distribution status

Skill version: **0.1.0**. Source baseline: Rise commit [`02c54552208f8ee099cc60d0a35d65adc8f4be8e`](https://github.com/guiestrela/quickshell-rise-review/tree/02c54552208f8ee099cc60d0a35d65adc8f4be8e), inspected in a clean local checkout on 2026-11-04. See [CHANGELOG.md](CHANGELOG.md) and [installation guidance](rise-bar/references/installation.md). The source inspection covers installer effects and data flows; it is not a complete security audit. Isolated collector-installation and lifecycle/recovery/uninstall regression suites passed against that baseline. No live desktop installation was performed; see [VALIDATION.md](VALIDATION.md) for evidence and limitations.

This skill package is licensed under the [MIT License](LICENSE), copyright © 2026 Guilherme Estrela. The separate Rise application has its own licensing terms.

## Agensi / MCP delivery

Product description: a maintenance-assistance skill for Rise Bar, with guidance for preserving variants, diagnosing failures, and reviewing changes. The Rise application is obtained separately.

When this package is delivered as text through MCP, have the agent read `SKILL.md` and fetch the relevant bundled reference text through the provider. Local copying into a Codex or other agent directory applies only to filesystem installation. If references are unavailable through the provider, provide them before carrying out dependent procedures.

Reading requires access to the supplied text. Code work requires access to the actual Rise checkout; installation requires terminal and package/source network access; desktop checks require the local graphical session. Optional collectors access provider usage/account data as detailed in the installation reference. Review the platform's detected permissions against these tasks before submission. Agensi delivery and packaging acceptance have not been tested by this project.

## Install the skill

For Codex, copy the complete `rise-bar` directory into your skills directory:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R ./rise-bar "${CODEX_HOME:-$HOME/.codex}/skills/"
```

If a skill named `rise-bar` is already installed, compare or back it up before copying. Start a new Codex session after installation so the skill is discovered.

Installing this skill only adds assistant instructions. It does not install Rise Bar, change desktop settings, or restart the shell. Bar installation guidance is in [installation.md](rise-bar/references/installation.md).

## Compatible AI agents

The same `rise-bar/` package can be used by agents that support the `SKILL.md` format. Copy the **complete directory**, including `references/`, into the appropriate skills location:

| Agent | Project installation | Documentation |
| --- | --- | --- |
| Claude Code | `.claude/skills/rise-bar/` | [Claude Code skills](https://code.claude.com/docs/en/skills) |
| Cursor | `.cursor/skills/rise-bar/` | [Cursor Agent Skills](https://cursor.com/docs/skills) |
| Gemini CLI | `.gemini/skills/rise-bar/` | [Gemini CLI Agent Skills](https://geminicli.com/docs/cli/skills/) |
| GitHub Copilot | `.github/skills/rise-bar/` | [Copilot Agent Skills](https://code.visualstudio.com/docs/agent-customization/agent-skills) |

For example, from this repository, install for Claude Code in a target project:

```bash
# Replace this with the project where you want to use the skill.
project_dir=/path/to/your/project
mkdir -p "$project_dir/.claude/skills"
cp -R ./rise-bar "$project_dir/.claude/skills/"
```

For another agent, replace `.claude/skills` with its path from the table. Compare or back up any existing `rise-bar` skill before copying. Reload the agent or start a new session after installation.

`SKILL.md` and its references contain the shared instructions. `agents/openai.yaml` supplies Codex discovery metadata; other agents do not need an equivalent file to read the skill. Compatibility here describes the package format and documented installation paths; execution has not been tested in every agent.

A local GitHub Copilot CLI smoke test on November 4, 2026 confirmed project discovery from `.github/skills/rise-bar/`, activation through the `skill` tool, and reading `references/installation.md` through the `view` tool. The test used a temporary project with command execution and file editing tools disabled. It did not install Rise or validate desktop behavior. Other agents and the Copilot IDE integration remain untested.

The agent needs access to the Rise checkout and, for desktop operations, the local Omarchy session and shell tools. A remote or cloud agent can review code and documentation, but desktop validation requires access to that session. Other assistants can be asked to read `rise-bar/SKILL.md` and follow its linked references manually if they can access the files.

## Usage

Ask your assistant to use the `rise-bar` skill and describe the task. For example:

- “Use the rise-bar skill to install Rise on my Omarchy desktop.”
- “Use the rise-bar skill to investigate why my bar disappeared.”
- “Use the rise-bar skill to fix a widget shared by V1 and V2.”
- “Use the rise-bar skill to review the vertical bar layout on Hyprland.”

The assistant should locate the actual Rise checkout and active installation before making changes, then follow the relevant references and validation steps.

## Repository layout

```text
rise-bar/
├── SKILL.md                    # Main workflow and safety guidance
├── agents/
│   └── openai.yaml             # Assistant discovery metadata
└── references/
    ├── installation.md         # Installation and update guidance
    ├── vertical-bar.md         # Vertical layouts and V2 shell styles
    └── code-review.md          # QML reviews and validation practices
```

See [SKILL.md](rise-bar/SKILL.md) for the complete workflow.

## Download package

The distributable is `dist/rise-bar-skill-0.1.0.zip`, with its checksum in `dist/SHA256SUMS`. It includes the full skill, references, MIT license, README, version, changelog, and [validation report](VALIDATION.md). Extract it before following local installation instructions. Platform acceptance must still be checked on Agensi; producing this archive does not publish it.
