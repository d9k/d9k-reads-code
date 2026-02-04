# Defold: engine

- [defold/defold at 1.9.8](https://github.com/defold/defold/tree/1.9.8)

## Rendering loop

- :beginner: [The render pipeline in Defold](https://defold.com/manuals/render/)

- :open_file_folder: `engine/docs/ENGINE_LIFECYCLE.md`
	- `// The engine update is calling [dmEngineUpdate()](../engine/src/engine.cpp), which is in turn calling [dmEngine::Step()](../engine/src/engine.cpp).`

- :open_file_folder: `engine/engine/src/engine_loop.cpp`
	- `result = params->m_EngineUpdate(engine)`

- :open_file_folder: `engine/engine/src/engine_main.cpp`
	- `static int EngineMain(int argc, char *argv[])`
		- `params.m_EngineUpdate = (dmEngine::EngineUpdate)dmEngineUpdate;`

- :open_file_folder: `engine/engine/src/engine.cpp`
	- `dmEngine::Step(engine)`
	- `void Step(HEngine engine)`
		- `StepFrame(engine, step_dt)`
	- `static void StepFrame(HEngine engine, float dt)`
		- if `engine->m_GOScriptContext`
			- `dmGameSystem::UpdateScriptLibs(script_lib_context)`
		- if `engine->m_RenderScriptContext`
			- `dmScript::Update(engine->m_RenderScriptContext)`
			- `dmExtension::Update(extension_params)`

- :open_file_folder: `engine/content/builtins/render/default.render_script`
	- `set_camera_world(state)`
		- if `camera.get_enabled(camera_components)`
			- `render.set_camera(camera_component.camera, { use_frustum = true })`
		- else
			- `camera_world = state.cameras.camera_world`
				- `// If no active camera was found, we use the default main "camera world" camera`

## Messages

- :open_file_folder: `script/src/script_msg.h`
	- `namespace dmScript`
		- `void InitializeMsg(lua_State* L);`

- :open_file_folder: `script/src/script_msg.cpp`
	- `static const luaL_reg ScriptMsg_methods[] =`
		- `{SCRIPT_TYPE_NAME_URL, URL_new},`
        - `{"post", Msg_Post}`
	- `luaL_register(L, SCRIPT_LIB_NAME, ScriptMsg_methods);`
	- `int Msg_Post(lua_State* L)`
		- `ResolveURL(L, 1, &receiver, &sender)`
		- if `lua_isstring(L, 2)`
			- `message_id = dmHashString64(lua_tostring(L, 2))`
		- `dmMessage::Result result = dmMessage::Post(&sender, &receiver, message_id, 0, (uintptr_t) desc, data, data_size, 0)`

- :open_file_folder: `dlib/src/dlib/message.cpp`