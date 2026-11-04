# Validation report — 0.1.0

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
