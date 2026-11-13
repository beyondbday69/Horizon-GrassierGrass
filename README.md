# Grassier Grass — Horizon native port

A native Rust port of [Grassier Grass](https://modrinth.com/mod/grassier-grass) 1.4.5 by
[leonardoinc22](https://modrinth.com/user/leonardoinc22), built as a **feature of the Horizon
Android client** rather than as a standalone mod.

Bakes grass blades on the grass blocks around the camera, sways them with wind, tints them to
the biome, and parts them for the player who walks through them.

> **Licence: All Rights Reserved.** The original is ARR and so is this port. It is published
> here with the author's written permission for the Horizon Android client, and that permission
> does not extend to anyone else. See [Licence](#licence). If you want to reuse this code, ask
> the author.

---

## What this repo is

Two things:

| Path | What it is |
|---|---|
| `grassiergrass-engine/` | The CPU blade mesher, game-free, buildable and testable on its own |
| `grassiergrass.hnmod` | The bundle that declares the feature to Horizon |

### The `.hnmod` is not a working mod by itself

This is the important part, and it is easy to get wrong.

`grassiergrass.hnmod` is a ~1 KB **marker**. It carries `horizon-native.json`,
`horizon-compiled.json` and a `src/lib.rs` stub whose export is a title and a description. It
registers the feature id `grassiergrass` with Horizon so the game turns the code on and shows a
row under *Options → Mods*.

The code that actually bakes and draws blades is **not** in the bundle. It is compiled into the
Horizon client itself, across `crates/horizon-client/src/game/grassiergrass/` (the renderer and
settings) and `crates/grassiergrass-engine/` (the mesher, mirrored here).

So: installing the `.hnmod` on a stock Horizon build does nothing. It only means something in a
client that already contains this feature. It is published so the feature is visibly declared
and versioned, not because dropping it in adds grass.

## Requirements

- A **Horizon Android dev build** with this feature compiled in
- Android, arm64-v8a
- Enable it after installing: **Options → Mods → Grassier Grass → enable** (mods are off by
  default)

## Settings

All under *Options → Mods → Grassier Grass*. Tap a row to step through its range; each value is
stored per key and survives a restart.

| Setting | Range | Default |
|---|---|---|
| Enabled | on / off | on |
| Radius | 8–48 blocks | 20 |
| Density | 8–64 blades per block, in 4s | 24 |
| Height | 0.10–0.60 | 0.30 |
| Width | 0.50–1.60 | 0.95 |
| Biome tint | on / off | on |

A change is applied on the **next bake**, which happens when the camera moves two blocks
horizontally or one vertically. Standing still after changing a setting shows the old mesh until
you move.

## What works

- Blades on `minecraft:grass_block`, only on a block whose space above is air
- Wind: the original's direction wander, flutter and noise scroll
- **Biome tint** — each column blends the biomes of the six quart cells around it with the game's
  own gaussian kernel, the same blend the terrain uses
- **Interaction** — blades within 1.6 blocks lean away from the player, more the closer they
  are, root planted. Walking through grass brushes it aside instead of clipping through you
- Depth-tested against terrain, so blades are hidden by blocks in front of them

## What does not

A deliberate vertical slice, not feature parity:

- `minecraft:grass_block` only — no short grass, tall grass, flowers, cactus or snow
- No LOD rings, clumps or entity trails
- Procedural WGSL noise instead of the original's noise textures, so the pattern is similar in
  character but not identical
- No Iris/Sodium integration
- Block edits are not watched; an edited block updates on the next camera move, not immediately

## Build and test

```sh
cargo test -p grassiergrass-engine
```

Nine unit tests cover the mesher's block classification, column scanning, blade placement and
determinism. Nothing else in this repo builds on its own — the renderer needs the Horizon client.

## How it fits into Horizon

The feature is gated the same way a bundled mod is: it draws only when `native_on("grassiergrass")`
is true and the loaded bundle lists the `grassiergrass` feature. The renderer is installed once at
startup, before settings are read, so the gate is checked per frame rather than at install time.

Three adaptations were needed because this tree's mod API differs from the revision the original
bundle pinned:

- **No `Host::client_state()`.** World reads are only available in `frame`, so the render pass
  records the camera and `frame` bakes around it. The anchor lags the camera by one frame, which
  the two-block trigger hides.
- **The dimension is a string**, not a `NativeWorldId`.
- **`RenderNeeds`, the depth attachment and the blend state** are this tree's.

## Licence

**All Rights Reserved.** Grassier Grass and this port are both ARR. See
[`grassiergrass-engine/LICENSE`](grassiergrass-engine/LICENSE).

Credits:

- Original mod, concept, shader and blade geometry: **leonardoinc22**
- Horizon native port: **beyondbday69**, with leonardoinc22's written permission

Only this clean rewrite is distributed. The original jar, its decompiled Java and its assets are
not included, and were not used as a source.

Not affiliated with leonardoinc22, Mojang, or Microsoft.
