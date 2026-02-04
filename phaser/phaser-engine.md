# Phaser: engine

- [phaserjs/phaser at v3.90.0](https://github.com/phaserjs/phaser/tree/v3.90.0)
- [ ] autocomplete problems
	- [x] exclude `dist/` from search and autocompletion
		- `"search.exclude": { "**/dist": true }`
	- [x] `npm i`

## How render works?

- :open_file_folder: `src/renderer/canvas/CanvasRenderer.js`
	- `render(scene, children, camera)`
		- for `var i = 0; i < childCount; i++`
			- `child.renderCanvas(this, child, camera);`

### renderCanvas() implementations

- :open_file_folder: `src/gameobjects/bitmaptext/static/BitmapTextCanvasRenderer.js`
- :open_file_folder: `src/gameobjects/shape/line/LineCanvasRenderer.js`
- :open_file_folder: `src/gameobjects/sprite/SpriteCanvasRenderer.js`
	- `camera.addToRenderList(src)`
	- `renderer.batchSprite(src, src.frame, camera, parentMatrix)`
		- :open_file_folder: `src/renderer/canvas/CanvasRenderer.js`
			- `alpha = camera.alpha * sprite.alpha`
			- if `sprite.isCropped`

### Culling

- :open_file_folder: `src/cameras/2d/BaseCamera.js`
	- `cull(renderableObjects)`
		- if `disableCull`