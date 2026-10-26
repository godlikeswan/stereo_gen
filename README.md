# stereo_gen

**Stereo Image Generator** — a single-page random dot stereogram (RDS) maker. Type a word, and it hides the word inside a field of random dots that only appears when you view the two halves with the right eye technique.

Everything lives in one file: `index.html`. No build step, no dependencies, no server required.

## Running it

Open `index.html` in a browser. No build step, no dependencies.

To serve it over HTTP instead:

```
npx serve .
# or
python -m http.server
```

## How to see the hidden text

The canvas is split into two halves. The left half is a fixed random-dot reference; the right half is the same noise re-sampled through the text's depth map. Viewed normally it looks like uniform noise. The shape only exists in the *difference* between the halves, so you have to fuse them:

- **Parallel viewing** — relax your eyes as if looking through the screen, past the image. The two halves drift together into a third image in the middle, and the text floats above the dot field.
- **Cross-eyed viewing** — cross your eyes until the two halves overlap. The text appears, but the depth is reversed.

Fullscreen mode is there for this: more dots and a bigger canvas make fusion much easier than a 720x480 thumbnail.

On first try it usually takes a minute. Get comfortable at a distance where the halves are roughly eye-width apart, and don't chase the effect — let it come to you.

## Controls

| Control | What it does |
| --- | --- |
| **Hidden text** | The text baked into the depth map. Re-renders on every keystroke. |
| **Seed** | Seeds the noise generator. The same seed always produces the same dot field, so a pattern survives a reload or a checkbox toggle. |
| **Shift & depth** | Moves the shape horizontally and sets how far it floats. One slider drives both, so dragging sweeps the shape across the canvas and changes its height at the same time. `0` makes the shape invisible. Negative and positive values push it in opposite directions. |
| **Show split line** | Draws a red dashed guide down the middle. Turn it off before exporting. |
| **Invert colors** | White dots on black instead of black on white. |
| **Copy / Download** | Export the canvas as a PNG, to the clipboard or as `stereogram.png`. |

In fullscreen (the ⛶ button at the bottom-right of the canvas) a **Zoom** slider appears, from 10% to 125%, scaled relative to fit-the-screen. 100% fills the viewport; above that the canvas is cropped rather than stretched. Press ✕ to exit.

## How it works

1. **Noise** — a box of black-or-white pixels is filled from a seeded PRNG (mulberry32), so output is reproducible.
2. **Mask** — the text is rendered onto an offscreen canvas the size of the full canvas, centred inside the right half and auto-shrunk if it would overflow 90% of that half's width, then read back as a 0–1 alpha mask.
3. **Blur** — the mask is softened with a separable box blur so depth ramps gradually at the glyph edges instead of stepping.
4. **Displace** — for every pixel of the right half, the mask is sampled at that position (offset by the slider) to get a depth value, and the pixel is copied from the base noise shifted by that amount. The source index wraps within the row so edge pixels don't drag in noise from the neighbouring scanline.
5. **Compose** — the untouched base noise is painted into the left half, the displaced copy into the right half.

Depth comes from the shear, not from the shape moving. That's why the outline never slides: the geometry that creates the illusion is the horizontal displacement between the two halves.

## Known limitations

- **Pattern width** and **Dot size** sliders are present but not wired to anything yet. They're placeholders for a future tiled-dot rendering mode.
- The depth map is built from a single text string; there's no image or custom-shape input.
- No persistence — settings reset on reload, and there's no URL state.
- The mask wrap-around means a large shift can pull the shape across the seam and back onto the other side.

## Possible directions

- Tiled dot patterns driven by the two idle sliders
- Uploaded depth maps or drawn shapes
- Anaglyph, wiggle, and top-bottom stereo output modes
- Shareable URLs encoding text, seed, and shift
