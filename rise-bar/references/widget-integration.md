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

The local slot expansion gives V1 32 positions (15 left, 3 center, 14 right),
including four empty targets per side and one empty center target by default;
the other center position holds the independent G23 Indicators module.
Empty targets show only in edit mode and disappear without leaving gaps when
locked. V2 allows up to 20 slots per side and 8 in the center through its `+`
controls. Preserve the 23 unique widget IDs (including independent G23 Indicators) while accepting repeated empty
strings; extend legacy region lists and split flags in place. Find G8 anywhere
in the center model rather than assuming it occupies index zero.

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
panels for normal integrations. For an explicitly requested native-parity port,
pass the original component's inspected Style tokens through the shared host
without restyling unrelated Rise panels.
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

## Status-indicator presentation and validation

1. Inspect both `variants/V1/modules/IndicatorsWidget.qml` and the V2 adapter, then trace `shared/components/IndicatorsWidget.qml` and its coordinator. Identify whether the reported defect concerns the idle marker, active glyphs, hover expansion or slot placement before editing. Change marker presentation in the adapters without changing shared action dispatch or persisted layout. Preserve each variant's palette and capsule contract.
2. Extend the existing contract test before editing production. Check idle visibility, unchanged neutral orientation, disappearance when an indicator activates or inactive controls are revealed, return after deactivation, and collapse when the module is disabled. Update every obsolete directional assertion, including assertions after state transitions; changing only the first marker assertion leaves contradictory test expectations.
3. Render idle, one-active and several-active states with and without reveal in both adapters. Derive the fixture coordinator's `hasActive` from the simulated statuses, not an absent member or an unrelated constant: otherwise a marker may remain visible over active glyphs while aggregate centering still passes. Check actual glyph ink against capsule bounds, not just the centered Text item. Retain active/revealed glyph and action assertions when extending the fixture to idle cases; skip an action target only when that state legitimately has no target.
4. Run the focused checks, then freeze the candidate and run the broad suite:

   ```bash
   python3 tests/test-omarchy-indicators.py
   python3 tests/test-indicators-visible-centering.py
   python3 tests/test-v2-inline-indicators.py
   bash tests/run-plugin-checks.sh
   ```

   Verify these entrypoints in the current checkout. Use `INDICATORS_ARTIFACT_DIR` with the render driver to retain images for inspection. Treat legacy vertical fixture cases as component coverage, not proof that the application supports side bars. Distinguish signal-handler tests from actual pointer-event acceptance.
5. After authorized deployment, compare both installed adapters, verify lifecycle readiness and preference/plugin snapshots, and inspect a fresh crop of the bar. An active status glyph correctly suppresses the idle marker; preserve it and report that live state rather than mistaking it for stale code. Validate the otherwise hidden idle state in the inert rendered fixture, without toggling real DND, stay-awake, recording or security settings just to obtain a screenshot.

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

## Native clock parity and reactive defaults

1. Read the installed native panel, model, hero and globe before changing the port. Distinguish original behavior from Rise-specific storage, IPC, surfaces and safety confirmation. Compare parsed catalogs as well as clock calculations; equal results for a bounded model matrix do not prove equal catalogs or UI.
2. Supply the shell's `Commons.I18n.formatLocale` and model word callbacks through the shared host. Include `t`, `tn`, number formatting, AM/PM and locale-aware dates where the inspected model supports them. Use that locale consistently for automatic hour format and units, not the process locale in one layer and shell locale in another. Preserve an absent setting as absent: inserting `hour24: false` during defaults merging turns an automatic choice into an explicit override. Test unset settings with `en_GB` and `en_US`, then explicit true/false under the opposite locale, including persistence/reopen.
3. Keep a standalone content fallback for inert component tests and wire real I18n/Style in the host. Extend composed fixture singletons when the host starts using new members, then run the real native import graph too. Test live locale/translation/token changes through V1/V2 in both orientations; matching initial English defaults can hide disconnected bindings.
4. Preserve the inspected meaning of `smoothMotion`: in Elsewhen it reduces labels/graticule while dragging or animating; it does not disable animation when false. Keep global motion policy through `Commons.Style.duration`. Use pointer press/release tests to show labels suppressed during movement and restored at rest, plus duration tests with smoothMotion disabled.
5. Resize the globe's owning container with its Canvas. For the inspected native expanded stage, use `Style.space(378)` rather than a separate fixed height; account for the Rise wrapper's padding independently. Assert containment and click Back after expansion: a centered Canvas taller than its parent can intercept the controls above even when its own height assertion passes. Measure mapped bounds before blaming scrolling or changing the fixture viewport.
6. Preserve the wrapping timezone confirmation and test it with a long IANA zone at narrow widths. Simulate command success/failure; never change the real timezone merely to verify text or callbacks. Report untranslated Rise-only controls and intentional extra actions instead of claiming complete localization or exact parity.
7. Compare the native painting pipeline, not just palette values, when matching globe labels. The native reference paints light text over the sea and repaints dark text through a coastline clip over land; a single foreground-colored pass can match the theme yet become illegible on pale continents. Reuse the inspected projected land geometry and test settled sea/land/coast labels, selection hitboxes and motion-detail suppression before calling the port equivalent.
8. Test the longest representative city list with the same viewport and scale as the reference. A partially visible Add action is not automatically a layout defect, but neither is a scrollbar proof of access: reveal the entire action with real scrolling, click it and verify search/focus. Change density or panel bounds only after comparing equivalent scenes; do not shrink typography merely to conceal inaccessible content.

