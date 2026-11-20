[![dot-procedural-generation - GameSpice addon](docs/banner.png)](https://moddingcommunity.com/a/242-dot-procedural-generation)

Generated levels as an ordered pipeline of deterministic steps, with a validator that rejects maps nobody could finish.

The pipeline produces a document (`DotProcGenDoc`: a grid, rooms, links and markers), not nodes. Your game turns the document into a world. This means:

- Two machines can agree on a map by comparing a seed and `fingerprint()`.
- "Is it finishable?" is a flood fill you can ask.
- One generator serves 2D (tiles) and 3D (extruded).
- Generation can be spread over several frames.

## Install

1. Copy `addons/dot_procedural_generation/` and [dot-core](https://github.com/gamespice-platform/core)'s `addons/dot_core/` into your project.
2. Enable it in *Project > Project Settings > Plugins*.

Only dot-core is required.

## Use

```gdscript
var p := DotProcGenPipeline.new()
p.width = 64
p.height = 64
p.add(rooms).add(corridors).add(spawns).add(validate)
p.seed_source = rng                       # optional; takes the session's seed

var res := p.generate(seed)
if res.ok:
    build(res.value as DotProcGenDoc)
```

To spread the work over frames (on the main thread, so it works in web exports without threads):

```gdscript
runner.pipeline = p
runner.finished.connect(_build)
runner.begin(seed)
```

Sliced and blocking runs give exactly the same map.

## Steps

| Step | Purpose |
| --- | --- |
| `DotProcGenRooms` | Places rooms. |
| `DotProcGenCave` | Cellular-automata caves. |
| `DotProcGenConnect` | Joins rooms with a minimum spanning tree, then adds `extra_loops` more corridors so the level has loops. |
| `DotProcGenMarkers` | Places spawns, objectives and other markers. |
| `DotProcGenValidate` | Refuses unplayable maps. |

## The validator

Caves can seal off pockets, and a scattered objective can land in a room with no door. The validator catches this.

```gdscript
var check := DotProcGenValidate.new()
check.from_kind = &"spawn"
check.min_connected_fraction = 0.6
check.fill_unreachable = true          # usually better than failing
```

A failure is not a crash: the pipeline retries with another seed, up to `max_attempts` (default 10). The attempt number is mixed into the seed, so seed 41 never silently produces the map of seed 42.

## Random streams

Each step draws from its own named stream (`DotProcGenRandom` hashes the name). Adding a step does not change the output of steps before it, so shared seeds keep producing the same map. A game's own randomness addon supplies only the seed, through `seed_source`.

## Debugging

`to_ascii()` prints the map as text. Print it when a check fails.

See [docs/design.md](docs/design.md) and [docs/history.md](docs/history.md). The self-test is `examples/procgen_selftest.tscn`.

## Status and licence

Part of the [Gamespice](https://gamespice.net) platform: a family of Godot addons. Written with Claude Code and maintained by `gamemann`. Every addon has headless tests, but little of this has been played by real users yet, so expect rough edges and please [open an issue](https://github.com/gamespice-platform/procedural-generation/issues) when you hit one.

MIT licence, see [LICENSE](LICENSE). The only closed part of the [Gamespice](https://gamespice.net) platform is the web back end; you can use your own instead of Gamespice's.
