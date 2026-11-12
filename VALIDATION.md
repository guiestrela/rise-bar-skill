# Validation report — 0.1.3

Date: 2026-11-12. Inspected and deployed Rise candidate:
`323bbac76af12b190b18248aa5093d7422dfcc3c`.

## Verified implementation and installation

- The checkout broad suite passed 75 check groups before ShellCheck was installed;
  the supplemental ShellCheck run then passed for 41 shell files.
- The final broad suite was rerun from the installed tree: **76 check groups
  passed, no skipped groups**, including ShellCheck 0.11.0. It ended with
  `ALL PLUGIN CHECKS PASSED` and exit 0.
- QML lint analyzed 254 files and reported **135 warnings** in that final suite.
  These were not suppressed or represented as zero warnings. Not every warning
  has been eliminated or established as a tooling false positive.
- Focused service, process, IPC and content probes passed. The composed V1/V2
  adapter fixture passed 13 checks and produced 11 captures covering horizontal,
  vertical, search, greetings, expanded globe, opening spin and moon states.
- The native unmodified World Clock host compiled with the actual Omarchy
  translation module, and the real English notification catalog key resolved.
  The native V1/V2 import-graph and inert display-controller integration passed.
- The installed and checkout commits matched. All 726 tracked paths were compared
  for actual content/link target and executable mode; both Git worktrees were
  clean after deployment.
- The live session remained V2/active/ready, with an empty variant error and a
  responsive shell ping. World Clock IPC returned three rows; its real V2 panel
  opened, rendered the city list and globe, and closed successfully.
- Five scoped preference/state files retained their pre-deployment hashes,
  including the active variant, both widget settings, shell configuration and
  Elsewhen preferences. Deployment used a backup outside the watched tree.
- Fresh inspected logs contained no Rise/Elsewhen ReferenceError, TypeError or
  component-unavailable failures after deployment.

## Scope and remaining limits

Clock edits, timezone mutation, profile/weather failures, keyboard/pointer and
persistence paths were exercised with deterministic/inert backends where
applicable. The live check did not change the OS timezone, user accounts,
monitor layout, cities or saved preferences. It does not certify physical
Bluetooth connections, OneDrive account sync, every live V1/vertical gesture,
all fonts/locales, or absence of future defects. The unrelated USB Bluetooth
controller failure is not claimed fixed by this World Clock release.

Skill-package verification covers YAML metadata, local links, fenced shell
syntax, distribution CRC/safe paths/content equality and SHA-256. No new
independent behavior evaluation was run in Copilot, Claude, Cursor, Gemini or
an MCP marketplace; the historical limitations below remain applicable.

# Historical validation — 0.1.2

Date: 2026-11-04. Documentation reflects local Rise HEAD
`d147987a87de167f761c1467bbfeeb3ec57f3589` plus uncommitted OneDrive and slot
changes inspected in this session. It does not assert that these features
already exist in that Git commit or the published upstream release.

Package checks for 0.1.2: Skill Creator validation, local Markdown links,
fenced Bash syntax, archive CRC and safe paths, extracted-file byte comparison,
extracted skill validation, and SHA-256 checksums. No code examples were
executed against accounts as part of the skill-package validation.

Current live validation from this session:

- The Rise checkout suite `bash tests/run-plugin-checks.sh` passed completely,
  including QML lint (235 files, 130 tooling warnings).
- The same suite passed from the active installation at
  `~/.config/omarchy/plugins/io.github.guiestrela.quickshell-rise`, whose
  installed revision was `607e5a00125a67bef278268a8b9c04eb4d1e05d8` with local
  modifications.
- `omarchy plugin list` showed Rise enabled, `omarchy-shell shell ping` returned
  `ok`, and the active shell log contained no Rise runtime errors. It did
  contain the known duplicate `omarchy.bar` handler and scanner path warnings;
  these are documented as host/tooling warnings, not bar failures.

Implementation evidence from the preceding work (not new skill behavior tests):

- The full Rise suite passed for the slot expansion. Focused V1 mouse-drag
  fixtures also exercised scaled hit testing, empty targets and collapse on lock;
  installed focused slot tests passed after deployment.
- Later OneDrive visual/preference changes received the production panel/IPC
  fixture, lint and import-graph compilation as applicable. The final height
  change received the panel/IPC fixture and rendered inspection. These focused
  reruns do not claim a new full-suite run after every visual edit.
