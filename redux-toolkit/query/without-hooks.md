# Redux-toolkit: query: without hooks

## rtk query 1.7.2 docs

https://redux-toolkit.js.org/rtk-query/usage/usage-without-react-hooks

```
// Adding a cache subscription
const result = dispatch(api.endpoints.getPosts.initiate())

// Removing the corresponding cache subscription
result.unsubscribe()

const result = api.endpoints.getPosts.select()(state)
const { data, status, error } = result
```

## rtk query 1.7.2 sources

```
const dispatchResult = dispatch(mutationAction.initiate(queryArgs));
const result = await dispatchResult;
dispatchResult.unsubscribe();
return result;
```

`unsubscribe()` calls `*Api/mutations/removeMutationResult"` redux action with payload `{requestId: <XXXXX}`.

### initiate()

in `init()` in `packages/toolkit/src/query/core/module.ts`:

```
initiate: buildInitiateQuery(endpointName, definition),
initiate: buildInitiateMutation(endpointName)
```

```
const {
  buildInitiateQuery,
  buildInitiateMutation,
} = buildInitiate({
```

### buildInitiate() in packages/toolkit/src/query/core/buildInitiate.ts

- `buildInitiateQuery()`
  - `queryThunk()`
- `buildInitiateMutation()`
  - `mutationThunk()`

### buildThunks()

in `packages/toolkit/src/query/core/buildThunks.ts`

### requestId?

`idGenerator`

`packages/toolkit/src/createAsyncThunk.ts`

in `createAsyncThunk()`

```
function actionCreator(
  arg: ThunkArg
): AsyncThunkAction<Returned, ThunkArg, ThunkApiConfig> {
  return (dispatch, getState, extra) => {
    const requestId = options?.idGenerator
      ? options.idGenerator(arg)
      : nanoid()
```

`nanoid()` in `packages/toolkit/src/nanoid.ts`

### selectors

`packages/toolkit/src/query/core/module.ts`

```
select: buildQuerySelector(endpointName, definition),
select: buildMutationSelector()
```

```
const { buildQuerySelector, buildMutationSelector, selectInvalidatedBy } =
  buildSelectors({
    serializeQueryArgs: serializeQueryArgs as any,
    reducerPath,
  })
```

### buildSelectors

`packages/toolkit/src/query/core/buildSelectors.ts`

- buildQuerySelector
  - selectQuerySubState
  - createSelector

