# ARPG devlog

I'm building an isometric action RPG in the Diablo II / Path of Exile lineage. Solo project,
written in C# on .NET with Godot 4 handling the rendering. The game's source is private for now,
so this repo is the public record of what it is and how it's put together.

![A caster burns a skeleton with a lightning skill converted to fire, then clears a torch-lit cave with chain lightning](images/caster-cave.gif)

| | |
|---|---|
| ![Melee against skeletons and a spider, then a Thundering Mouthpiece warcry item](images/warcry-field.gif) | ![Leaving town into the outskirts: skeleton archers, gold, and a Page of Frost Conversion](images/outskirts-loot.gif) |

## What's playable today

Character classes, click-to-move and WASD isometric combat, and a skill system where spells,
attacks, curses, buffs, and warcries combine through a socketed "page" system. Every attack
resolves through an accuracy-versus-evasion hit roll. Diablo-style loot with affix rolls, rarity
tiers, and identification. Monster packs with champion and rare ranks. A streaming multi-zone
overworld with towns, vendors, and a stash, an itemized map endgame, plus saves, a full settings and keybinding shell,
mouse-wheel zoom, and every piece of player-facing text running through translatable string
tables. Characters are now full human bodies that wear their armour piece by piece, capes
included, and the first licensed environment kits are in. Most of the world is still placeholder
art. Systems come first, art passes later.

![Town at evening, with a vendor and a portal home](images/town-evening.jpg)

| | |
|---|---|
| ![Inventory and equipment](images/inventory.jpg) | ![Character sheet and a magic weapon](images/character-sheet.jpg) |
| ![Lightning beam in a dungeon](images/dungeon-beam.jpg) | ![Chain lightning through a spider pack](images/dungeon-chain.jpg) |

| | |
|---|---|
| ![Cliff dressing](images/cliff-rock-dressing.png) | ![Ledge courses](images/ledge-courses.png) |

![Overworld at the cliff boundary](images/world-cliff-boundary.png)

## How it's built

The core decision: the entire game simulation is a plain .NET library with no engine dependency.
Fixed-point math instead of floats, one seeded RNG, a fixed timestep, and every mutation flowing
through commands. Godot renders what the sim says and never touches it. The same seed produces a
byte-identical world on every run, and a suite of 17,500+ simulation tests enforces that. One of
them runs the whole game twice from the same seed and fails if the state hashes ever differ. The
full suite used to take 73 minutes; an optimized test build got it down to about 5.

Other things I care about:

- Headless verification. The real game boots without a window and asserts on the rendered world
  itself: geometry, collision lining up with visuals, draw call budgets, occlusion behavior.
  Visual changes ship with screenshots that a person actually looks at.
- A Lua extension system where the base game is itself the first mod. Items, damage types, stats,
  tags, monsters, affixes, vendors, and classes are all Lua prototype data loaded through a
  sandboxed interpreter, so adding a new damage type takes zero engine changes. The sandbox works
  by removing capability instead of scanning for abuse: no filesystem, no OS access, no wall
  clock, and Lua's built-in `math.random` is deleted outright so a mod can't smuggle
  nondeterminism into the sim.
- 230+ architecture decision records. The docs update in the same commit as the code they
  describe, so they never drift.
- Performance budgets as part of the design rather than an afterthought: pooled VFX, shared
  materials, capped dynamic lights, and harness checks that fail the build when a budget is
  blown.

## The AI workflow

This project doubles as my testbed for agentic development at scale. I design, direct, and review.
AI coding agents implement, test, and review each other's work in parallel git worktrees, and
every change passes independent code, QA, and architecture reviews plus the full test suite before
it merges. Reviewers break the code on purpose to prove each new test actually catches its bug.
About 850 pull requests have gone through that pipeline so far, all solo. I ran the same workflow
at my last job and my measured ticket throughput went up 4.1x.

![Class select](images/class-select.png)

## 16-23 October 2026: patch 0.2.0, the first alpha build

The first alpha build went out on 23 October as patch 0.2.0. The week's work:

- Maps, the first endgame. A map is an item that stores a zone and an area level (1, then 5 to 80
  in steps of 5). You place it in the town Map Device to open a portal into a generated run.
  Dungeon maps use the dungeon layout; Cave maps are carved mazes of about 150 monsters. Kills
  inside a run drop more maps, about 3.5 per full clear on average, so a run can sustain the next
  one. The vendor has a Maps tab that always sells tier 1 Dungeon and Cave maps, plus one of each
  near your level.
- Inscriptions rebuilt from scratch. The two old ones are gone and eight new ones cover every gear
  slot. Some of them grant an aura that costs nothing and stays on while the item is equipped. An
  aura never stacks with a cast of the same buff; the higher level wins.
- Attack skills. Every class now starts with a dagger and a Basic Attack that adapts to the
  equipped weapon. Combat and Marksman Manuals grant attacks the same way spell books grant
  spells. The first two are Arrowfall, a bow volley that rains 12 arrows on the aim point, and
  Leap Slam, which hits and stuns everything around the landing. Weapon attacks can now crit at a
  150% base multiplier; spells still never do. Spells and ranged attacks no longer root you, so
  you cast and shoot while moving at half speed. Melee still roots.
- Items to item level 80. Every weapon, armour, shield, belt and crown line gets new tiers up to
  item level 80. Weapon base damage is cut from item level 25 up (item level 34-80 bases now deal
  70% of their old damage), and the flat damage prefixes on weapons are rescaled.
- A passive tree pass of about 120 new nodes across all three attributes, plus a search box that
  highlights every matching node.
- Smaller things: walk-in maze caves, ravine streams with plank bridges, graphics presets, a
  three-tab character sheet, and a long list of fixes.

On the engineering side the approach didn't change. All of it runs on the same deterministic,
fixed-point sim, and it went through the same agent workflow: test-first lanes that write a
failing test before the code, and cold code reviews where the reviewer sees only the diff and the
requirements, not the reasoning behind them. The sim suite is now at about 17,500 tests.

## Status

In active development; the first alpha build is out. Currently in the works:

- Monster pathfinding on a shared flow field, so packs route around buildings instead of getting
  stuck.
- A loot rework that rolls drops as a decision tree (currency or gear, then jewellery, armour, or
  weapon, and so on), so adding more item variants never skews what drops.

More notes as systems land.

---
*Nathaniel Whittaker · [GitHub](https://github.com/nathanielwhittaker) ·
[LinkedIn](https://www.linkedin.com/in/nathaniel-whittaker-0b2863323/)*