- The installed shell was validated/restarted and fresh logs inspected after
  applying changes. The latest inspected restart responded to ping and had no
  errors in the inspected log tail. Saved layout/plugin state was compared for
  the slot deployment; this does not prove all future live interactions.
- During separately authorized troubleshooting, the OneDrive service recovered,
  a sync completion was observed, and quota/full-status checks succeeded after
  raising the wrapper limit from 30 to 90 seconds. A direct quota call took
  32.7 seconds. This is a local recovery observation, not a general cloud-sync
  certification or authorization for future account actions.

The updated skill has not received new independent agent behavior evaluation.
Earlier Copilot and external-platform limitations below remain historical;
Claude/Cursor/Gemini/MCP acceptance is not newly established by this release.

# Historical validation — 0.1.1

Date: 2026-11-04. This release adds instructions based on the local Rise checkout
at HEAD `d147987a87de167f761c1467bbfeeb3ec57f3589` **with uncommitted OneDrive
integration changes**. These paths and behaviors are not asserted to exist in
that commit alone or in the public upstream release.

- Skill metadata, local Markdown links, fenced Bash syntax and archive integrity
  were checked for this package revision; see the update's validation output.
- Source-session evidence: the Rise broad suite passed after initial OneDrive
  integration. Later visual edits received focused panel/IPC runtime checks,
  lint and import-graph compilation as applicable. The installed shell restarted
  and responded to ping without integration errors in the inspected fresh logs.
- These are implementation/session checks, not an independent behavioral test
  of the updated skill. The new reference has not been re-evaluated in Copilot,
  Claude, Cursor, Gemini or an MCP distribution platform.
- No new real-account quota, resync, login, pause or synchronization test was run
  to validate this skill update. End-to-end OneDrive sync remains unverified.

The agent compatibility and external-platform limitations recorded below still
apply. Historical 0.1.0 results are retained separately and are not new runs.

# Historical validation — 0.1.0

Date: 2026-11-04. Skill tests used temporary projects. Rise baseline: `02c54552208f8ee099cc60d0a35d65adc8f4be8e` (clean local checkout).

| Check | Result | Scope |
| --- | --- | --- |
| Skill Creator `quick_validate.py rise-bar` | PASS | PyYAML supplied with `uv run --with pyyaml` in a temporary cache; no system package change |
| Local Markdown links and Bash syntax | PASS | Relative links resolved; all fenced Bash examples parsed with `bash -n`, not executed |
| Copilot CLI 1.0.105 | PASS | Project skill discovery, activation, and `view` reading of revised installation reference; correct baseline, optional collectors, and MCP handling; write/shell tools unavailable |
| Rise `tests/install-ai-backends-integration.sh` | PASS | Mock Omarchy/systemd, temporary home; installation, repeat installation, missing systemd and timer-enable failure; no provider authentication |
| Rise `tests/qs-quattro-runtime-regression.sh` | PASS | Isolated lifecycle, ownership, fail-open, registry, uninstall, legacy and generic regressions |
| Distribution archive | PASS | Extraction, CRC, contents, safe member paths, byte-for-byte comparison, extracted skill validation and SHA-256 checked locally |
| Claude Code 2.1.318 | BLOCKED | Read-only invocation returned “Not logged in”; requires user login before behavioral validation |
| Cursor / Gemini CLI | NOT RUN | Only mise launchers present, no installed runtime; resolution attempted downloads and failed due network/DNS/timeout |
| Copilot IDE / Agensi MCP | NOT RUN | No connected IDE test session or Agensi connector available |
| Live desktop installation/removal | NOT RUN | Mock tests do not establish behavior in the user's graphical session or supported version combinations |

The Copilot run used only `skill` and `view` tools. No provider usage collector was executed against a real account. Real shell services, monitors, credentials, and plugin selection were not changed by these tests.

## Remaining external validation

- Authenticate Claude Code locally and repeat activation/reference reading in a temporary project.
- Install/authenticate Cursor and Gemini CLI, then repeat read-only skill activation using their documented project paths.
- Open a temporary VS Code project containing `.github/skills/rise-bar/` and confirm Copilot activation/reference reading.
- Upload the archive to an Agensi draft and verify actual resource names, reference delivery, permission declarations and packaging acceptance before publishing. No platform limits or acceptance are asserted here.
- On a disposable Omarchy/Hyprland test session, record OS, Omarchy, Hyprland, Quickshell and Qt versions; test basic installation, optional components, rollback and removal while comparing plugin state and logs. Do not infer live desktop support from mocked tests.
