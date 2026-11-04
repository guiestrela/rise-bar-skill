# Install Quickshell Rise

This installs the bar, not the AI agent skill. Inspect existing local changes and enabled plugins before choosing installation or update. Basic installation excludes AI collectors and the recovery hook; add them only when they match the requested scope.

## Source reference and inspection

Documentation baseline: `guiestrela/quickshell-rise-review` commit `02c54552208f8ee099cc60d0a35d65adc8f4be8e`, inspected locally on 2026-11-04. This identifies the source inspected, not a security certification or a tested minimum version. The remote repository and its installers are outside this package.

Before execution, inspect the intended checkout's manifest, `scripts/install-ai-backends`, all four `scripts/*-usage` collectors, their `systemd/*-usage.{service,timer}` units, and `contrib/post-boot.d/quickshell-rise`. Compare against the baseline or document the chosen revision and differences. Use `git rev-parse HEAD` and `git status --short` to record the actual revision and modifications. Do not print credentials during inspection.

The manager command below follows the remote repository; it is **not pinned** to the baseline. Check the installed revision before running optional scripts. For a strictly pinned installation, first confirm the installed manager's supported local-source/revision workflow; do not invent a commit flag or silently treat the moving remote as pinned.

## Basic installation

Requires Omarchy with Quickshell plugin support and a working Hyprland/Wayland session. Keep either JetBrains Mono Nerd font variant; they conflict if installed together.

```bash
(pacman -Q ttf-jetbrains-mono-nerd-basic >/dev/null 2>&1 || \
  pacman -Q ttf-jetbrains-mono-nerd >/dev/null 2>&1 || \
  omarchy pkg add ttf-jetbrains-mono-nerd) && \
omarchy pkg add ttf-material-symbols-variable && \
fc-cache -f && \
omarchy plugin add https://github.com/guiestrela/quickshell-rise-review.git --enable
```

Confirm the repository when the manager asks. The manager does not run legacy `install.sh`. Material Symbols prevents icon names appearing as overlapping text. For the optional media waveform, install `cava` with PipeWire support using `omarchy pkg add cava`.

After the authorized installation, restart and validate:

```bash
omarchy restart shell
omarchy plugin validate "$HOME/.config/omarchy/plugins/io.github.guiestrela.quickshell-rise"
omarchy plugin list
omarchy-shell shell ping
```

Compare the enabled plugin list and fresh logs with the pre-install state. Desktop interaction remains a separate check.

## Optional AI usage collectors

The inspected installer requires Python 3 and a working user systemd manager. It installs all four collectors into `~/.local/bin/` and service/timer units into `~/.config/systemd/user/`, then enables their timers (approximately every five minutes). It also disables and removes legacy `claude-usage-cookie` and `claude-usage-calc` units and binaries. Back up any existing files that it would replace before running it.

| Collector | Data accessed | External access | Default output |
| --- | --- | --- | --- |
| Claude | `~/.claude/.credentials.json` OAuth token and project JSONL usage | Authenticated `https://api.anthropic.com/api/oauth/usage` | `~/.cache/claude-usage.json` |
| Codex | Local CLI app-server rate limits and session rollouts under `CODEX_HOME` | The authenticated CLI may contact its provider; the collector has no direct HTTP endpoint | `~/.cache/codex-usage.json` and `codex-usage-activity.json` |
| OpenCode | Local `opencode.db` SQLite messages | No network calls in the inspected collector | `~/.cache/opencode-usage.json` |
| Copilot | `GH_TOKEN`/`GITHUB_TOKEN` or token obtained from `gh auth token` | Authenticated `https://api.github.com/copilot_internal/user` | `~/.cache/copilot-usage.json` |

Caches contain usage/account metadata and should be treated as private. Some paths can be overridden by environment variables; inspect the installed source and service environment. Provider login is separate. Never request, print, or copy tokens into reports. Do not run authenticated collectors merely to test this skill.

After source inspection and selection of these integrations:

```bash
rise_plugin_dir="$HOME/.config/omarchy/plugins/io.github.guiestrela.quickshell-rise"
"$rise_plugin_dir/scripts/install-ai-backends" "$rise_plugin_dir"
```

Consult the checkout's `docs/integrations.md` for provider dependencies and authentication.

## Optional recovery hook

The inspected hook schedules a transient user systemd job after an Omarchy update, waits for the update lock, and can restart the integrated shell. It also contains legacy standalone bar management paths; inspect those before use in another installation mode. Installing the hook copies it into `~/.config/omarchy/hooks/post-update.d/quickshell-rise` and can replace an existing file of that name. Preserve that file first if present.

```bash
omarchy hook install post-update \
  "$HOME/.config/omarchy/plugins/io.github.guiestrela.quickshell-rise/contrib/post-boot.d/quickshell-rise"
```

## Disable, remove, or recover from partial installation

`&&` stops later steps after failure; it does not roll back completed steps. Record which packages, files, units, hook, and plugin selection existed beforehand. On failure, inspect the last successful step and restore only changes made by this installation. The collector installer can leave installed files or some timers enabled if a later systemd operation fails.

To stop the optional collectors without deleting their data:

```bash
for provider in claude opencode codex copilot; do
  systemctl --user disable --now "$provider-usage.timer"
  systemctl --user stop "$provider-usage.service"
done
```

For complete collector removal, first confirm that the installed units and binaries belong to this installation. Remove only its four named binaries from `~/.local/bin/` and corresponding `.service` and `.timer` files from `~/.config/systemd/user/`, then run `systemctl --user daemon-reload`. Restore pre-existing copies and previous enabled states from backup rather than deleting them. Preserve caches and credentials unless data deletion was requested. Restore migrated legacy Claude files from backup if rolling back that migration.

To remove the optional hook, compare `~/.config/omarchy/hooks/post-update.d/quickshell-rise` with the installed source, then remove only that hook or restore its previous copy. Removal prevents future scheduling; inspect any running `qsrise-post-update-*` user unit before stopping that specific job.

To disable the bar, use `omarchy plugin disable io.github.guiestrela.quickshell-rise`; to remove it, inspect local changes and use `omarchy plugin remove io.github.guiestrela.quickshell-rise`. Restore the previously selected bar through the manager, restart the shell if required, and compare enabled plugins. Do not bulk-disable plugins or delete saved preferences. Keep shared fonts and CAVA unless they were newly installed solely for Rise and removal is requested.
