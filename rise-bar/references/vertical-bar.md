# V1/V2: orientation, styles, and panels

Use this reference for panel appearance and conditional orientation work. **Verified main/release 1.1.6 supports Top/Bottom only:** `barVertical` is fixed false, Left/Right are rejected and `VerticalBarContent.qml` is absent. The vertical rail/width procedures below describe a historical or different feature candidate whose enabling revision is not established here; do not use them as current main instructions. Check the actual revision and `tests/test-horizontal-only-bar.py` first. Approved popup/header/capture guidance remains applicable independently of vertical support.

Paths beginning with `variants/V2/...` are relative to the repository root.
Other module paths are relative to `variants/V2/`.

## V1: vertical surfaces

V1 uses `variants/V1/modules/VerticalBarContent.qml`, with surfaces for each
run defined by the saved splits, a visual `pillWidth` of 32 px, and a reserved
`verticalBarWidth` of 40 px. Preserve grouping, borders, and effects between
runs when changing widgets.

`standardizeSurface()` removes backgrounds and borders from widgets' direct
children; it does not reach backgrounds loaded through a `Loader`.
The V1 wrappers for `CpuTemperatureWidget.qml`, `OneDriveWidget.qml`, and
`StorageWidget.qml` use `backgroundComponent: null` when `barVertical` is true
to avoid double frames. `NordVPNWidget.qml` follows the same shared Loader
surface contract. Preserve their own backgrounds in horizontal mode. When
adding shared widgets, check which component draws the surface before adding
another background or border.

IDs differ between variants: temperature/storage use G20/G21 in V1 and
G16/G18 in V2. Do not transfer orders or preferences between these IDs.

## Where to make changes

- `variants/V2/Theme.qml`: persisted position in `barPosition`, `barVertical`, width derived from `verticalBarWidth`, colors, and preferences. Frost changes `barBg`; panel and tooltip backgrounds use `bg` to remain readable. Preserve the cache field previously used by Notch Frost.
- `variants/V2/BarSlot.qml`: owns slot models, saved order, lifecycle controller, and the window for each output. The horizontal presentation and `modules/VerticalBarContent.qml` reuse these models; only the active presentation instantiates widgets. Width, `exclusiveZone`, the mask, and `VariantRoot.barSize` must agree.
- `VerticalBarContent.qml`: separate columns for Left/top, Center/middle, and Right/bottom. `Instantiator` creates delegates in their corresponding columns; avoid reparenting Repeater delegates between different parents. Columns measure actual widgets, including workspaces with variable heights. Layouts exceeding the available height scroll without overlapping regions.
- `modules/SidePanelPositioner.qml`: applies X/Y bindings only in vertical orientation and restores horizontal bindings when leaving it. Every new panel attached to the bar needs this adaptation, including components shared with V1 that expose a `surfaceHost`.
- `modules/AiPanelSurface.qml`: renders the rounded background on the sides and the connected outline in horizontal mode. Hiding the entire component to remove the connector also removes the background of transparent cards.

## Visual and interaction contracts

### Restyle an approved popup

1. Freeze the approved middle: city list, globe, catalog, controls, positions and behavior. Treat the outer frame, header and named action buttons as separate targets. Do not resume an older redesign or original-parity goal after the user approves the current composition.
2. Read the actual reference adapters, such as `variants/V2/panels/WallpaperManagerQuickPanel.qml` and `NordVPNPanel.qml`. Reuse their `ConnectedPanelSurface`, shadow, owner-active binding, anchor, reveal and `SidePanelPositioner` contract. A tooltip with similar colors is not the reference panel renderer.
3. Keep V2's popup fill on `root.bg`; `barBg` can carry frost/opacity intended for the bar. Do not alter the compositor, global theme or approved content to mask a local surface mismatch. Respect V1's separate surface contract.
4. Derive named action controls' idle/hover/focus fills, text, border, font and radius from the active variant. Preserve `Add a city` and `Jump to a city` positions and functions. A selected tab in an example does not authorize a permanently accented action button.
5. Add a geometry or real-pointer regression before changing production code, verify RED, then apply the smallest patch and verify GREEN. Wait for QML layout readiness before subsequent pointer events; do not replace a click with a direct handler call merely to pass the fixture.

### Active-variant content colors

Check both live IPC and saved variant before interpreting a screenshot's variant name. A V2 reference can be shown while V1 is actually active. Clarify only if that changes whether to preserve or switch the selected variant.

For the approved Rise Elsewhen palette, both adapters pass their active root's foreground/background, muted, accent, seal, separator and idle/hover fills through `contentPaletteOverride`. Do not let V1 silently fall back to Commons while V2 uses Rise, or hardcode the example's green/accent. Preserve variant-specific frame geometry and approved fonts/layout. Test real city-row foregrounds and reactive ink/seal changes in both adapters, not only the new header.

### Elsewhen header contract

For the approved Rise header, follow the implementation in `shared/panels/WallpaperManagerQuickPanel.qml`, not the original Elsewhen hero typography. Confirm these values against the current reference before reusing them:

