# Shooter assets

Low-poly GLB models for a top-down 3D shooter, ready for three.js (`GLTFLoader`).
Picked from two Kenney packs, both CC0 (public domain): credit "Kenney (www.kenney.nl)"
is appreciated, not required.

- `characters/`: [Mini Characters 2.0](https://kenney.nl/assets/mini-characters), 8 characters and a blaster
- `scenery/`: [Prototype Kit 1.0](https://kenney.nl/assets/prototype-kit), floor, walls, cover and markers

Every value below was measured from these files.

## Conventions

- **Units:** 1 unit = 1 grid tile. Y is up. Every origin is at the bottom center of the model.
- **Facing:** characters face **+Z**. To face a direction `(dx, dz)` on the ground plane, set
  `model.rotation.y = Math.atan2(dx, dz)`.
- **Textures:** each folder has its own `Textures/colormap.png`, loaded by a relative path from
  inside the GLB. The two files differ, so keep both folders as they are and don't merge them.
  Colors come from that one atlas: tint with a material color or an emissive flash, not new textures.
- **Shadows:** flat faces show shadow acne with three.js defaults. Set
  `light.shadow.bias = -0.0005` and `light.shadow.normalBias = 0.02`.

## Characters

`characters/character-<name>.glb`: `soldier`, `raider`, `robot`, `alien`, `knight`, `worker`,
`zombie`, `male-c` (a police officer). One per player seat.

- 0.67 tall (zombie) to 0.84 (robot, with its antenna), most about 0.75. A circle of radius 0.3
  covers the body.
- All 8 share one rig. Its joints are `root`, `torso`, `head`, `arm-left`, `arm-right`,
  `leg-left` and `leg-right`, with no hand bones.
- **Clone with `SkeletonUtils.clone()`** (`three/addons/utils/SkeletonUtils.js`). A plain
  `.clone()` of a skinned model shares the skeleton, so every copy plays the same pose.
- There is no horizontal root motion: `walk`, `sprint` and `die` only bob `root` up and down, so
  place the model's root at the game position.

Clips for a shooter (each file has 32 clips):

| Clip | Length | Animates | Use |
|---|---|---|---|
| `idle` | 1.33 s loop | torso, arms, head | standing |
| `walk` | 0.67 s loop | whole body | moving |
| `sprint` | 0.50 s loop | whole body | moving fast |
| `holding-right` | 0.17 s, a pose | **arms only** | blaster held out, layered over the above |
| `holding-right-shoot` | 0.20 s | torso, arms, head | recoil: play once per shot, `LoopOnce`; starts and ends on the `holding-right` pose |
| `die` | 0.33 s | whole body | `LoopOnce` + `clampWhenFinished` |
| `emote-yes` | 0.67 s | upper body | victory |

`idle`, `walk` and `sprint` animate the arms too, so a locomotion clip running alongside
`holding-right` makes the gun arm swing. Strip the arm tracks from the locomotion clip:

```ts
const legsOnly = (clip: THREE.AnimationClip) => {
  const c = clip.clone();
  c.tracks = c.tracks.filter((t) => !/^arm-(left|right)\./.test(t.name));
  return c;
};
mixer.clipAction(legsOnly(walk)).play();
mixer.clipAction(holdingRight).play(); // the arms come from here only
```

### The blaster

`characters/weapon-shooter.glb`, about 0.1 wide, 0.23 tall and 0.3 long. It is gripped at its
origin, with the barrel along +Z.
Parent it to the character's `arm-right` joint:

```ts
const gun = blaster.clone();
gun.position.set(-0.24, 0, 0.03); // in the fist
gun.rotation.y = -Math.PI / 3;    // barrel along the character's facing
model.getObjectByName("arm-right")!.add(gun);
```

The offset assumes `holding-right` (or `holding-right-shoot`) is playing. With that pose, the
muzzle is at **(-0.19, 0.38, 0.43)** in character space, before the character's rotation. That
is right of center (the gun hand is on the character's right, which is -X), at chest height, and
in front of the body. Start tracers and muzzle flashes there.

## Scenery

`scenery/<name>.glb`. Footprints are X × Z at rotation 0, centered on the origin unless noted. A
piece rotated 90° about Y swaps X and Z.

| Model | Footprint | Height | Notes |
|---|---|---|---|
| `floor-square` | 1 × 1 | 0 | floor tile, checkered |
| `shape-cube` | 1 × 1 | 1 | solid block: outer walls, big cover |
| `shape-cube-half` | 1 × 1 | 0.5 | low block |
| `shape-cylinder` | 1 × 1 (round, r 0.5) | 1 | pillar |
| `shape-hexagon` | 0.87 × 1 | 1 | hex pillar |
| `crate`, `crate-color` | 0.5 × 0.5 | 0.5 | small cover (grey or orange) |
| `wall` | 0.2 × 1 | 1 | thin wall, runs along Z |
| `wall-low` | 0.2 × 1 | 0.5 | low thin wall, runs along Z |
| `column` | 0.2 × 0.2 | 1 | thin post |
| `column-low` | 0.2 × 0.2 | 0.5 | short post |
| `indicator-round-a` | 1 × 1 | 0.2 | animated ring (clip `animation`): spawn pad |
| `indicator-round-b` | 1 × 1 | flat | animated ring (clip `animation`) |
| `indicator-special-area`, `indicator-special-cross` | 1 × 1 | flat | floor markings |
| `flag` | 0.1 × 0.5 | 0.95 | decoration; pole at the origin, cloth toward +Z |
| `target-a-round` | 0.12 × 0.5 | 0.5 | decoration |

Markers and decorations don't block anything. For everything else, the footprint above is
exactly the box to collide against. If gameplay uses those same boxes, what players see blocking
them is what blocks them. Next to a 0.75-tall character, a 1-high block hides it completely and
a 0.5-high one reaches about chest height.
