---
name: game-assets-enhancement
description: Point at any game codebase (or a new game) to dramatically upgrade its graphics — derive a style guide, generate individual sprite assets via fal.ai, replace flat/placeholder art, and add multi-layer parallax depth (sky / clouds / hills / buildings / playfield / foreground) with procedurally-placed individual sprites, never baked looping strips.
---

# Game Assets Enhancement

Turn a functional-but-plain game into one that looks hand-crafted. This skill
covers three jobs, usually run in order:

1. **Style guide** — derive (or define) the game's visual language and write it
   down so every asset generated afterward matches.
2. **Asset generation** — generate every visual element as its **own individual
   sprite** through the fal.ai pipeline, styled against a single anchor.
3. **Depth & parallax** — rebuild the scene as layered, procedurally-dressed
   parallax so the world has depth (side-scrollers especially), plus a juice
   pass of cheap code-level polish.

Works on an existing codebase (audit → enhance in place) or a brand-new game
(define style first, build layered from day one). Engine-agnostic: the recipes
apply to HTML canvas / JS, Phaser, SwiftUI/SpriteKit, Unity 2D, Godot — only
the integration syntax changes.

---

## Phase 0 — Audit (existing codebase only)

Before generating anything:

1. **Inventory the visuals.** Find every drawn element: backgrounds, player,
   enemies, obstacles, pickups, UI chrome, particles. Note which are code-drawn
   shapes (rects/circles), which are placeholder images, and which are baked
   multi-object images that need splitting.
2. **Map the scene graph.** How is the background drawn? One image? A looping
   strip? Is there any layering at all? Where's the render order defined?
3. **Find the world clock.** Identify (or plan) the single source of truth for
   world scroll position — parallax hangs off exactly one clock.
4. **List the sprite manifest.** Write out every asset you'll generate, one
   line each, grouped by layer. This manifest drives the whole job — every
   entry becomes its own file. If the current art has a row of buildings baked
   into one image, the manifest lists *each building* separately.

Back up any existing art before replacing it (e.g. copy to `art/originals/`).

---

## Phase 1 — Style guide

Write a short `STYLE.md` (or a styles section) for the project before
generating asset #1. Every later prompt quotes from it. If the game already
has a look worth keeping, derive the guide from its best existing sprite; if
not, define one fresh. A proven default that reads great for casual games:

- **Hand-painted cartoon:** thick warm (brown) outlines, saturated
  kid-friendly colors, soft painterly shading. No flat vector, no
  gradients-as-style, no photo textures. (Swap this line for the game's own
  style if it has one — pixel art, flat-shade, etc. — the *process* below is
  the same.)
- **Palette anchors:** pick 5–7 named colors with jobs, e.g. warm wood browns,
  a teal/sea-green secondary, cherry red for danger, leafy green for success,
  gold/amber for score & celebration, cream for cards/UI. Write hex/RGB values
  down.
- **Semantic color is sacred:** red = danger/wrong, green = correct/go,
  amber = time, gold = celebration. Never swap these.
- **Focal hierarchy:** the background must be *calmer* than the playfield —
  muted detail, a light pool at the play area's center, and a code vignette
  (radial gradient, clear → black ~0.30 opacity, centered on the action). If
  the background competes, dim the background — don't brighten the subject.

### The lighting rules (what separates "cohesive" from "clipart")

Raw, unmodulated hues read cheap no matter how nice the shapes are. Every
sprite in the game obeys the same five points:

1. **Shared warm undertone** in every color — the whole game is "warm light
   falling on saturated things".
2. **One light direction** — pick where the sun/lamp is and light every sprite
   to it: bright highlight planes on the lit side, crisp cool shadow shapes
   opposite, a thin warm rim light.
3. **Real value range** — distinct shadow and highlight *shapes*, not one flat
   mid-tone (anime-style cel shading, not airbrush mush).
4. **Aerial perspective** — the farther back a parallax layer sits, the
   lighter, hazier, and more desaturated its sprites. This alone sells depth.
5. **A code light grade** — a low-opacity warm radial wash from the light
   source plus the gentle vignette, drawn over the composed scene. Cheap, and
   it fuses separately-generated sprites into one lit world.

---

## Phase 2 — Asset generation pipeline (fal.ai MCP)

### The cardinal rule: one sprite = one object

