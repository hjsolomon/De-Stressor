# De-Stressor for Photopea

A Photopea plugin for texture-driven distressing, in the spirit of Texturelabs'
**Distressor** panel for Photoshop: load a grunge texture, grab a layer, dial in
the wear on a live preview, and apply. The texture eats away the layer's
transparency, with extra wear concentrated along the artwork's own edges.

No textures are bundled. You load your own: scans you've made, or free
textures you're licensed to use (for example CC0 sets from ambientCG or
Poly Haven, or Texturelabs' free downloads under their licence). Loaded
textures are stored in your browser (IndexedDB) so they're there next time.

## Instructions

1. Open an image in Photopea and select the pixel layer you want to distress.
2. Open **Window → Plugins → De-Stressor**.
3. **Texture:** click **+** (or drop image files on the panel) to load one or
   more textures, then click one to use it.
4. **Grab layer:** pulls the selected layer into the panel's preview.
5. Adjust the sliders and watch the preview. **Show mask** toggles between the
   result and the black/white wear mask.
6. **Apply distress:** processes at full resolution and adds
   `<name> (Distressed)` above the original, replacing or hiding the original.

## Controls

| Control | What it does |
| --- | --- |
| **Amount** | Wear across the whole surface. Roughly the fraction of it that's removed, whatever texture you use. |
| **Edge wear** | Extra wear near the layer's edges, where real wear happens. |
| **Edge reach** | How far (in document pixels) edge wear reaches into the shape. |
| **Softness** | Hard, crisp breaks vs. soft, faded transitions. |
| **Scale** | Texture size. 100% = the texture just covers the document; smaller values tile it (mirrored, so no seams). |
| **Rotation** | Texture rotation. |
| **Invert texture** | Swaps which tones of the texture wear away (dark by default). |
| **Shuffle / Reset placement** | Randomly offsets and flips the texture for a new variation, or puts it back. |
| **Replace original layer** | Delete the original instead of hiding it. |
| **Output mask only** | Adds the wear mask as a grayscale layer instead (original untouched), so you can use it as a layer mask yourself. |

Copy/paste and named presets save all settings, including which texture is selected.

## How it works

Photopea plugins are web pages in an iframe that `postMessage` scripts to
Photopea and receive `app.echoToOE(...)` / `saveToOE(...)` results back.

1. **Export just the layer.** The plugin briefly hides every other layer,
   exports the document as PNG, then restores each layer's visibility and
   opacity exactly as it was, so other layers don't leak in.
2. **Texture field.** The texture is placed (scale, rotation, offset, mirrored
   tiling) at document size, converted to luminance and histogram-equalized,
   so Amount behaves the same for light and dark textures.
3. **Edge proximity.** An exact Euclidean distance transform of the layer's
   alpha gives every pixel its distance to the nearest edge.
4. **Erosion.** Each pixel keeps `smoothstep(thr ± softness, texture)` of its
   alpha, where `thr = amount + edgeWear × proximity`.
5. The result is opened in Photopea and copied back into your document above
   the original, keeping the original's opacity and blend mode.

The preview runs the same code at reduced resolution. Apply re-exports the
layer and processes it at full size (about 3–4 s for a 4000×4000 layer).

## Notes & limitations

- Works on plain pixel layers. Rasterize text, shapes and smart objects first.
- The result is baked into a new layer's transparency. To keep things
  editable, use **Output mask only** and apply the mask yourself.
