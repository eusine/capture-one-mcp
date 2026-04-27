# Capture One MCP Surface

This project exposes Capture One's macOS AppleScript scripting surface through MCP. It is intentionally a Capture One-specific low-level bridge: it can inspect Capture One state, read/write supported settings, and locate preview cache files. It does not include an automated image-editing/style-matching agent. See [`automation-scope.md`](automation-scope.md) for the boundary.

## Connection model

```text
MCP client <-> capture-one-mcp <-> osascript <-> Capture One
```

The server talks to Capture One via the installed scripting dictionary:

```text
/Applications/Capture One.app/Contents/Resources/CaptureOne.sdef
```

Tested local app bundle:

```text
/Applications/Capture One.app
com.captureone.captureone16
Capture One 16.7.3.x
```

## Safety model

Read-only tools are available by default. Mutating tools require:

```bash
CAPTURE_ONE_MCP_ALLOW_WRITE=1
```

This keeps accidental rating/adjustment/export/capture operations locked unless the user explicitly starts the server in write mode.

## Current tools

### Read-only

- `capture_one_status`
- `capture_one_selected_variants`
- `capture_one_list_recipes`
- `capture_one_adjustment_fields`
- `capture_one_get_selected_adjustments`
- `capture_one_find_selected_preview_cache`
- `capture_one_convert_selected_preview_cache`

`capture_one_convert_selected_preview_cache` converts internal preview cache files to temporary JPEGs for vision analysis. It does not invoke Capture One export/process.

### Write/export/capture gated by `CAPTURE_ONE_MCP_ALLOW_WRITE=1`

- `capture_one_set_selected_adjustments`
- `capture_one_set_selected_rating`
- `capture_one_process_selected`
- `capture_one_capture`

## Adjustment coverage

The generic adjustment read/write path currently covers direct scalar/text/boolean fields from `adjustment settings`, including:

- base characteristics: `color profile`, `film curve`, `white balance preset`
- white balance: `temperature`, `tint`
- global tone: `exposure`, `brightness`, `contrast`, `saturation`
- HDR/recovery: `highlight recovery`, `shadow recovery`, `white recovery`, `black recovery`
- levels: `level shadow rgb`, `level highlight rgb`, `level target shadow rgb`, `level target highlight rgb`, `level midtone rgb`, plus RGB channel variants
- color balance: shadow/midtone/highlight hue/saturation/lightness
- texture: clarity, dehaze, sharpening, noise reduction
- finish: vignetting, film grain, moire

Nested/special objects need dedicated typed helpers before they should be exposed:

- curves (`rgb curve`, `luma curve`, `red/green/blue curve`)
- advanced color editor settings
- RGB color object fields
- layers/masks/local adjustments

## Preview cache lookup

Observed session-style cache layout:

```text
<image folder>/CaptureOne/Cache/Proxies/<raw filename>.cop
<image folder>/CaptureOne/Cache/Proxies/<raw filename>.cof
<image folder>/CaptureOne/Cache/Thumbnails/<raw filename>.[uuid].cot
```

Observed formats:

- `.cop`: JPEG XL container, readable by macOS `sips`
- `.cot`: JPEG thumbnail
- `.cof`: grayscale JPEG/focus sidecar

Catalog-specific layouts still need more samples. The lookup code checks selected image-folder and current-document path/folder candidates, but managed/referenced catalog variations should be hardened with real fixtures.
