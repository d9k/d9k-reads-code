# React: external store connect

## `useSyncExternalStore()`, `v19.0.0`

Used in [connect | react-redux](../react-redux/connect.md).

## Docs

 - :beginner: [useSyncExternalStore() | React docs](https://react.dev/reference/react/useSyncExternalStore)
``
```
useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

- `subscribe`: A function that takes a single callback argument and subscribes it to the store. When the store changes, it should invoke the provided callback, which will cause React to re-call getSnapshot and (if needed) re-render the component. The subscribe function should return a function that cleans up the subscription.

- `getSnapshot`: A function that returns a snapshot of the data in the store that’s needed by the component. While the store has not changed, repeated calls to getSnapshot must return the same value. If the store changes and the returned value is different (as compared by `Object.is`), React re-renders the component.

- :open_file_folder: `CHANGELOG.md`:

> ## 18.0.0 (March 29, 2022)
>
> `useSyncExternalStore` is a new hook that allows external stores to support ***concurrent*** reads by forcing updates to the store to be synchronous. It removes the need for `useEffect` when implementing subscriptions to external data sources, and is recommended for any library that integrates with state external to React.

## Code

- :open_file_folder: `packages/react/src/ReactHooks.js`

```ts
export function useSyncExternalStore<T>(
  subscribe: (() => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T,
): T {
  const dispatcher = resolveDispatcher();
  return dispatcher.useSyncExternalStore(
    subscribe,
    getSnapshot,
    getServerSnapshot,
  );
}
```

- :open_file_folder: `packages/react-reconciler/src/ReactFiberHooks.js`

```ts
    useSyncExternalStore<T>(
      subscribe: (() => void) => () => void,
      getSnapshot: () => T,
      getServerSnapshot?: () => T,
    ): T {
      currentHookNameInDev = 'useSyncExternalStore';
      mountHookTypesDev();
      return mountSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
    },
```

```ts
    useSyncExternalStore<T>(
      subscribe: (() => void) => () => void,
      getSnapshot: () => T,
      getServerSnapshot?: () => T,
    ): T {
      currentHookNameInDev = 'useSyncExternalStore';
      updateHookTypesDev();
      return updateSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
    },
```

```ts
function mountSyncExternalStore<T>(
  subscribe: (() => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T,
): T {
  const fiber = currentlyRenderingFiber;
  const hook = mountWorkInProgressHook();

  let nextSnapshot;
  const isHydrating = getIsHydrating();

  if (isHydrating) {
    /* . . . . . */
    nextSnapshot = getServerSnapshot();
    /* . . . . . */
  } else {
    nextSnapshot = getSnapshot();
    /* . . . . . */
  }

  /* . . . . . */

    const rootRenderLanes = getWorkInProgressRootRenderLanes();
    if (!includesBlockingLane(rootRenderLanes)) {
      pushStoreConsistencyCheck(fiber, getSnapshot, nextSnapshot);
    }
```