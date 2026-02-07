# Love2D: Gyrotest by Snusmumriken

2026.01, ask https://t.me/Snusmumriken for code

## Rendering

- :open_file_folder: `game.lua`
	- `game:draw()`
		- `self.camera:attach()`
			- `physo:WorldDraw()`
		- `self.camera:detach()`

- :open_file_folder: `physo.lua`
	- for `k, v` in `pairs(self.World_BodiesToUpdate)` do
		- `if `v.draw`
			- `v:draw()`

- :open_file_folder: `map/types.lua`
	- `Cube:draw()`
	- `Brick:draw()`

## Updating

- :open_file_folder: `game.lua`
	- `game:update(dt)`
		- :open_file_folder: `self.map:update(dt)`
		- :open_file_folder: `self.physo:WorldUpdate(dt)`
		- :open_file_folder: `self.camera:lookAt(self.ball:getPosition())`
		- :open_file_folder: `self.effects:updateAll(dt)`

- :open_file_folder: `tiled.lua`
	- `map:update(dt)` // used for update image from hard drive, not for animation

- :open_file_folder: `physo.lua`
	- `physo:WorldUpdate(dt)`
		- for `k, v` in `pairs(self.World_BodiesToUpdate)`
			- `if v.update then v:update(dt) end`

## Creation and declaration of world bodies

- :open_file_folder: `physo.lua`
	- `physo:createBody(x, y, bodytype, shape)`
		- `self.body = love.physics.newBody(self.World, x, y, bodytype)`
		- `fix = love.physics.newFixture(self.body, shape)`

- :open_file_folder: `map/types.lua`
	- `Brick:new(x, y, tile)`
		- `self:createBody(x, y)`
		- `self.tile = tile`
		- `return { [0]  = Wall, [4]  = Laser, [11] = Cube, [12] = Brick, … }`
			- :link: `game.load()`

- :open_file_folder: `game.lua`
	- `game:load(path)`
		- `self.path = path or 'map/map1.lua'`
		- `self.map = require'tiled':new(self.path)`
		- `Tileclasses = require'map.types'`
		- fn `mapper(instance, tile, layer, x, y)`
			- `class = Tileclasses[instance.idV`
			- `instance.obj = class(x, y, instance)`
		- `map:mapTiles(mapper, _, switchSwitches)`

- :open_file_folder: `tiled.lua`
	- `map:mapTiles(callback, preSolve, postSolve, ...)`
		- `layers = self:getLayers(… l.type == "tilelayer")`
		- for `layer` in `layers`
			- for `instance` in `layer.data`
				- `tile = instance:getTile()`
				- `postSolve(instance, tile, layer, x, y, ...)`

## Culling

TODO