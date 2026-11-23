# One-prompt assets

Ready-made art for the one-prompt games on [colyseus.io/ai](https://colyseus.io/ai). The prompt
hands the coding agent a fixed set of models, so it spends its time on the game and the netcode,
not on modelling.

Each kit is downloaded as a zip from this repo's [releases](../../releases), and has an
`ASSETS.md` for the agent: units, orientation, animation clips, where the weapon attaches and the
collision footprint of every piece.

| Kit | Download | Contents |
|---|---|---|
| [`shooter/`](shooter) | [`shooter-assets.zip`](../../releases/download/shooter-v2/shooter-assets.zip) | 8 animated characters and a blaster, 17 arena pieces, with tested code for three.js and `@pixi/3d` |

A release's zip never changes once a prompt links to it. Changes ship as a new release, so
[`shooter-v1`](../../releases/tag/shooter-v1), with notes for three.js only, stays as the
three.js run downloaded it.

## Credits

All models are by [Kenney](https://www.kenney.nl), CC0 (public domain), from
[Mini Characters](https://kenney.nl/assets/mini-characters) and
[Prototype Kit](https://kenney.nl/assets/prototype-kit). Each folder keeps Kenney's
`License.txt`. If you like them, consider [supporting Kenney](https://kenney.nl/donate).
