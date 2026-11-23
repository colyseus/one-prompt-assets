# Shooter assets

Low-poly GLB models for a top-down 3D shooter. Picked from two Kenney packs, both CC0 (public
domain): credit "Kenney (www.kenney.nl)" is appreciated, not required.

- `characters/`: [Mini Characters 2.0](https://kenney.nl/assets/mini-characters), 8 characters and a blaster
- `scenery/`: [Prototype Kit 1.0](https://kenney.nl/assets/prototype-kit), floor, walls, cover and markers

Every value below was measured from these files. The facts hold for any engine; the last two
sections give tested code for [three.js](#in-threejs) and [`@pixi/3d`](#in-pixi3d).

## Conventions

- **Units:** 1 unit = 1 grid tile. Y is up. Every origin is at the bottom center of the model.
- **Facing:** characters face **+Z**. To face a direction `(dx, dz)` on the ground plane, set
  `model.rotation.y = Math.atan2(dx, dz)`.
- **Textures:** each folder has its own `Textures/colormap.png`, loaded by a relative path from
  inside the GLB. The two files differ, so keep both folders as they are and don't merge them.
  Colors come from that one atlas: tint with a material color or an emissive flash, not new textures.

## Characters

`characters/character-<name>.glb`: `soldier`, `raider`, `robot`, `alien`, `knight`, `worker`,
`zombie`, `male-c` (a police officer). One per player seat.

- 0.67 tall (zombie) to 0.84 (robot, with its antenna), most about 0.75. A circle of radius 0.3
  covers the body.
- All 8 share one rig, with no hand bones. Its glTF nodes are the same in every file:

  | Node | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
  |---|---|---|---|---|---|---|---|---|
  | Name | `character-<name>` | `root` | `leg-left` | `leg-right` | `torso` | `arm-left` | `arm-right` | `head` |

- There is no horizontal root motion: `walk`, `sprint` and `die` only bob `root` up and down, so
  place the model's root at the game position.

Clips for a shooter (each file has 32 clips):

| Clip | Length | Animates | Use |
|---|---|---|---|
| `idle` | 1.33 s loop | torso, arms, head | standing |
| `walk` | 0.67 s loop | whole body | moving |
| `sprint` | 0.50 s loop | whole body | moving fast |
| `holding-right` | 0.17 s, a pose | **arms only** | blaster held out, layered over the above |
| `holding-right-shoot` | 0.20 s | torso, arms, head | recoil: play once per shot; starts and ends on the `holding-right` pose |
| `die` | 0.33 s | whole body | play once and hold the last frame |
| `emote-yes` | 0.67 s | upper body | victory |

**Layering.** `idle`, `walk` and `sprint` animate the arms too. Played together with
`holding-right`, they blend with it and the gun arm swings (by up to 53° in `walk`). Take the arm
tracks (`arm-left` and `arm-right`) out of the locomotion clip, so the arms come from
`holding-right` alone.

### The blaster

`characters/weapon-shooter.glb`, about 0.1 wide, 0.23 tall and 0.3 long. It is gripped at its
origin, with the barrel along +Z, and the muzzle at (0, 0.088, 0.224) in its own space.

In the fist, the gun sits at **(-0.24, 0, 0.03)** in the `arm-right` joint's space, turned
**-π/3 about Y**. That assumes `holding-right` (or `holding-right-shoot`) is playing. With that
pose, the muzzle is at about **(-0.19, 0.38, 0.43)** in character space, before the character's
rotation. That is right of center (the gun hand is on the character's right, which is -X), at
chest height, and in front of the body. Start tracers and muzzle flashes there.

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

## In three.js

Load with `GLTFLoader`.

- **One character per player: clone with `SkeletonUtils.clone()`**
  (`three/addons/utils/SkeletonUtils.js`). A plain `.clone()` of a skinned model shares the
  skeleton, so every copy plays the same pose.
- **Layering.** Track names are `<joint>.<property>`:

  ```ts
  const legsOnly = (clip: THREE.AnimationClip) => {
    const c = clip.clone();
    c.tracks = c.tracks.filter((t) => !/^arm-(left|right)\./.test(t.name));
    return c;
  };
  mixer.clipAction(legsOnly(walk)).play();
  mixer.clipAction(holdingRight).play(); // the arms come from here only
  ```

- **The blaster.** Joints are scene nodes, so parent the gun to `arm-right`:

  ```ts
  const gun = blaster.clone();
  gun.position.set(-0.24, 0, 0.03); // in the fist
  gun.rotation.y = -Math.PI / 3;    // barrel along the character's facing
  model.getObjectByName("arm-right")!.add(gun);
  ```

- **Shadows.** Flat faces show shadow acne with the defaults. Set
  `light.shadow.bias = -0.0005` and `light.shadow.normalBias = 0.02`.

## In @pixi/3d

Load with `Assets.load<Scene3DSource>(url)`. Tested with `@pixi/3d` 0.1.0 and `pixi.js` 8.23.

- **One character per player: `new Model3D({ source })`.** Each model gets its own animation
  player, so no cloning is needed.
- **Layering.** Track labels are `<glTF node>.<property>`, so the arms are the tracks of nodes 5
  and 6:

  ```ts
  import { Animation } from "@pixi/3d";

  const legsOnly = (clip: Animation) => new Animation({
    label: `${clip.label}-legs`,
    duration: clip.duration,
    tracks: clip.tracks.filter((t) => !/^[56]\./.test(t.label)), // arm-left, arm-right
  });
  model.player!.play(legsOnly(model.player!.getAnimation("walk")!));
  model.player!.play("holding-right"); // the arms come from here only
  ```

  Clips playing together blend by weight, property by property. While `holding-right-shoot`
  plays (`loop = false`, then `gotoAndPlay(0)` for each shot), give it a higher `weight` than
  `holding-right`, so the recoil wins over the held pose.
- **The blaster.** Joints aren't scene nodes, so the gun can't be parented to `arm-right`. Add it
  to the character model and set its transform from the joint every frame, after the animation
  has posed the skeleton. Build the character with `autoUpdate: false` and update its player
  yourself, so the pose is current when you read it:

  ```ts
  import { Matrix4, Model3D, Quaternion, Vector3, type Skeleton } from "@pixi/3d";

  /** A joint's posed transform in its model's space. */
  function jointMatrix(sk: Skeleton, node: number, out: Matrix4): Matrix4 {
    const q = new Quaternion();
    const m = new Matrix4();
    out.identity();
    for (let i = node; i >= 0; i = sk.parent[i]) {
      const p = i * 3;
      const r = i * 4;
      q.set(sk.localRotation[r], sk.localRotation[r + 1], sk.localRotation[r + 2], sk.localRotation[r + 3]);
      m.compose(
        { x: sk.localPosition[p], y: sk.localPosition[p + 1], z: sk.localPosition[p + 2] }, q,
        { x: sk.localScale[p], y: sk.localScale[p + 1], z: sk.localScale[p + 2] },
      );
      out.premultiply(m);
    }
    return out;
  }

  const GRIP = new Matrix4().compose(
    { x: -0.24, y: 0, z: 0.03 },                                                // in the fist
    new Quaternion().setFromAxisAngle({ x: 0, y: 1, z: 0 }, -Math.PI / 3),       // barrel forward
    { x: 1, y: 1, z: 1 },
  );

  const model = new Model3D({ source: character, autoUpdate: false, castShadow: true });
  const gun = new Model3D({ source: blaster, castShadow: true });
  model.addChild(gun);
  const armRight = model.skeletons[0].indexOf("arm-right");
  const pose = new Matrix4();
  const position = new Vector3();
  const rotation = new Quaternion();
  const scale = new Vector3();

  app.ticker.add((ticker) => {
    model.player!.update(ticker);
    jointMatrix(model.skeletons[0], armRight, pose).multiply(GRIP);
    pose.decompose(position, rotation, scale);
    gun.position.copyFrom(position);
    gun.quaternion.copyFrom(rotation);
    gun.scale.copyFrom(scale);
  });
  ```

- **Shadows.** The defaults show no acne on these models: `castShadow` on the light and the
  models, `receiveShadow` on the floor, and a `shadowQuality` on the view.