Never generate a composite. Not a "row of buildings", not a "forest strip",
not a "background with trees". If the manifest says skyline, generate
building-1, building-2, building-3… as separate assets, each on its own
transparent canvas, and let *code* compose them procedurally (Phase 3). This
is what makes the world regenerable, recolorable, resizable, and non-repeating
piece by piece.

### Pipeline steps

1. **Generate the style anchor first.** Pick the single most important sprite
   (usually the player or the hero object) and iterate on it until it nails
   the style guide. Everything else is generated *against* it.
2. **Generate with `fal-ai/nano-banana-pro`** — for every subsequent asset,
   ALWAYS pass the anchor (or the best sprite of the same set) via `/edit`
   (`image_urls`) as the style reference: *"using the exact same hand-painted
   cartoon style as this…"*. Edits preserve composition, so tuned layouts
   survive recolors and relights. Chain the first good result of a set as the
   style anchor for the rest of that set.
3. **Cut out with `pixelcut/background-removal`**, `sync_mode: false` always.
   Edited sprites come back flattened — re-cutout every time, even after a
   small edit.
4. **Trim to the alpha bounding box and downscale.** ~512px on the long edge
   is plenty for most in-game sprites; go larger only for full-screen
   backdrops.
5. **Interchangeable set pieces share one footprint.** Variants that slot into
   the same spot (buildings in the skyline slot, trees in the treeline slot)
   all render at one fixed dimension on the shared axis so any of them fits
   flush; vary the *other* axis by kind (buildings vary height, never base
   width). Composite paired icons onto identical canvases so they render at
   the same scale in code.