### Tight clock test loop

Use a named Qt Quick test while diagnosing one content boundary; the selector
avoids rerunning the entire interaction suite on every iteration:

```bash
QT_QPA_PLATFORM=offscreen QT_QUICK_BACKEND=software QT_QPA_PLATFORMTHEME='' \
  /usr/lib/qt6/bin/qmltestrunner \
  -input tests/tst_elsewhen_panel_content.qml \
  ElsewhenPanelContent::test_smooth_motion_reduces_labels_only_while_moving
```

Confirm the runner path and import prerequisites for the current checkout. If the content imports `qs.Commons` or other host modules, prepare the same inert import tree as the official test wrapper and pass its root through `-import`; the direct selector above does not create those modules. Prefer passing the selector through the wrapper when it supports that operation. Do not remove production imports to recover the shorter command. After the focused loop, run the content, service, adapter and native-import runners and the broad suite.
Capture adapters with `RISE_ELSEWHEN_CAPTURE_DIR` when supported. Keep scratch
under the active profile's scratch directory. If a generated IPC socket path is
too long, use a short symlink in the accessible runtime directory pointing to
that scratch, verify its target, and remove only that alias afterward. Before
native Wayland probes, verify the compositor socket and export its actual name
when the caller lacks `WAYLAND_DISPLAY`; never infer that a missing variable
means the graphical session is stopped.

## Country-selector keyboard assessment

Use this as an assessment and implementation-test recipe, not a claim that keyboard support is already deployed.

1. Load the user's screenshot, locate both country selectors in `shared/panels/NordVPNPanel.qml`, and trace their selection callbacks to `NordVpnController.qml`. Inspect the input types, focus ownership, ListView current-index/highlight behavior and the panel's Escape handler. A styled Rectangle with MouseArea does not inherit text editing or keyboard navigation; exclusive window keyboard focus alone does not supply a focused selector.
2. Report what exists separately from what can be added. When implementation is authorized, preserve the active V1/V2 styling and share the behavior rather than duplicating it in variant adapters. Keep the country list supplied by the controller as the selection authority.
3. Define the keyboard contract before editing: Tab/Shift+Tab moves focus; typing filters the list without case sensitivity and can match display spaces against canonical underscores; Up/Down moves a visible highlighted candidate; Enter confirms only that candidate; Escape first dismisses the active chooser and restores its committed selection instead of immediately closing the whole panel. Keep only one chooser active so typing and navigation have an unambiguous owner. Do not promise translated country-name aliases unless explicitly implemented and tested.
4. Separate draft query/highlight from committed country. Confirm only a canonical member of `countryOptions`; never send arbitrary typed text to a VPN command. Selection may save the existing preference but must not invoke Connect, Apply, auto-connect toggling or any other service mutation. Preserve those explicit action boundaries.
5. Before changing production, add RED-capable tests for each selector, focus transfer, substring/full-name filtering, no matches, canonical underscore names, callback rejection, list bounds/scrolling, Escape propagation, cancel/reopen and dynamic country-list availability. Use an inert controller and assert zero connect/apply/toggle calls from typing, arrows or selection. Send a second pointer click into the expanded editor to verify caret editing, not just initial typing. Exercise both fields and their action buttons in a height-constrained composed panel: scrolling a ListView internally does not prove that the picker or the next Tab target is visible in the outer panel. Deliver actual key events before claiming keyboard behavior; extracted-function tests establish only filtering/selection logic.
6. Run the focused QtTest and native Quickshell panel drivers before the broad suite:

   ```bash
   python3 tests/test-nordvpn-country-keyboard.py
   python3 tests/test-nordvpn-selection-ui.py
   node tests/qr04n-panel-v1.mjs
   bash tests/run-plugin-checks.sh
   ```

   Verify these entrypoints against the current checkout. Give native drivers the actual accessible Wayland runtime; keep the controller inert. Preserve production directory imports in the native fixture and extend explicit dependency-copy lists when introducing a shared picker. A flat materialized QtTest can resolve unqualified sibling types that fail in Quickshell; use explicit namespaces such as `import "." as CommonComponents` and `CommonComponents.UiText`. Keep native panel instantiation mandatory in the broad runner, not merely a one-off probe or lazy compile check. Freeze the candidate only after both event behavior and native imports pass.
