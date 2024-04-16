# Vue Flow

- https://github.com/bcakmakoglu/vue-flow
- Aliases for this file:
	- `$core`: `core="packages/core/src"`
- version: `@vue-flow/core@1.33.5`

## Demo Edges: where named templates are defined?

- [ ] [Edges | Vue Flow Examples](https://vueflow.dev/examples/edges/)
- Like `<template #edge-button="buttonEdgeProps">`

- `{  id: 'e5-8', type: 'button' }`
- 💎 _`<VueFlow>`_,  :open_file_folder:`$core/container/VueFlow`
	- `provide(Slots, slots)`
	- `<EdgeRenderer />`
- :gem: _`<EdgeRenderer>`_ , :open_file_folder: `$core/container/EdgeRenderer`
	- `<EdgeWrapper :type="getType(edge.type, edge.template)"`
		- :arrow_right_hook: `fn getType(type, template?)`
			- `name = type || 'default'`
			- ``slots?.[`edge-${name}`]``
- `fn getEdgeTypes()`
	- `defaultEdgeTypes` from :open_file_folder: `@core/utils/defaultNodesEdges.ts`
		- `default | straight | step | smoothstep | simplebezier`
- `InjectionKey`
	- [Dependency Injection with Provide/Inject in Vue.js 3 | Development | Borstch](https://borstch.com/blog/development/dependency-injection-with-provideinject-in-vuejs-3)

## Demo edges connection validation: why `isValidSourcePos` is deprecated?

[Connection Validation | Vue Flow](https://vueflow.dev/examples/edges/validation.html)

- :page_with_curl:`node.ts` at :open_file_folder:`$core/types`:

```ts
export interface NodeProps /* ... */{ /* ... */
  /**
   * @deprecated will be removed in next major release
   * called when used as target for new connection
   */
  isValidTargetPos?: ValidConnectionFunc /* ... */
```

- :page_with_curl:`handle.ts` at :open_file_folder: `$core/types`:

```ts
export type ValidConnectionFunc = (
  connection: Connection,
  elements: { edges: GraphEdge[]; nodes: GraphNode[]; sourceNode: GraphNode; targetNode: GraphNode },
) => boolean
```

`@deprecated` added at `chore(core): deprecate some node props` 2024.02.05 `ab482bc`. Commit additional lines:

```
- * @deprecated will be removed in next major release and replaced by just `isValidConnection` prop
+ * @deprecated will be removed in next major release
```

- :page_with_curl:`flow.ts` at :open_file_folder:`$core/types:
```ts
export interface FlowProps { /* ... */
	isValidConnection?: ValidConnectionFunc | null
```