6. **Moving parts are separate sprites.** Anything that animates independently
   (a wheel, a flag, a door, a character's held item) is its own asset, and
   the parent is generated WITHOUT it — prompt *"empty space where the X would
   go"*.
7. **Animated characters get a real frame cycle, not a two-frame swap.** A
   running/walking/flapping character needs a proper cycle — for a run: 4
   frames minimum (contact → pass → opposite contact → opposite pass).
   Generate frame 1 first and approve it, then generate EVERY other frame as
   an edit **of frame 1** (not chained frame-to-frame, which drifts), with a
   prompt that pins everything: *"keep the character ABSOLUTELY IDENTICAL —
   same head angle, same expression, same arms, same tail, same colors, same
   size and position on the canvas. ONLY redraw the legs: this is the passing
   frame — both legs gathered under the body, near knee bent high…"*. Naming
   only the moving limbs is what keeps the rest stable across frames.
   Normalize the whole cycle onto one shared canvas (bottom-anchored for
   ground characters) so frames don't jump, and key frame advance to **world
   travel** (distance moved), not wall time, so the cycle always matches
   ground speed. Pick one frame (usually a tucked pass pose) as the held
   airborne/jump pose.
8. **No text zones baked into sprites.** No sign plates, labels, or blank
   panels waiting for numbers — bodies stay clean and ONE code-rendered
   card/label carries any words. Baked plates double up with the UI, can't
   animate, and lock layout to artwork.
9. **Reuse beats regenerate.** Mirror + hue-shift one arrow instead of
   generating a second "similar" arrow — pixel-identical components read as
   deliberate design; near-duplicates read as sloppiness.
10. **Relight in place to restyle a set.** To change the whole game's mood,
   edit each existing sprite ("same building, same shapes, but lit by warm sun
   from the upper right…") — composition survives, so no layout retuning.

11. **Big soft-edged set pieces (mountains, fog banks, clouds) must be
    generated as ISOLATED silhouettes** — prompt *"plain solid white
    background all around; the landform must not touch the left or right
    canvas edges; only the flat bottom edge is straight"*. Painted aerial
    haze that runs to the canvas edge survives background removal as a
    semi-opaque RECTANGLE, and the sprite renders as a hard-edged slab in
    the scene. If a generation comes back haze-to-edge, don't try to rescue
    it in processing — regenerate with the isolation wording (chain from a
    correctly-composed sibling if one exists). As a safety net when
    processing hazy sprites: dissolve near-white pixels (alpha ×
    `(242 − min(r,g,b)) / 30`, clamped) and feather the outer ~9% of the
    left/right edges to zero alpha so soft slopes dissolve instead of
    ending in a wall.

### Manifest order that works

Generate in this order so each layer can reference the ones before it:
style anchor → playfield objects → background set pieces (per-layer, far to
near, applying aerial-perspective desaturation as you go) → foreground props →
UI/affordance sprites (arrows, guides, icons — painted in-style, not system
icon fonts).

---

## Phase 3 — Scene construction & parallax

### The layer stack

Rebuild the scene as ordered layers, back to front. For a side-scroller the
canonical stack (with scroll factors relative to the playfield = 1.0):

| # | Layer | Factor | Content |
|---|---|---|---|
| 1 | Sky | 0–0.05 | Painted gradient/backdrop; essentially fixed |
| 2 | Clouds | ~0.1 | Individual cloud sprites, plus their own slow self-drift |
| 3a | **Far hills / mountains** (required) | 0.10–0.15 | Big hazy desaturated range pieces, near-continuous chain |
| 3b | **Near hills / mountains** (required) | 0.25–0.35 | Warmer, more contrast, still calmer than the playfield |
| 4 | Buildings / trees (mid) | 0.4–0.6 | Individual set-piece sprites, one footprint per slot family |
| 5 | Playfield / ground plane | **1.0** | The gameplay: player, obstacles, pickups, ground |
| 6 | Foreground whip layer | 1.5–1.8 | Occasional props that sweep past close to camera, slightly blurred |

**The hills are not optional decoration — they ARE the depth.** Every outdoor
scrolling scene includes the two hill/mountain layers, and they should be
**dominant**: sprite heights around 0.30–0.50 of screen height for the far
range and 0.22–0.36 for the near ridge, with LOW skip rates and spacing well
under a sprite width so pieces overlap into continuous ranges rather than
isolated bumps on the horizon. Two ranges sliding at different factors is
what sells parallax; one timid strip of hills reads flat. Differentiate them
with aerial perspective: far = palest, haziest, least contrast; near = warmer
and more detailed but still visibly calmer than the playfield.

**Readability rule: the background must never impersonate an obstacle.** If a
sprite family is used as (or resembles) something the player must react to —
the cacti they jump, the rocks they dodge — it must NOT appear in any
background or mid layer at any size: a small background cactus reads as a
thing to jump and causes false inputs. Dress background layers only with
shapes the gameplay never uses (mountains, dry trees, ruins, clouds). The
foreground whip layer is the one exception: obstacle-family props are fine
there because big + blurred + faster-than-gameplay reads as camera depth, not
threat — the blur is mandatory for that reason.

For a non-scrolling game the same stack still applies as static depth:
backdrop → object layer → foreground framing props that crop off the screen
edge (depth without 3D). In all cases nothing interactive is ever baked into
the background, and every playfield sprite is its own element — its own asset,
its own view/node, its own layout anchor — so any piece can be regenerated,
resized, or repositioned without touching the rest.

### Parallax rules

- **One constant world clock** drives every layer's offset
  (`offset = worldX × factor`). The drift never speeds up, slows down, or
  jumps for any game event — only the focal element moves on events. A steady
  world is what makes the moving parts read.
- **Mid-scene dressing is individual sprites placed procedurally** — never one
  giant looping strip. Strip loops need mirror-tiling and the mirrored twins
  are visible. Mirror-tiling is acceptable ONLY for genuinely uniform textures
  (a plain ground fill, a flat wall).

  Procedural placement recipe:
  - The world is divided into **slots** per layer (e.g. a building slot every
    ~220 world units).
  - `hash(layerId, slotIndex)` — any cheap deterministic integer hash — picks
    the variant, the spacing jitter (±30–40% of slot width), the size wobble
    (±10%), and an occasional skip (empty slot). Deterministic means the same
    scenery reappears if the player backtracks, with zero storage.
  - Spawn slots a screen ahead of the camera, cull a screen behind. Never
    store the whole world.
  - Adjacent slots avoid repeating the same variant (re-hash or +1 the pick).
- **Foreground props sweep past occasionally** — roughly one per long cycle,
  not a constant stream — with a slight blur for depth of field, and they
  NEVER hit-test. Gameplay input is always the topmost interactive layer.
- **Layers resize from their anchored seam.** A strip pinned to the horizon
  grows UPWARD when scaled — shared edges between layers must never move, or
  gaps flash at the seams on resize.
- **Aerial perspective per layer** (from the style guide): far layers get the
  lighter/hazier/desaturated versions of the sprites. If needed, apply a cheap
  per-layer tint/haze overlay in code instead of regenerating.
- **Camera = a focus value, not teleports.** Interpolate the camera/world
  position along a focus index so rolling/walking parts can read the
  *interpolated* travel each frame and match ground speed exactly (in SwiftUI
  that means `Animatable`; in a canvas loop it's just per-frame lerp).

### Polish pass (cheap, high-impact)

- **Never open static.** A scene whose fantasy has motion opens already
  moving (ease-out, no delay) — the player must never see the world "start
  up".
- **Idle life on the focal element** — a gentle bob, an ambient particle loop —
  with ground-contact parts staying planted.
- **The code light grade** from Phase 1: warm radial wash + vignette over the
  composed scene. Do this last; it's what fuses the layers.
- **Landings never overshoot.** Anything dropping onto a surface: ease-in fall
  that stops DEAD at contact, a quick squash (≈0.78 wide / 1.12 tall, anchored
  at the bottom, ~0.07s), a dust burst at the contact seam (anchor the burst
  slightly BELOW the visual seam — burst patterns fan upward), then spring
  back.
- **Reactions are layered:** instant snap within one frame of input
  (~0.04s) → rattle/jitter → brightness flash → recover. Input registers on
  touch-down/press, never on release, and the first visual reaction lands
  within one frame.
- **Settled things freeze forever.** Physics debris drops, tumbles, settles —
  then becomes immovable. Nothing idles, twirls, or jitters in the background
  competing with the playfield.

---

## Phase 4 — Integration & layout conventions

- **All positions are fractions of the viewport/scene size** in a per-scene
  `Layout` constant table — never absolute pixels. The playfield focal point
  sits around `(0.5 × width, 0.55–0.72 × height)` for portrait; center-weight
  horizontally for landscape scrollers.
- **One sprite, one node/view, one layout anchor.** The file structure mirrors
  the manifest.
- **Render order is the layer stack**, explicit and in one place — not
  emergent from insertion order scattered across files.
- **Time bases live in state, not locals.** A phase anchor or start-time
  captured as a plain local in render code resets on every re-render and snaps
  animation clocks to zero (in SwiftUI: `@State`; in JS: module/instance
  state, not per-frame `Date.now()` diffs from a re-created base).
- Web-specific: preload the sprite manifest before first paint, use
  `image-rendering` appropriate to the style, draw layers to the main canvas
  in stack order (or use stacked canvases/CSS transforms for the far layers if
  the game is DOM-based).

---

## Verification

Don't call it done from stills alone:

1. **Screenshot** each layer toggled on cumulatively (sky → +clouds → +hills →
   +buildings → +playfield → +foreground) to check seams, scale, and aerial
   perspective steps.
2. **Record video / animated capture** of the world scrolling — stills cannot
   judge parallax feel, easing, or drift steadiness. Watch for: mirrored
   twins, variant repeats in adjacent slots, seam gaps on resize, foreground
   props stealing attention.
3. **Squint test:** blur your eyes at a screenshot — the playfield focal point
   must still be the obvious brightest/highest-contrast region. If a
   background layer wins, dim it.

---

## Checklist

1. [ ] Visual inventory + sprite manifest written (every object its own line)
2. [ ] `STYLE.md`: style sentence, palette anchors with values, semantic
       colors, light direction
3. [ ] Style anchor sprite generated and approved
4. [ ] All manifest sprites generated via the pipeline (edit-with-reference →
       background-removal → trim → footprint-normalized), originals backed up
5. [ ] No composite sprites anywhere — every building/tree/cloud/prop is its
       own cutout
6. [ ] No baked text/label zones in any sprite
7. [ ] Layer stack implemented with per-layer scroll factors off ONE world
       clock — BOTH hill/mountain layers present and dominant, no
       obstacle-lookalike sprites in any background or mid layer
8. [ ] Animated characters have a full frame cycle (4+ frames for a run),
       frames edited from frame 1, shared canvas, keyed to world travel
9. [ ] Mid/far dressing placed procedurally (slot + hash: variant, jitter,
       size, skip), spawn-ahead/cull-behind
10. [ ] Foreground whip layer: occasional, blurred, never hit-tests
11. [ ] Aerial perspective across layers; code light grade (warm wash +
        vignette) over the composed scene
12. [ ] Opens already in motion; idle life on the focal element; landing
        squash recipe wherever things drop
13. [ ] Layout as viewport fractions; seams anchored; resize-safe
14. [ ] Verified with layer-by-layer screenshots AND a scrolling video capture
