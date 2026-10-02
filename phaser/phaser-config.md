# Phaser: configuration

## GameConfig

- [Types.Core | Phaser Help](https://docs.phaser.io/api-documentation/typedef/types-core#gameconfig)

- `type?: number`
	 - `Phaser.AUTO, Phaser.CANVAS, Phaser.HEADLESS, or Phaser.WEBGL`.
	 - Which renderer to use.
	 - `AUTO` picks `WEBGL` if available, otherwise `CANVAS`.

- `stableSort?: number | boolean`
	- built-in for older browsers

- `scale?: Phaser.Types.Core.ScaleConfig`

## ScaleConfig

- [Types.Core | Phaser Help](https://docs.phaser.io/api-documentation/typedef/types-core#scaleconfig)

- `mode?: enum ScaleModes: NONE | WIDTH_CONTROLS_HEIGHT | HEIGHT_CONTROLS_WIDTH | FIT | ENVELOP | RESIZE | EXPAND`