- Keep a separate literal `Elsewhen` heading above the separator; retain the home caption and `World Clock`/globe composition below it.
- Use shared `UiText`, `root.mono`, `root.ink`, 13px type, `Font.Medium`, letter spacing 2, a 24px heading row and elision before the close control. Do not substitute the content's Commons font or hardcode a machine's font name.
- Place `✕` at the right: `root.mono`, 12px, `root.muted` at rest and `root.seal` on hover, with a 120ms color transition and an 8px expanded hit area. Close the existing panel through its visibility/close path; do not terminate the shell. Preserve accessible naming and keyboard activation.
- Leave **12px between the title row and separator**, without moving the close icon out of vertical alignment. Account for positioner spacing: with a 2px Column spacing, an 8px spacer contributes a total gap of 2 + 8 + 2 = 12px. Do not increase spacing across the approved central content.
- Assert font family, size, weight, letter spacing, color, row/divider order and gap. Exercise real hover/click on the production close control in V1/V2 fixtures, plus bounds and existing city/globe interactions. Distinguish these fixtures from a physical installed-session click test.

### Capture and installed-component acceptance

1. Discover the actual integrated-shell PID. Through `terminal`, query `quickshell ipc --pid "$pid" call quickshell-rise.variant current`, `state`, `error`, and `quickshell-rise.lifecycle ready`. Wait for the settled active/ready state before interpreting an empty or fallback variant during reload.
2. Distinguish backend failure from absent UI. An X11 window enumerator may return zero windows for Wayland layer-shell panels. If Hermes computer use is available, run its doctor yourself. When authorized, use an available session-native capture route such as `grim`; never switch routes to evade an explicit approval denial.
3. Open through the existing `quickshell-rise.elsewhen open` IPC or the reference panel's existing UI. Do not assume Wallpaper Manager has an IPC opener or confuse its manager with the wallpaper picker. Batch open, settled reveal, capture and close to avoid focus changes invalidating the image. Do not select wallpapers, change VPN state or mutate preferences just to compare panels.
4. Match output, scale, backdrop and reveal state. Popup anchors can differ, so align the actual sampled coordinates. Moving a pointer between outputs can invalidate ownership or hit coordinates; save and restore pointer position for authorized foreground input. Stop coordinate retries if the popup disappears, rather than clicking through to another application. A successful dispatch alone does not establish an input effect.
5. Compare static, constant-fill samples away from content and edges. Label alpha fits as estimates; reject degenerate backgrounds, out-of-range results or large residuals. Old images do not establish current runtime state, and a fill estimate does not identify the responsible QML/compositor layer. Do not claim whole-panel pixel-perfect identity from a few matching samples.
6. Verify component replacement, not only installed bytes or `ready=true`. If an authorized automatic reload still renders old QML, preserve the evidence and use one supported `omarchy restart shell` when within scope. Discover the new PID afterward; do not reuse the old one, clear preferences or repeatedly rewrite unchanged files.
7. Compare scoped preference hashes and enabled plugins, installed source bytes, fresh logs and the actual requested header/contour. Remove temporary diagnostics. Report live render, pointer fixtures and unconfirmed live actions separately; a visible close icon does not establish a successful physical click.

In V2, `verticalBarWidth` is 36 px for Full/Dock/Notch and 40 px for Fit.
Fit keeps an outer gap of 2 px on each side; content uses side margins of
1 px, preserving 34 px internally across all styles. When changing the width,
check widgets, the mask, the exclusive zone, and panel anchors; also update
the geometry and expected positions in test fixtures.

Full occupies the available height; Fit has an outer margin; Dock touches the edge; Notch adds curves at the ends. Notch curves must fit within the input mask. Changing position preserves the selected style.

The vertical G8 group uses icon-only weather and a compact clock. Publish distinct Y anchors for weather and calendar based on the actual child positions. Respect `clock12h` and `widgetContentColor` so time and weather follow the group's contrast. Do not add temperature or other elements without considering the user's choice.

Workspaces reuse `WorkspaceWidget.qml`: a column on the sides and a row in Top/Bottom; preserve styles and the 5/10/active modes. The animated frame must follow the correct axis. Do not overwrite saved compactness settings to achieve vertical mode: derive the presentation from `barVertical`.

For the expanded slot capacities, empty-target persistence and scaled drag hit testing, see [widget-integration.md](widget-integration.md). V1 empty targets use the rail surface vertically and are visible only while editing.

Drag operations capture source and destination before changing models: the change may destroy the delegate. Do not access the delegate after the swap. Preserve anchor isolation and window recovery for each output.

## Evidence and tests

Run relevant tests without touching real accounts, monitors, or services:

```bash
python3 tests/test-vertical-bar-layout.py
python3 tests/test-v1-vertical-bar.py
python3 tests/test-v1-empty-slots.py
node --test tests/expanded-bar-slots.test.mjs
python3 tests/test-vertical-workspaces.py
python3 tests/v2-default-layout-regression.py
node --test tests/bar-style-settings.test.mjs
python3 tests/test-bar-style-preview.py
```

The layout test uses the real workspace and panel surface with inert service widgets. It validates overlap, resizing, Full/Fit/Dock/Notch, anchors, and horizontal restoration. Rendered background sampling uses `magick`. Other tests require `qs`, Qt/QML, and Node as specified by the runner. Also reject `ERROR qml:`: a later `Qt.quit()` can mask an earlier `Qt.exit(1)`.

Lint and offscreen fixtures do not prove physical interaction, multiple outputs, or actual dragging. After applying changes to the installation, validate the plugin, check `omarchy-shell shell ping`, and inspect the PID's own log. Report separately what was tested and what still requires visual inspection; do not change user preferences merely to obtain evidence.
