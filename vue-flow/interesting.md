# Vue Flow: interesting findings

## #TS: event handlers names from events names

- :page_with_curl:`hooks.ts` in :open_file_folder:`packages/core/src/types`:

```ts
export type FlowHooksOn = Readonly<{
  [key in keyof FlowEvents as `on${Capitalize<key>}`]: EventHookOn<FlowEvents[key]>
}>
```

- :page_with_curl:`store.ts` in :open_file_folder:`packages/core/src/types`:

```ts
export type VueFlowStore = {
  readonly id: string
  readonly emits: FlowHooksEmit
  readonly vueFlowVersion: string
} & FlowHooksOn &
  ToRefs<State> &
  Readonly<ComputedGetters> &
  Readonly<Actions>
```