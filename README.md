# De-Stressor for Photopea
 
A from-scratch Photopea plugin that mirrors what Texturelabs' **Distressor** panel
does for Photoshop: pick a distress style, drag a few sliders, and it erodes the
active layer's transparency to create a worn / weathered edge - non-destructively,
inside Photopea.
 
All 14 styles are generated
algorithmically at run time from a seeded noise engine, so the plugin is fully
self-contained and free to use/host/modify.
 
## Functionality
 
- **14 procedural distress styles** — Paper Grain, Cracked Paint, Rust, Concrete,
  Scratches, Splatter, Torn Edges, Dust & Grain, Halftone Grunge, Canvas Weave,
  Wood Grain, Ink Bleed, Static Noise, Organic Blotches — each with a live 64×64
  preview thumbnail.
- **Sliders**: Scale, Amount, Contrast, Edge Softness, Rotation, plus an Invert
  toggle — matching the "fine-tune with sliders" workflow of the original panel.
- **Randomize seed** to get a fresh variation of the current style instantly.
- **Copy / Paste settings** between layers in the same session, plus **named,
  saved presets** (stored in the browser via `localStorage`, so they persist
  across sessions on the same machine).
- **Replace original layer** or **keep it hidden underneath** the distressed
  result — your choice, per apply.
- Works at the document's actual resolution and is applied through the layer's
  alpha channel (transparency), the same mechanism Distressor itself uses,
  rather than tinting or blending colors on top.
## Logic
 
Photopea plugins are plain web pages loaded in an iframe inside Photopea. They
communicate by `postMessage`-ing a string of JavaScript to `window.parent`;
Photopea runs it against its own Photoshop-like scripting API (`app`,
`app.activeDocument`, `ArtLayer`, etc.) and streams back anything passed to
`app.echoToOE(...)`, followed by a final `"done"`. See
`photopea.com/api` and `github.com/photopea/photopea/issues/7937` for background
— Photopea's own docs on this are intentionally minimal, so this plugin sticks
to the documented, community-verified conventions (tag → duplicate/export via
`saveToOE("png")` → process pixels locally in the plugin with Canvas → reopen
the result with `app.open()` and `duplicate()` it back into your document).
 
Because the plugin does the actual pixel math itself (in a `<canvas>`, using
the layer's real pixels), the result is a real erosion of the artwork's alpha
channel — not just a semi-transparent texture laid on top.
 
## Instructions
 
1. Open an image in Photopea and select the layer you want to distress.
2. Open **Window → Plugins → De-Stressor**.
3. Pick a style thumbnail, adjust Scale / Amount / Contrast / Edge Softness /
   Rotation, optionally toggle Invert.
4. Click **Apply distress**. The plugin reads the layer's pixels, generates the
   mask, erodes the alpha channel, and sends a new layer named
   `<original name> (Distressed)` back into your document — replacing or
   hiding the original, per the "Replace original layer" toggle.
5. Tweak sliders and click Apply again to iterate; use **Randomize seed** for
   variation within the same style; use **Copy/Paste settings** or **Save as…**
   to reuse a look across layers or documents.
   
## Notes & limitations
 
- This targets simple raster layers. Layers inside groups, text layers, and
  smart objects aren't specifically handled — flatten or rasterize first if
  you hit an error.
- Very large documents (e.g. multi-thousand-pixel canvases) will be slower,
  since the mask is generated at full document resolution and the round trip
  goes through PNG-encoded data URIs.
