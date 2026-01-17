# Vue

## Generate stack trace (Vue 2.6)

- :sleuth_or_spy: `found in` in error trace inside Vue component in browser

- `generateComponentTrace()`
	- Not in `exports.`?

## vue-server-renderer (Vue 2.6)

- Где генерация :open_file_folder: `packages/vue-server-renderer/build.dev.js`?
	- :open_file_folder: `packages/vue-server-renderer/README.md`
		- :open_file_folder: `src/platforms/web/entry-server-renderer.js`
			- :open_file_folder: `src/server/bundle-renderer/create-bundle-renderer.js`
		- :open_file_folder: `packages/vue-server-renderer/server-plugin.js`

- :open_file_folder: `packages/vue-server-renderer/index.js`
	- Уже включает в себя :open_file_folder: `require('./build.dev.js')`
