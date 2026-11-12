# Widget integration and OneDrive

## Inspect and preserve ownership

Read the source plugin's manifest, model, service, panel, helper scripts and
license before adapting it. Inventory its account controls, keyboard/mouse
shortcuts, notifications, preferences, state paths and external dependencies.
A replacement may have different semantics: Cloud/rclone mounts remote folders;
OmaOneDrive monitors the abraunegg OneDrive client's local sync and systemd units.
Do not silently substitute one workflow for the other. If replacement versus
coexistence is unclear, resolve it before removing the previous widget.

Keep backend state, scheduling, commands and panel content in `shared/`.
V1/V2 should register the widget and supply only their real presentation
adapters. Create one coordinator in the active Theme, not one per BarSlot or
monitor. Gate routine work on the widget/panel being enabled. Unloading a variant
must release its timers, processes and IPC registration.

Preserve the source license and attribution for imported code, including any
host UI components adapted to make the plugin self-contained. Resolve helper
paths relative to the shared integration directory. Do not add account setup,
package installation or service mutations as startup side effects.

## Slots, state and panel routing

Inspect both variants before assigning IDs: most existing G IDs differ between
V1 and V2. In the inspected local implementation, G22 is OneDrive in both
variants and replaces the former Cloud slot. Reusing that slot preserves its
saved order and visibility; it does not migrate rclone data into OneDrive.
Keep persisted fields append-only and preserve old customized permutations,
empty slots and base/extra metadata. Check earlier migration helpers too: a
validator that only accepts older IDs can reject the new saved layout before
its own migration runs.

Read capacities and registered IDs from the current layout schema rather than
copying counts from an older release. Empty targets show only in edit mode
and collapse without gaps when locked. Preserve all registered widget IDs,
including independent Indicators and World Clock entries where present, while
accepting repeated empty strings. Extend legacy region lists and split flags
in place, and check tray placements before inserting newly registered modules.
Find G8 anywhere in the center model rather than assuming index zero.

Drag hit testing must use mapped screen bounds at both corners when the bar
scales to fit an output. When a widget leaves a Loader, its old implicit width
may remain: gate content visibility and width on the slot's actual occupancy,
so a vacated slot collapses after locking. See the focused slot tests below.

Wire the panel into popup exclusivity, active-output routing, horizontal bar
anchors, vertical anchor maps and SidePanelPositioner. V2 connected surfaces
also need owner visibility and panel inset state. Keep backgrounds below the
controls (`z: -1` where the existing surface contract requires it).

When replacing code, leave existing credentials, settings, mounts, user units
and sync folders intact unless the user requests a specific migration or
removal. Preserve changes made concurrently through the UI. A preferences file
rewritten during startup may contain a schema extension or user edit; compare
fields before restoring an earlier snapshot.

## Theme and typography

Use the active Rise Theme instead of fixed plugin colors or host-global
styling. Common roles are `ink`, `sumi`, `seal`, `sealRaw`, `bg`, `sep`,
`fillIdle`, `fillHover`, `fillActive` and `mono`. Inspect the current variant's
contracts: V1 commonly supplies `pillRadius`, `tileRadius`, `pillBorder` and
`pillBorderW`; V2 supplies `panelRadius`, `panelButtonRadius`,
`panelOuterBorderColor` and `panelOuterBorderW`. The V2 outer-border toggle must
also affect panel tooltips. Use typed QML color properties before accessing
RGBA channels.

Keep typography, spacing, separators, progress tracks, icons and badges within
the same adapter contract as the rest of the panel. Compare neighboring Rise
panels rather than inheriting an external plugin's large hero typography.
The current OneDrive presentation uses a name-only `OneDrive` header above a
one-pixel `sep` line, with status and pause controls below it. Its title is
13 px with 2 px tracking, and its usage meter uses `sep` for the track and
`seal` for normal usage. These describe this widget's current design, not a
mandatory redesign of other Rise panels. Its cloud icon uses a 14 px glyph in the bar (the shape is 1.3 times wider
than that height) and an 8 px badge; compare rendered bounds, not only nominal
size, when matching neighboring glyphs. Keep the shared `OneDriveWidget`
default and the V1/V2 registrations at the same size unless a deliberate
variant-specific visual exception is documented.

