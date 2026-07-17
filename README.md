# 🎨 Game Assets Enhancer — a Claude Code skill

Turn a functional-but-plain game into one that looks hand-crafted.

Point Claude Code at any game codebase (or a brand-new game) and this skill will
derive a style guide, generate every visual element as its own individual sprite
via [fal.ai](https://fal.ai), replace flat placeholder art, and rebuild the scene
as multi-layer parallax with procedurally-placed sprites — sky, clouds, hills,
buildings, playfield, foreground. Typical run: **about 20 minutes** from invoking
the skill to a transformed game.

<!-- TODO: before/after screenshots
![before](assets/before.png)
![after](assets/after.png)
-->

Built and battle-tested while making my own iOS games with Claude Fable 5 —
I share the full story on my YouTube channel.
<!-- TODO: link ep 120 when live -->

## What it actually does

Three jobs, usually run in order:

1. **Style guide** — derives (or defines) the game's visual language and writes
   it into a `STYLE.md`, so every asset generated afterwards matches. Includes
   lighting rules (single light direction, shared warm undertone, aerial
   perspective) — the difference between "cohesive game" and "clipart".
2. **Asset generation** — every visual element becomes its **own transparent
   sprite**, generated against a single style anchor, background-removed,
   trimmed, and footprint-normalized. Character animations get real frame
   cycles (4+ frames, all edited from frame 1 so they don't drift). Never
   composite images, never baked text.
3. **Depth & parallax** — rebuilds the scene as layered parallax driven by one
   world clock, with procedural slot-based placement (deterministic hashing —
   scenery reappears if you backtrack), foreground whip layers, and a cheap
   code "light grade" that fuses the layers into one lit world. Ends with a
   polish pass: landing squash, input snap, idle life on the focal element.

When you later edit a sprite, the skill modifies just that sprite — each object
in the game is a separate image, so you can modify, duplicate or add sprites
without regenerating the rest of the scene.

## Requirements

- **[Claude Code](https://claude.com/claude-code)** — developed and tested with
  **Claude Fable 5**. It should work on other models but expect more
  hand-holding; one-shotting a full graphics overhaul in a single 20-minute run
  is a Fable-tier trick.
- **A [fal.ai](https://fal.ai) account + API key** — asset generation runs
  through the fal.ai MCP server. The skill uses:
  - `fal-ai/nano-banana-pro` for sprite generation and style-referenced edits
  - `pixelcut/background-removal` for cutouts
- **The fal.ai MCP connected to Claude Code** — follow the MCP setup
  instructions in the [fal.ai docs](https://docs.fal.ai). ⚠️ **Tip:** add
  `--scope user` to the end of the `claude mcp add` command, otherwise the MCP
  is only registered for the current folder and won't be available in your
  other projects.

💸 **Cost note:** generation costs real money on your fal.ai account. A full
game overhaul (30–60 sprites plus a few edit iterations) typically lands in the
single-digit dollars, but iterating heavily on style will cost more. The skill
is written to reuse and edit sprites rather than regenerate wherever possible.

## Installation

**As a personal skill (available in every project):**

```bash
git clone https://github.com/adamlyttleapps/claude-skill-game-assets-enhancer.git \
  ~/.claude/skills/game-assets-enhancement
```

**Or as a project skill (checked into one repo):**

```bash
git clone https://github.com/adamlyttleapps/claude-skill-game-assets-enhancer.git \
  .claude/skills/game-assets-enhancement
```

That's it — Claude Code picks up the skill automatically. Verify with
`/skills` or just ask Claude to use it.

## Usage

Open Claude Code inside your game project and prompt something like:

> use the game-assets-enhancement skill to upgrade the graphics of this game

Or for a new game:

> build me a chrome-dinosaur style endless runner, then apply the
> game-assets-enhancement skill to make it look hand-crafted

Useful follow-ups once it's run:

> restyle the whole game to SNES-era pixel art using the skill's relight-in-place recipe

> the tree sprite is missing its trunk — regenerate just that sprite

What to expect during a run: Claude audits your visuals, writes a sprite
manifest and `STYLE.md`, generates a style-anchor sprite first, then works
through the manifest (each sprite generated → background-removed → trimmed),
backs up your original art to `art/originals/`, rebuilds the scene as parallax
layers, and finishes by verifying with screenshots and a scrolling capture.

## Notes & limitations

- **A default art style is baked in** — hand-painted cartoon: thick warm
  outlines, saturated colors, soft painterly shading (clean-line anime with
  watercolor-style backgrounds). The skill derives the style from your existing
  art when the game already has a look worth keeping, and you can override the
  style sentence in `STYLE.md` — the *process* is style-agnostic.
- **Engine-agnostic recipes, iOS-first testing.** I built this for my iOS games
  (SwiftUI + SpriteKit). The recipes are written for any 2D stack — HTML
  canvas/JS, Phaser, Unity 2D, Godot — but I haven't personally verified WebGL
  / three.js projects. Play around and show me what you build.
- **Results vary between runs** — that's generative art. The pipeline is built
  around a style anchor and edit-with-reference precisely to keep a whole set
  cohesive, but expect to regenerate the odd sprite.

## License

[MIT](LICENSE) — do whatever you like, attribution appreciated.

Built by [Adam Lyttle](https://adamlyttleapps.com).
<!-- TODO: add YouTube + X links before publishing -->
