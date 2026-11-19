# V1/V2: orientation, styles, and panels

Use this reference for panel appearance and conditional orientation work. Establish support from the current Themes, component files and `tests/test-horizontal-only-bar.py` where present. If `barVertical` is fixed false, Left/Right are rejected and `VerticalBarContent.qml` is absent, the vertical rail/width procedures below do not apply to that candidate. Do not reintroduce the feature to satisfy legacy tests. Approved popup/header/capture guidance remains applicable independently of vertical support.

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

### Restyle an approved popup without changing its content

1. Record the approved inner composition and the exact remaining targets: frame, connector/anchor, or named action buttons. If a new instruction approves the content, stop older parity work on its density, catalog, controls and layout. Keep styling and behavioral acceptance separate.
2. Trace the actual reference adapter and renderer. A V2 tooltip using `bg`, `panelOuterBorderColor` and a radius is not equivalent to a panel using `ConnectedPanelSurface`. Inspect the reference's frame transparency, surface slot, shadow, connector, owner-active binding, target anchor, reveal progress and stacking. Reuse the chosen contract rather than approximating its shape with an unrelated Rectangle. Consult references/code-review.md for reparented-surface ordering.
3. Keep variant decoration in the adapter and approved content in the shared body. When a connected surface owns the fill and outline, avoid an opaque duplicate card or border that can obscure the connector. Preserve `SidePanelPositioner`, per-output anchors and horizontal/vertical restoration. Do not migrate every variant to one palette merely because a reference uses it.
4. For action-button restyling, derive idle/hover/pressed fill, text, accent, border, font and radius coherently from the active variant's existing button contract. A selected tab or toggle in the example shows the active state, not an action's resting appearance; do not make actions permanently accented or copy the theme's current accent as a constant. Do not combine `Commons` text/border/idle colors with only a Rise accent and call that variant styling. Test with deliberately different root and Commons palettes, then change the theme after creation to verify reactive bindings. Keep the approved native-content palette independent when only buttons or frame are in scope.
5. Diagnose color differences along the rendered path: bar fill, popup surface, content background and action fill. `barBg` can use frost/opacity while `bg` remains nearly opaque, so identical RGB values need not produce identical pixels. Compare the same backdrop, orientation, scale and reveal state; do not blindly replace `bg` with `barBg` or recolor the whole content to chase a screenshot.
6. Establish settled runtime before capturing. Through `terminal`, query `quickshell ipc --pid "$pid" call quickshell-rise.variant current`, then `state`, `error`, and `quickshell-rise.lifecycle ready` on the actual integrated-shell PID. Use bounded reads through the loading transition. Inspect the implementation of display helpers before interpreting them: `lifecycle.version` may fall back to V1 while `runningVariant` is temporarily empty. Do not change saved variant, rewrite identical files, clear caches or restart repeatedly because of that transient label.
7. Verify real render and input before acceptance: open the existing popup through its supported IPC, wait for settled layout/reveal, capture the actual surface, and inspect the requested contour and controls. A matching installed file, palette binding or artificial-color fixture is not proof of the live result. Use available capture tools before requesting another user screenshot; stop at an explicit tool-approval denial rather than retrying through a different path. Preserve functions and preferences and remove temporary runtime instrumentation before delivery.

### Match content colors to the active variant

