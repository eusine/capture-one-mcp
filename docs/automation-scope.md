# Automation Scope

Capture One MCP is a dedicated control surface for Capture One, not a retouching bot and not a generic SDEF framework.

The goal is to expose the parts of Capture One that are useful for external tools: inspect the active work context, read/write supported adjustments, locate rendered previews, and hand those previews to analysis systems such as vision models.

## What this project automates

### Work context

- Detect whether Capture One is installed/running.
- Inspect the active document where the scripting surface exposes it.
- Read the current selected variants.
- Read process recipes.

### Safe Capture One controls

- Read supported adjustment fields from the selected variants.
- Write supported scalar/text/boolean adjustment fields when write mode is explicitly enabled.
- Set simple organizational fields such as rating.
- Trigger selected-variant processing/export through Capture One recipes.
- Trigger tethered capture where Capture One supports it.

### Preview acquisition

- Locate Capture One preview cache candidates for selected variants.
- Convert preview cache files to ordinary image files for external analysis.
- Keep this separate from Capture One export/process so analysis loops can be lightweight.

### Vision-assisted workflows

Vision models can compare a reference image and the current Capture One preview, then describe perceptual differences in Capture One terms:

- subject/background priority
- luminance structure
- white balance and color cast
- color density by region/hue
- material rendering, such as skin, hair, fabric, leather, and product surfaces
- finish, including clarity, haze, grain, and vignette

The MCP server provides the control and preview surface. A higher-level client may decide how to use a vision report, but the server itself should not claim to fully automate subjective grading.

## What this project does not try to automate

- Replacing a photographer, retoucher, or colorist.
- Guaranteeing exact visual matching from a single reference.
- Shipping a one-click “make this look like that” black box.
- Driving Capture One's UI sliders as the main control path.
- Building a generic macOS scriptable-app MCP generator.
- Depending on the Capture One Plugin SDK unless a specific missing capability proves it is necessary.

## Control philosophy

Prefer official Capture One scripting support over screen scraping. Use UI automation only as a fallback for observation or unavailable features.

Prefer explicit fields over opaque commands. A tool should say what it reads or writes, and mutating tools should stay behind `CAPTURE_ONE_MCP_ALLOW_WRITE=1`.

Prefer perceptual analysis over fixed recipes. Tone matching should first determine what matters in the reference, then map that intent to Capture One controls. Hard-coded rules like “muted means desaturate” or “commercial means warm skin” do not belong in the MCP layer.
