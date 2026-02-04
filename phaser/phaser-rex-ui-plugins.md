# Phaser: Rex UI plugins

- https://github.com/rexrainbow/phaser3-rex-notes

## Textarea?

- :open_file_folder: `examples/ui-simplelabel/textarea.js`
- :balloon: [rexUI: Text-area](https://codepen.io/rexrainbow/pen/JzBZzy)
- :balloon: [rexUI: Textbox](https://codepen.io/rexrainbow/pen/MzGoJv)
	- :symbols::mag_right: `rexUI.add.textArea`
		- :open_file_folder: `docs/docs/ui-textareainput.md`
	- :open_file_folder: `templates/ui/textareainput/TextAreaInput.js`
		- `extends Scrollable`
			- :open_file_folder: `templates/ui/utils/scrollable/Scrollable.js`
				- `extends Sizer`
					- :open_file_folder: `templates/ui/sizer/Sizer.js`
						- `extends BaseSizer`
							- :open_file_folder: `templates/ui/basesizer/BaseSizer.js`
								- `extends Container`
									- :open_file_folder: `plugins/gameobjects/container/containerlite/ContainerLite.js`
										- `extends Base`
											- :open_file_folder: `plugins/gameobjects/container/containerlite/Base.js`
		- `inputText = new CanvasInput(scene, inputTextConfig)`
			- :open_file_folder: `plugins/gameobjects/dynamictext/canvasinput/CanvasInput.js`
				- `extends DynamicText`
					- :open_file_folder: `plugins/gameobjects/dynamictext/dynamictext/DynamicText.js`
						- `extends Canvas`
							- :open_file_folder: `plugins/gameobjects/canvas/canvasbase/Canvas.js`


## Checkbox implementation

- :open_file_folder: `plugins/gameobjects/shape/checkbox/Checkbox.js`

## Sizing

rex: `Sizer / FixedWidthSizer / GridSizer / OverlapSizer` are the cornerstone of `rexui`. The concept of rexui is setting position and size of children game objects, according to configuration. Others are applications of these 4 kinds of sizer The naming is based on wxwidget. Godot, unity have similar ui framework