Static panel labels should use `shared/components/UiText.qml`, whose
`Text.NativeRendering` matches Rise's small text. Merely selecting the same
font family and pixel size does not reproduce that rendering. Keep animated
or scaled text on the rendering mode required by its own component.

Bar widgets use `root.pillH` for their horizontal capsule height. The shared
OneDrive, CPU-temperature, and NordVPN components own the Loader geometry;
their V1 adapters provide only the themed background component and suppress it
vertically, where the rail supplies the surface. Keep the shared Loader
centered and sized to `root.pillH` so these modules do not render shorter
capsules than neighboring widgets. A direct shared-widget registration can
otherwise omit the V1 pill even when the panel itself is correctly themed.

OneDrive preferences use themed Full/Compact chips, bounded −/+ steppers and
a notifications switch; avoid platform-default ComboBox/SpinBox chrome in
this panel. The current design hides the scrollbar with AlwaysOff but retains
manual scrolling and keyboard navigation. Opening preferences does not scroll
automatically. The host expands to content height, capped only by available
screen space; do not reintroduce a fixed 664 px cap that clips preferences.
Screen bounds still take priority when the complete content cannot fit.

For the OneDrive UI compatibility layer, inspect
`shared/integrations/onedrive/ui/Style.qml`, `Color.qml`, `qmldir` and the
composed `Panel.qml`. A Binding connects Style to the active Theme and restores
it on destruction. Keep it reactive across theme/variant changes. Use explicit
namespaces for local singleton types such as Color, which can collide with Qt
types. Register imported directory components in qmldir: qmltestrunner's
implicit lookup can succeed while Quickshell's scanner fails.

## OneDrive contracts

Current paths (verify against the checkout before editing):

- `shared/integrations/onedrive/`: original account model, coordinator, helpers,
  composed panel, local UI compatibility types, licenses and backend fixtures.
- `shared/services/OneDriveService.qml`: shared preferences and IPC adapter.
- `shared/components/OneDriveWidget.qml`: aggregate status, icon/badge and clicks.
- `shared/panels/OneDrivePanel.qml`: Rise panel host; variant adapters supply surfaces.

Preferences are shared in `~/.cache/quickshell-rise-onedrive.json`; visibility
remains variant-specific. IPC retains `io.github.salemsayed.omaonedrive` for
notification actions, so do not simultaneously enable the standalone plugin
with that same target. Account selection must happen before an action or a
panel-opening retry. Notification repair must never fall back to a different
account when the target no longer exists. Close the exclusive-focus panel
before launching a terminal, browser or file manager.

Routine polling is local. Explicit quota and full-sync checks contact Microsoft;
the source also retries an old failed quota check when the panel opens.
Pause/resume and timed pause control the selected account's own unit. Preserve
the distinction between user and system units; system units have no user-timer
resume. Login, reauthentication and resync stay in the client's interactive
terminal flow. Never run them against a real account merely to test the widget.

## Timeouts and recovery

Separate client/network failures from stale query results in the panel. Inspect
the selected account's service and filtered journal; avoid printing credentials,
file names or complete account cache contents. Check current connectivity and
whether a later sync completed before treating a historical timeout as an
ongoing sync failure. Prefer local state reads; a fresh quota or full-sync query
contacts Microsoft and should be within the user's requested troubleshooting
scope, not an incidental fixture test.

The inspected integration uses 90 seconds in both helper constants
`QUOTA_TIMEOUT_SECONDS` / `SYNC_STATUS_TIMEOUT_SECONDS` and Account.qml's
`cloudTimeoutSec`; keep the displayed limit and watchdog consistent. This was
raised after an authorized quota diagnosis succeeded in 32.7 seconds while the
old wrapper killed it at 30 seconds. Measure before changing a limit; do not
hide failures, delete caches or restart services indiscriminately. Restarting
the shell reloads UI code; it does not itself repair the OneDrive connection.

