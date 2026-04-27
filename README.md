# Capture One MCP

AppleScript-backed MCP server for Capture One on macOS.

This server uses Capture One's installed scripting dictionary (`CaptureOne.sdef`), not screen scraping. By default the server expects Capture One at `/Applications/Capture One.app`; override with `CAPTURE_ONE_APP` if needed.

## Tools

Read-only by default:

- `capture_one_status` — install/running status, app version, current document, selected count
- `capture_one_selected_variants` — selected variants as TSV
- `capture_one_list_recipes` — process recipes as TSV
- `capture_one_adjustment_fields` — list supported adjustment fields
- `capture_one_get_selected_adjustments` — read tone/color adjustment values from selected variants
- `capture_one_find_selected_preview_cache` — locate internal Capture One preview/thumbnail cache files for selected variants
- `capture_one_convert_selected_preview_cache` — convert internal preview cache to temporary JPEGs for vision analysis, without Capture One export

Write/export tools are locked unless started with `CAPTURE_ONE_MCP_ALLOW_WRITE=1`:

- `capture_one_set_selected_adjustments` — generic selected-variant adjustment writer
- `capture_one_set_selected_rating`
- `capture_one_process_selected`
- `capture_one_capture`

## Adjustment coverage

The server currently exposes 83 directly writable adjustment fields from Capture One's AppleScript dictionary, including white balance, exposure, contrast, saturation, color balance, levels, highlight/shadow recovery, clarity, dehaze amount, vignette, sharpening, noise reduction, film grain, and moire.

Nested/special objects need dedicated helpers next: curves, color editor settings, and RGB color coercion. Those are controllable in principle, but should not be treated as a loose string API.

## Preview cache notes

Capture One can maintain per-image internal cache folders such as:

```text
<image folder>/CaptureOne/Cache/Proxies/<raw filename>.cop
<image folder>/CaptureOne/Cache/Proxies/<raw filename>.cof
<image folder>/CaptureOne/Cache/Thumbnails/<raw filename>.[uuid].cot
```

On the checked local sample, `.cop` is a JPEG XL container readable by macOS `sips`, `.cot` is JPEG, and `.cof` is a grayscale JPEG focus/preview sidecar. `capture_one_convert_selected_preview_cache` uses `sips` to create temporary JPEGs for vision models; it does not ask Capture One to export/process the image.

Session/catalog/folder-browser storage can differ, so cache lookup checks the selected image folder plus current-document path/folder candidates. Real catalog packages still need a live catalog sample to harden lookup rules.

## Install / build

```bash
npm install
npm run build
```

## Run

```bash
node dist/index.js
```

Enable mutating tools only when you want the model to change Capture One state:

```bash
CAPTURE_ONE_MCP_ALLOW_WRITE=1 node dist/index.js
```

## MCP client config example

```json
{
  "mcpServers": {
    "capture-one": {
      "command": "node",
      "args": ["/absolute/path/to/capture-one-mcp/dist/index.js"],
      "env": {
        "CAPTURE_ONE_MCP_ALLOW_WRITE": "0"
      }
    }
  }
}
```

For export/capture/rating changes, set `CAPTURE_ONE_MCP_ALLOW_WRITE` to `1`.