1. Load the supplied screenshot and separately identify branding, inner hero, city rows, secondary labels and actions. Query the actual integrated-shell `quickshell-rise.variant current` and lifecycle readiness, then compare the saved active variant; a light header with green content can come from two palette sources rather than a stale screenshot or failed theme reload.
2. Trace the variant adapter's `contentPaletteOverride`, the shared body's palette fallback and the row's `foreground`/`surfaceBase` bindings. If V1 falls back to Commons while V2 injects Rise tokens, changing V2 or merely recoloring the header cannot fix the current V1 content. Clarify a genuinely ambiguous variant target once; preserve the user's current variant unless switching was explicitly chosen.
3. For the approved active-bar Elsewhen contract, supply `bg: root.bg`, `ink: root.fg`, `accent: root.accent`, `seal: root.seal`, `sep: root.sep`, `fillIdle: root.fillIdle`, `fillHover: root.fillHover` and `sumi: root.muted` through the adapter's `contentPaletteOverride`. Keep existing fonts, city order, controls and variant-specific frame geometry; do not hardcode the screenshot's RGB or copy an entire Theme.
4. First make a composed regression fail against deliberately distinct Commons and root palettes. Check both adapters' palette tokens, rendered city-row foreground/background and reactive changes to root ink/seal after creation. Search the fixture for older V1-stays-Commons assertions and update that superseded color contract without removing row, font, frame or interaction coverage. Run adapter and shared-content suites after the change.
5. Apply only the affected production adapter after comparing its installed baseline and backing it up. Preserve the live variant and preference hashes, perform authorized component replacement, then capture the actual popup and inspect city/time/hero text as well as branding. Deliver a tight popup crop. Label earlier audit results with their old candidate fingerprint; they do not automatically cover a later color patch.

### Add panel branding above an approved content header

1. Read the shared Wallpaper Manager header before matching its style. Use a separate left-aligned UiText title with active `root.mono`, `root.ink`, 13px, `Font.Medium`, letter spacing 2 and a 24px row. Reserve room for a right-aligned `✕` using `root.mono`, 12px, muted at rest, seal on hover and a 120ms color transition. Expand the hit area by 8px and retain accessible naming plus keyboard activation. Close through the existing panel visibility path; do not replace the inner World Clock heading or globe target.
2. Move the existing one-pixel separator below the title row with a 12px gap, leaving the home caption and remaining content in their original relative order. Include positioner spacing in the measurement: a Column with 2px spacing and an 8px intervening spacer yields the intended 12px gap. Do not enlarge all Column gaps or shift the close icon vertically to create the space. Derive header height from the containing Column to avoid overlap.
3. Before production edits, add a failing composed-adapter assertion for the exact title, typography and geometry. Use `mapToItem(header, 0, 0)` to compare positions in a common coordinate system: title-row bottom plus the intended gap <= separator top, separator bottom <= home-caption top, caption bottom <= inner-heading top. Exercise actual hover/click on the production close control in both adapters and assert panel dismissal; keep physical live input acceptance separate from fixture results.
4. Run both the panel-adapter and shared-content runners after the change. Apply only the changed production file with a pre-change backup; retain unrelated installed changes.
5. After settled reload, inspect the exact new title in a current native capture. If it is absent despite matching installed bytes, follow the component-replacement gate in SKILL.md rather than editing the layout again. Deliver a tight header crop to demonstrate the requested ordering without exposing unrelated windows.

### Standardize module width against the Control Center

1. Measure the rendered Control Center's outer capsule in the selected variant and style, then identify the shared rail/surface property that owns that width. Do not copy a glyph's width or the window's exclusive zone as the module-width reference.
2. Enumerate every module in the variant registry and check its outer background, Loader and content bounds. Apply one coherent width contract to the vertical surfaces, including small icons, Indicators, tray, clock, workspaces, media and sliders. Keep variant-specific colors, borders and height behavior.
3. Center content within that common surface and test overflow, clipping, hover and pointer targets. Keep long modules tall enough for their controls rather than shrinking or stretching them into square tiles.
4. Test Left/Right, affected styles, disabled/empty states and restoration to Top/Bottom. Compare actual rendered widths and retain a per-module result so a few passing examples do not stand in for all modules.
5. Apply validated changes with backup and re-check the live variant immediately before deployment. Preserve current ordering, theme and user preferences; do not switch the user's bar to another variant for test evidence.

The inspected V2 geometry uses `verticalBarWidth` of 36 px for Full/Dock/Notch and 40 px for Fit; read the current contract before relying on these values.
Fit keeps an outer gap of 2 px on each side; content uses side margins of
1 px, preserving 34 px internally across all styles. When changing the width,
check widgets, the mask, the exclusive zone, and panel anchors; also update
the geometry and expected positions in test fixtures.

