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

## Deploying a reviewed local commit without publication

Use this path for an approved local candidate that is not yet upstream. Keep application, merge/push and version changes separate; installation approval does not authorize publication.

1. Freeze the tested source commit. Record source and installed HEAD/status and the manifest. Require the installed tracked payload to match the proposed base by bytes, symlink targets and executable status; stop on unexplained edits or untracked collisions. Inventory full permission bits separately, because Git status does not track ordinary group-write differences.
2. Inspect the current plugin watcher/reload implementation and obtain any required shell-wide reload authorization before writing. Snapshot the active variant from both its state file and IPC, relevant preference/configuration hashes, and enabled-plugin selections. Back up the complete installed directory, including `.git`, under private permissions outside the watched plugin tree.
3. Build and retain the exact patch outside the watched tree, using resolved commit IDs rather than a moving branch tip:

   ```bash
   git -C "$source" diff --binary "$base" "$candidate" -- > "$backup/applied.patch"
   git -C "$installed" apply --check "$backup/applied.patch"
   git -C "$installed" apply "$backup/applied.patch"
   ```

   Run each step only after the preceding one succeeds. A binary Git patch preserves tracked deletions and mode changes; copying only changed surviving files can leave a removed catalog installed. Do not copy the source `.git` directory or reset installed metadata to pretend an unpublished commit was fetched.
4. Read back the union of base/candidate paths. Check surviving file bytes or symlink targets, executable status, and explicit absence of deleted paths. Require unchanged installed full permission bits to match the backup; apply only intentional candidate mode changes. Classify pre-existing source/installed non-executable permission differences instead of treating them as content failures or normalizing them incidentally.
5. Validate and request the supported reload. Where the installed shell exposes the inspected rescan method, use:

   ```bash
   omarchy plugin validate "$installed"
   omarchy-shell shell rescanPlugins
   omarchy-shell shell ping
   omarchy-shell quickshell-rise.variant current
   omarchy-shell quickshell-rise.variant state
   omarchy-shell quickshell-rise.variant error
   omarchy-shell quickshell-rise.lifecycle ready
   ```

   Wait for settled state, rediscover the integrated-shell PID if it changed, inspect fresh logs, and compare preferences and enabled-plugin selections with the snapshot. Do not count rescan acceptance or repeated watcher messages as proof that the changed panel rendered.
6. Verify the exact changed control in the running panel when an authorized input route is available. Keep payload/lifecycle verification separate from visual interaction evidence; if the panel was not observed, say so instead of claiming rendered acceptance. Retain the patch and before/after evidence with the durable backup.
7. Report the source candidate and installed Git base separately. A successfully applied local payload can intentionally leave installed HEAD at the old base with precisely the expected dirty patch; call this a local deployment, not a completed upstream update. Use the following procedure only after the candidate is actually integrated upstream and an upstream update is authorized.

## Updating an installation with already-integrated local fixes

Use this sequence when local fixes have already been copied into the installation and are now included in the approved upstream revision. Payload equality alone does not synchronize the installed Git checkout.

1. Record source and installed HEAD/status separately. Inspect the installed manager script/help for its actual confirmation, fetch, fast-forward, validation and reload behavior; do not invent force flags. Resolve the exact approved incoming commit and compare version metadata according to the user's publication scope.
2. Enumerate tracked modifications, staged changes and untracked paths. Refuse unexplained or staged edits. Require every dirty/untracked path to exist in the approved incoming tree with identical bytes, symlink target and executable status. A missing incoming counterpart or different content needs a separate preservation/merge decision.
3. Back up the complete installed directory **including `.git`** outside the watched tree, under private permissions. Snapshot the current variant, scoped preference hashes and enabled-plugin list immediately before changes. Keep backup and final evidence in durable storage, not only expiring scratch.
4. Fetch the installed repository's remote HEAD and require it to equal the approved target before updating:

   ```bash
   git -C "$installed" fetch --quiet origin HEAD
   git -C "$installed" rev-parse FETCH_HEAD
   ```

   Stop on an unexpected target. Do not silently admit a moving upstream revision just because the version string is unchanged.
5. If verified integrated fixes obstruct fast-forward, preserve them explicitly. Git merge autostash does **not** include ordinary untracked files; an incoming file with the same name can still block the update. Only after all paths passed step 2 and the full backup exists:

   ```bash
   git -C "$installed" stash push --include-untracked \
     -m "Preserved local fixes already integrated upstream"
   git -C "$installed" rev-parse refs/stash
   ```

   Record that stash ID with the backup. Stashing and updating can trigger shell-wide plugin reloads, so perform this only within the authorized reload scope.
6. Run the inspected native updater. Where its help confirms `--yes` and the user already approved this exact update:

   ```bash
   omarchy plugin update io.github.guiestrela.quickshell-rise --yes
   ```

   On failure, retain backup/stash and diagnose the exact result. Never turn a failed update into an unreviewed reset or force-push.
7. **Do not automatically pop the saved stash:** its changes are already present upstream, so replay can conflict or overwrite the accepted files. Preserve it as rollback evidence and document that condition.
8. Read back installed HEAD and clean status; compare all tracked contents, symlink targets and executable bits with the approved source. Verify plugin validation, shell ping, settled current/state/error/ready, fresh reload logs, unchanged preferences and plugin list, and applicable rendered behavior. An installed detached HEAD is acceptable if it is the verified target; do not switch branches merely for appearance. Confirm remote merge, hosted CI and any requested release separately; a no-version-bump merge must not create a new tag/release.

## Rollback and source/install parity

1. Pause developers, automated deploys and pending update jobs affecting the bar. Inspect the installation independently of the checkout with `git status --short`, `git log` and `git reflog --date=iso`; choose the revision immediately before the rejected deployment, not an arbitrary parent of the latest source commit.
2. Back up the installed tree outside the watched plugin directory, including its Git metadata. Snapshot the active variant and relevant preferences separately; record hashes without printing private contents. Abort on unexplained local edits instead of overwriting them.
3. Restore only the authorized installation to that verified revision. Preserve the development checkout and rejected commits for recovery. Reload using the supported, authorized scope, then query variant/current, lifecycle readiness and fresh logs; a successful file restore alone does not establish a running rollback.
4. If asked to align the source repository with the restored installation, preserve the newer source tip on a backup branch. On a clean intended branch, restore the selected tree and commit the reversal normally rather than force-pushing history. For example, with `base` already resolved and verified:

   ```bash
   git restore --source="$base" --staged --worktree -- .
   git diff --cached --check
   git write-tree
   git rev-parse "$base^{tree}"
   ```

   Require the two tree IDs to match before committing. Validate the plugin, commit, push only when within the requested scope, and read back the exact remote branch with `git ls-remote origin refs/heads/main` before claiming publication.
5. When asked whether VS Code and the installation are identical, first identify the editor's actual checkout and current branch. Compare the union of tracked paths on disk by content hashes, symlink targets and executable bits, and report untracked files separately. Check status before and after the comparison to detect concurrent edits. Matching commit IDs do not cover dirty files; different commit IDs can have identical trees after a history-preserving reversal. Do not compare `.git` internals as application content.
6. Keep further work on a separate candidate branch from the accepted base. A local unpublished work branch may legitimately show VS Code's **Publish Branch** button when no upstream is configured; do not publish or switch branches just to remove that indicator.

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
