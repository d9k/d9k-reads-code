# Vue Flow

- https://github.com/bcakmakoglu/vue-flow
- `@core`: `packages/core/src`
- version: `@vue-flow/core@1.33.5`

## Demo Edges: where named templates are defined?

- [ ] [Edges | Vue Flow Examples](https://vueflow.dev/examples/edges/)
- Like `<template #edge-button="buttonEdgeProps">`

- `{  id: 'e5-8', type: 'button' }`
- 💎 _`<VueFlow>`_,  :open_file_folder:`@core/container/VueFlow`
	- `provide(Slots, slots)`
	- `<EdgeRenderer />`
- :gem: _`<EdgeRenderer>`_ , :open_file_folder: `@core/container/EdgeRenderer`
	- `<EdgeWrapper :type="getType(edge.type, edge.template)"`
		- :arrow_right_hook: `fn getType(type, template?)`
			- `name = type || 'default'`
			- ``slots?.[`edge-${name}`]``
- `fn getEdgeTypes()`
	- `defaultEdgeTypes` from :open_file_folder: `@core/utils/defaultNodesEdges.ts`
		- `default | straight | step | smoothstep | simplebezier`
- `InjectionKey`
	- [Dependency Injection with Provide/Inject in Vue.js 3 | Development | Borstch](https://borstch.com/blog/development/dependency-injection-with-provideinject-in-vuejs-3)