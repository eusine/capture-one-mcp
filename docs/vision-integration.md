# Vision Integration Notes

This document defines how a vision model should analyze Capture One previews so its output can be mapped to Capture One controls. It deliberately avoids shipping an automated editing workflow.

## Role split

```text
Capture One MCP = exact state/control bridge
Preview cache = current rendered preview source
Vision model = perceptual analysis
External orchestrator = decides whether/how to apply edits
```

The vision model should not pretend it can infer exact Capture One slider values from pixels alone. It should report perceptual deltas and recommended control directions.

## Subject-priority rule

For fashion/commercial/editorial references, prioritize the model/subject over the background unless background matching is explicitly requested.

Recommended analysis order:

1. face/skin brightness, warmth, and color cast
2. skin contrast and direct-flash definition
3. makeup and hair color density; these should remain alive in commercial work
4. black clothing depth, lifted/matte black level, separation, and glossy highlights
5. accent colors such as shoes/lip/nails; control distraction without killing product color
6. background tone only as secondary support

Do not over-brighten or over-cool the whole frame just to match the background. If background and subject conflict, protect the subject.

For commercial fashion, do not equate “muted” with extreme global desaturation. A reference can be muted because its level endpoints, contrast, and highlight response are soft while makeup, hair, and product/accent color remain present. Prefer moderate saturation adjustments first; use channel/local controls for loud colors instead of globally flattening the file.

## Capture One-oriented analysis axes

### Base render

Relevant controls:

- `color profile`
- `film curve`
- `white balance preset`

Useful notes:

- `Linear Response` gives a neutral base and more grading freedom.
- `Auto` can move the baseline per image.
- `Film Standard` and other film curves can bake in contrast/response before later adjustments.

### Exposure and dynamic range

Relevant controls:

- `exposure`
- `brightness`
- `contrast`
- `highlight recovery`
- `shadow recovery`
- `white recovery`
- `black recovery`
- level fields

Vision should describe:

- overall exposure
- black point: crushed / deep / neutral / lifted / washed
- white point: clipped / bright / neutral / muted / dull
- midtone placement
- highlight rolloff
- whether blacks should be lifted/matte or deep/crushed
- Capture One `level midtone rgb` is not Lightroom-style gamma centered around 1.0 in this bridge; observed default is `0.0`. Treat it as a sensitive offset control: use tiny moves around the current value, not jumps like `0.96`, which can blow the image brighter than intended.

### White balance and cast

Relevant controls:

- `temperature`
- `tint`
- RGB/channel levels where needed

Vision should describe:

- warmer/cooler direction
- green/magenta direction
- whether cast affects skin, shadows, highlights, or the full image

### Color density and grading

Relevant controls:

- `saturation`
- shadow/midtone/highlight color balance hue/saturation/lightness
- future advanced color editor helper

Vision should describe:

- subject saturation
- skin hue/saturation/luminance
- hair color density
- black clothing color contamination
- accent colors that pull too much attention

### Texture and finish

Relevant controls:

- `clarity amount`
- `clarity structure`
- `dehaze amount`
- sharpening/noise reduction fields
- grain/vignetting fields

Vision should describe:

- flash crispness vs digital harshness
- local contrast
- haze/softness
- grain and muted editorial finish

## Tone matching priorities

Vision should first identify the reference's photographic intent, then map that intent to Capture One controls. Avoid hard-coded style recipes such as “always desaturate” or “always warm skin.” The same adjective can mean different slider moves depending on the image.

Analyze in this order:

1. **Subject priority** — decide what must match first: face/skin, product, clothing, hair, background, or the whole frame. In commercial/fashion images, the model/product usually wins over the backdrop.
2. **Luminance structure** — compare exposure, black point, white point, midtone placement, highlight rolloff, and shadow openness. Decide whether the reference is contrasty, compressed, lifted, clipped, matte, glossy, or flash-flat before touching color.
3. **Color cast and white balance** — separate global WB errors from local casts. Judge whether warmth/coolness or green/magenta bias affects skin, neutrals, shadows, highlights, or only the background.
4. **Color density** — compare saturation by subject region and hue family. Preserve intentional color in makeup, hair, clothing, and product details; reduce only the hues or regions that are louder than the reference.
5. **Material rendering** — inspect how skin, hair, fabric, leather, metal, and other materials carry contrast, texture, gloss, and specular highlights. These often matter more than global saturation.
6. **Finish** — only after tone/color are close, evaluate grain, clarity, haze, vignette, and overall polish.

Control-selection rules:

- Prefer the broadest control that explains the mismatch, but do not use a global control when the mismatch is local to skin, clothing, hair, or background.
- Treat saturation, tint, and color balance as relative corrections, not style labels. A muted reference can still have strong makeup/hair/product color; a warm reference can still need less magenta.
- Shape blacks and highlights according to the reference's endpoint behavior: lifted, crushed, glossy, matte, clipped, or compressed. Do not assume deeper black or brighter white is automatically closer.
- Keep `level midtone rgb` at or near its current value unless analysis specifically proves the midtone pivot is wrong. In this bridge it has been observed to default to `0.0`; move it only in tiny increments.
- When a reference has conflicting cues, state the tradeoff explicitly: e.g. preserving skin color may prevent the background from matching exactly.

## Suggested vision response schema

```json
{
  "summary": "short perceptual diagnosis",
  "confidence": 0.0,
  "priority": "subject|background|balanced",
  "subject": {
    "skin_brightness": "too_dark|matched|too_bright",
    "skin_temperature": "too_cool|matched|too_warm",
    "skin_tint": "too_green|matched|too_magenta",
    "face_contrast": "too_flat|matched|too_hard",
    "hair_density": "too_light|matched|too_heavy",
    "black_clothing": "crushed|deep|matched|lifted|washed",
    "accent_colors": "too_muted|matched|too_loud"
  },
  "global_tone": {
    "exposure": "lower|hold|raise",
    "contrast": "lower|hold|raise",
    "black_point": "lower|hold|lift",
    "white_point": "lower|hold|raise",
    "saturation": "lower|hold|raise"
  },
  "recommended_control_directions": [
    {
      "field": "Capture One field name",
      "direction": "decrease|increase|set|hold",
      "magnitude": "tiny|small|medium|large",
      "reason": "perceptual reason"
    }
  ],
  "do_not_change": ["aspects already close enough"]
}
```

## Deterministic metrics to pair with vision

A future orchestrator should pair vision analysis with image metrics from the preview JPEG:

- luminance histogram percentiles
- shadow/highlight clipping percentages
- average Lab/HSV globally and by tonal band
- subject-mask skin/hair/clothing samples where masks are available
- edge/detail proxy for clarity/sharpness

Vision should lead the perceptual interpretation; metrics should prevent slider hallucination.