7. Before an authorized local application, inspect the plugin watcher's reload scope and ask once if application requires broader shell-plugin reload permission. Refresh preference hashes and enabled-plugin snapshots immediately before writing, rather than comparing against a stale development-start snapshot that may predate user edits. Back up the old panel outside the watched tree, install a newly referenced component before its consuming panel, then compare installed bytes and inspect fresh reload logs. Read `quickshell-rise.variant` current/state/error and `quickshell-rise.lifecycle` ready through `omarchy-shell`; do not guess a shortened IPC target. Verify the active variant and preferences remain unchanged. Report isolated key-event acceptance separately from live-panel interaction and real VPN connectivity; never reconnect the user's VPN merely to test this UI.

## Bluetooth command boundaries

1. Read `bluetoothctl list`, `bluetoothctl show` and the current controller inventory before changing pairing state. Distinguish a missing/off adapter from a panel helper passing the wrong controller identifier.
2. Validate `hciN` and the device MAC before building paths or commands. Resolve the selected adapter with:

   ```bash
   busctl --system --timeout=5s get-property org.bluez "/org/bluez/$adapter" org.bluez.Adapter1 Address
   ```

   Validate the typed address reply. `bluetoothctl select` takes that address, not the kernel name `hci0`. Prefer the BlueZ property contract over assuming a particular sysfs address attribute. Fail before device actions if lookup fails.
3. Keep asynchronous command clients alive until their terminal response. Piping `connect` followed by `quit` can end the client before connection completes. For an already known device, a synchronous method boundary is:

   ```bash
   busctl --system --timeout=11s call org.bluez "$device_path" org.bluez.Device1 Connect
   ```

   Construct `device_path` from the validated owning adapter and MAC, with colons replaced by underscores. `Disconnect` uses the same interface. Bound the subprocess separately, propagate BlueZ errors/nonzero exits, and keep the UI watchdog consistent. This verifies method completion, not usable audio or input.
4. Add regressions for controller resolution, non-default adapters, missing/invalid lookup replies, timeouts and method failure propagation using inert subprocesses. Read back deployed helper bytes. A real read-only `info` query establishes lookup/read access only; do not label it a successful connection test.
5. After authorization to connect a named device, read back `Device1.Connected`, `Paired`, `Bonded` and `ServicesResolved` on the same object. Report the returned failure rather than converting command dispatch into success. Obtain explicit consent before removing a pairing or power-cycling the controller; resetting the controller can interrupt unrelated mice and audio.
6. If the live attempt also fails outside the panel, inspect `journalctl -b -u bluetooth` and narrowly filtered kernel Bluetooth logs. Treat controller hardware-error events as evidence to investigate the controller/driver/firmware, not proof of physical damage or a panel-only bug. Contradictory pairing responses and failed discovery are unresolved diagnostic evidence, not a validated recipe for clearing records or repeatedly resetting services. Stop reporting a software patch as an end-to-end fix when real connection acceptance fails.

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
python3 tests/test-display-integration.py
```

This probe compiles variants and exercises a simulated display controller; it
does not establish physical monitor behavior or live OneDrive synchronization.
Run the broad repository checks for changes spanning layout or lifecycle.
After applying authorized changes to the installed plugin, validate it, ping
the shell and inspect fresh logs. Report fixture, compilation, render and live
session evidence separately; a successful restart does not prove cloud sync.