Full occupies the available height; Fit has an outer margin; Dock touches the edge; Notch adds curves at the ends. Notch curves must fit within the input mask. Changing position preserves the selected style.

The vertical G8 group uses icon-only weather and a compact clock. Publish distinct Y anchors for weather and calendar based on the actual child positions. Respect `clock12h` and `widgetContentColor` so time and weather follow the group's contrast. Do not add temperature or other elements without considering the user's choice.

Workspaces reuse `WorkspaceWidget.qml`: a column on the sides and a row in Top/Bottom; preserve styles and the 5/10/active modes. The animated frame must follow the correct axis. Do not overwrite saved compactness settings to achieve vertical mode: derive the presentation from `barVertical`.

For the expanded slot capacities, empty-target persistence and scaled drag hit testing, see [widget-integration.md](widget-integration.md). V1 empty targets use the rail surface vertically and are visible only while editing.

For vertical slot editing, trace the complete horizontal begin/move/end/cancel
contract before porting it. A vertical press/release swap is not equivalent to
dragging with ghost, hit testing, cancellation, tray support and persistence.
Use vertical delegates as targets, map both rectangle corners, and account for
scroll, clipping, scaling and separate tray windows in a consistent coordinate
space. Preserve each variant's fixed/base/extra slot rules instead of creating
a second ordering schema. Acceptance tests must deliver pointer events to the
production handlers, not only call the swap function directly.

Drag operations capture source and destination before changing models: the change may destroy the delegate. Do not access the delegate after the swap. Preserve anchor isolation and window recovery for each output.

## Evidence and tests

### Stabilize pointer fixtures before diagnosing production

1. Wait for `Instantiator`/positioner layout readiness before choosing pointer coordinates. Assert delegate counts against model counts, correct region/index metadata, expected module identities, and non-overlapping positions. Poll these invariants with a bounded timeout instead of relying solely on a fixed sleep; asynchronous layout can otherwise route a click into a different slot.
2. Locate source and destination by model identity and region/index, then assert each GID against the fixture's explicit model. Do not infer a module's index from a default order: saved layouts and migrations can move it legitimately.
3. Map the centers of the settled cells into the event target's coordinate space. After press, assert the production drag state and source model/index; after movement, assert destination model/index and visible feedback; after release, assert order, persistence and cleanup. This identifies the first broken boundary rather than treating a wrong source index as proof of a production defect.
4. Cover an occupied target, an empty target, tray movement and cancellation, then rerun horizontal interaction regressions. Use inert services and the real production handlers.
5. Report behavioral test names, not just QtTest totals: initialization and cleanup also count as passes. A synthetic pointer test proves the handler path in its fixture, not physical input on the installed Wayland bar. Keep broad-suite failures visible even when the focused interaction tests pass.

Run relevant tests without touching real accounts, monitors, or services:

```bash
python3 tests/test-vertical-bar-layout.py
python3 tests/test-v1-vertical-bar.py
python3 tests/test-v1-empty-slots.py
python3 tests/test-v1-vertical-slot-interaction.py
python3 tests/test-v2-vertical-slot-interaction.py
python3 tests/test-tray-panel-runtime.py
node --test tests/expanded-bar-slots.test.mjs
python3 tests/test-vertical-workspaces.py
python3 tests/v2-default-layout-regression.py
node --test tests/bar-style-settings.test.mjs
python3 tests/test-bar-style-preview.py
```

The layout test uses the real workspace and panel surface with inert service widgets. It validates overlap, resizing, Full/Fit/Dock/Notch, anchors, and horizontal restoration. Rendered background sampling uses `magick`. Other tests require `qs`, Qt/QML, and Node as specified by the runner. Also reject `ERROR qml:`: a later `Qt.quit()` can mask an earlier `Qt.exit(1)`.

Lint and offscreen fixtures do not prove physical interaction, multiple outputs, or actual dragging. After applying changes to the installation, validate the plugin, check `omarchy-shell shell ping`, and inspect the PID's own log. Report separately what was tested and what still requires visual inspection; do not change user preferences merely to obtain evidence.