## World Clock / Elsewhen

Compare against the actual original under the installed Omarchy shell's
`plugins/panels/elsewhen/`, not an obsolete Downloads copy. Read its manifest,
Panel, HeroTitle, Globe, GlobeFooter and CityRow, and preserve attribution for
ported components and data.

Keep state and asynchronous work in `shared/services/ElsewhenService.qml` and
`ElsewhenProcessAdapter.qml`; presentation lives in the shared Elsewhen panel
and `shared/integrations/elsewhen/`. V1/V2 wrappers supply only genuine surface
and placement differences. Do not merge the variants' entire Themes to avoid
two small adapters.

Validate city search/add/remove/reorder, duplicate-zone identity, people and
profile enrichment, 12/24-hour and temperature units, daylight scrubbing,
keyboard focus/Escape, home selection, globe drag/tilt/zoom/jump, opening spin,
greetings, arc-caption fallback, moon animation, persistence and reopening.
Make system-timezone changes require explicit confirmation; test success and
failure with an inert backend, never by changing the user's real clock.

The Rise-owned IPC target is `quickshell-rise.elsewhen`; do not duplicate
`omarchy.elsewhen`. Use `open` rather than `show`, which collides with the
Quickshell CLI subcommand. Read-only `times`, plus authorized `open`/`close`,
can verify the installed target without modifying city preferences.

Keep notification formatting and native `qs.Commons` translation imports in
the production host. Offscreen tests must explicitly substitute external
translation/notification I/O, not delete the feature. Supplement them with
native compilation of the unmodified import graph. Inspect deterministic
V1/V2 captures in both orientations and the live installed panel; report
which interactions were simulated. Public weather/profile requests are not
proof of private-account access or successful OS-timezone mutation.

Focused commands from the Rise checkout, using its existing scratch policy:

```bash
python3 tests/test-elsewhen-panel-content.py
python3 tests/test-elsewhen-service.py
python3 tests/test-elsewhen-process-adapter.py
python3 tests/test-elsewhen-ipc.py
python3 tests/test-elsewhen-panel-adapters.py
python3 tests/test-display-integration.py
```

## Bluetooth boundaries

Resolve the owning `hciN` through BlueZ Adapter1.Address before using
`bluetoothctl select`, which expects a controller address. Await connection
and disconnection method replies; piping `quit` immediately afterward can
end the client early. Read back actual connection state before claiming success.
If direct pairing outside Rise causes kernel controller errors or power
cycling, stop repeated trials and investigate the adapter/driver separately.
Do not erase unrelated pairings or reset a connected mouse without consent.

## Focused validation

Use the checkout's existing fixtures where present:

```bash
python3 tests/test-onedrive-runtime.py
node --test tests/onedrive-layout.test.mjs tests/expanded-bar-slots.test.mjs
python3 tests/test-v1-empty-slots.py
bash shared/integrations/onedrive/tests/run
python3 tests/test-source-layout.py
```

The empty-slot fixture exercises production V1 rows and mouse drag handlers,
including a scaled bar and collapse on lock. The panel/IPC fixture checks
preference updates and bounds, native text, theme bindings and panel height
using inert state and commands; the backend harness uses
stub Quickshell processes. For visual inspection, the runtime fixture supports
`RISE_ONEDRIVE_PREVIEW=/tmp/onedrive-preview.png`. Exercise Full/Compact, missing
client/authentication, multiple accounts, theme changes and border toggles.
Preserve test intent when updating old source assertions after namespacing or
moving UI elements. Keep command-boundary checks covering the actual adapters.

Compile the real import graph for both variants with the existing inert
integration probe when session access is available:

```bash
TMPDIR=/tmp python3 tests/test-display-integration.py
```

This probe compiles variants and exercises a simulated display controller; it
does not establish physical monitor behavior or live OneDrive synchronization.
Run the broad repository checks for changes spanning layout or lifecycle.
After applying authorized changes to the installed plugin, validate it, ping
the shell and inspect fresh logs. Report fixture, compilation, render and live
session evidence separately; a successful restart does not prove cloud sync.
