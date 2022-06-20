# Redux-toolkit: query: createApi()

## docs (rtk query 1.6.0)

## sources (rtk query 1.6.0)

https://redux-toolkit.js.org/rtk-query/overview

Doc: the data fetching and caching logic is built on top of Redux Toolkit's `createSlice` and `createAsyncThunk` APIs

### createApi() at `src/query/react/index.ts`

`function buildCreateApi(modules: Modules): CreateApi`

```typescript
const createApi = /* @__PURE__ */ buildCreateApi(
  coreModule(),
  reactHooksModule()
)
```

- args
  - See [createApi() args type](./query-create-api-args.md)
  - See doc on args: https://redux-toolkit.js.org/rtk-query/api/createApi

- see [buildCreateApi()](#build-create-api), [reactHooksModule()](#react-hooks-module) definition below

### coreModule() in `src/query/core/module.ts`

- args: `name`, `init()`

- `buildSlice()` from `src/query/core/buildSlice.ts`
  - arg `queryThunk: QueryThunk`
  - `createSlice()`
    - 1 `querySlice`
    - 2 `mutationSlice`
    - 3 `invalidationSlice`
    - 4 `subscriptionSlice`
    - 5 `configSlice`
  - return `{ reducer, actions }`
    - `actions: SliceActions`

- query slice:
  - actions: `queryResultPatched()`, `removeQueryResult()`
  - calls: `applyPatches()`, `copyWithStructuralSharing()`, `updateQuerySubstateIfExists()`,
  - listens `queryThunk`: `.pending`, `.fulfilled`, `.rejected`


- mutation slice:
  - actions: `unsubscribeMutationResult()`:
  - calls: `updateMutationSubstateIfExists()`

- invalidation slice:
  - calls:
    - `calculateProvidedByThunk()`
  - listens
    - `queryThunk`:
      - `isAnyOf(isFulfilled(queryThunk), isRejectedWithValue(queryThunk))`
    - other slices actions:
      - query slice: `removeQueryResult()`

- subscription slice:
  - actions: `unsubscribeQueryResult()`, `updateSubscriptionOptions()`
  - listens slices actions calls:
    - query slice: `removeQueryResult()`
    - `queryThunk`: `.pending`, `.rejected`

- config slice:
  - actions: `middlewareRegistered()`
  - state: `online`, `focused`, `middlewareRegistered`
  - listens: `onOnline`, `onOffline`, `onFocus`, `onFocusLost`
    - см. `src/query/core/setupListeners.ts`

```typescript
> import { keys } from 'ts-transformer-keys'
> import { SliceActions } from 'src/query/core/buildSlice'
> keys<SliceActions>().sort()
[
  'middlewareRegistered',
  'queryResultPatched',
  'removeQueryResult',
  'resetApiState',
  'unsubscribeMutationResult',
  'unsubscribeQueryResult',
  'updateSubscriptionOptions'
]

> keys<SliceActions['resetApiState']>()
[ 'type', 'match' ]
```

<details>
<summary>TODO reducer type detection fails!</summary>

```typescript
> import { schema } from 'ts-transformer-json-schema';
> import { buildSlice } from 'src/query/core/buildSlice'
> schema<ReturnType<typeof buildSlice>['reducer']>()
{}
```

</details>

`actions` of `buildSlice()` result consist of `*Slice` created above.

```typescript
const actions = {
  ...configSlice.actions,         // 5
  ...querySlice.actions,          // 1
  ...subscriptionSlice.actions,   // 4
  ...mutationSlice.actions,       // 2
  resetApiState,
}
```

### <a name="react-hooks-module">reactHooksModule()</a> in `src/query/react/module.ts`

- uses `batch(fn: Function)` - ensure that multiple actions dispatched outside of React only result in a single render update,
- returns `{ injectEndpoint(endpointName, definition) {`

      ```typescript
      const { buildQueryHooks, buildMutationHook, usePrefetch } = buildHooks({
        api,
        moduleOptions: { batch, useDispatch, useSelector, useStore },
      })
      ```

    ```typescript
    injectEndpoint(
      endpointName: string,
      definition: EndpointDefinition<any, any, any, any>
    ): void
    ```

  - fn calls: `buildQueryHooks()`

    ```typescript
    const {
      useQuery,
      useLazyQuery,
      useLazyQuerySubscription,
      useQueryState,
      useQuerySubscription,
    } = buildQueryHooks(endpointName)
    ```

See [EndpointDefinition](#endpoint-definition) below.

### <a name="build-create-api">buildCreateApi()</a> in `src/query/createApi.ts`

- args: only varargs `(...modules)`

- inner declared `baseCreateApi()`
  - `serializeQueryArgs: defaultSerializeQueryArgs`:

    ```typescript
    > defaultSerializeQueryArgs({'endpointName': 'endpointName', queryArgs: {arg1: 1, arg2: 'test'}, endpointDefinition: null as any})
    'endpointName({"arg1":1,"arg2":"test"})'
    ```

  - `optionsWithDefaults`:

    ```typescript
    {
      reducerPath: 'api',
      serializeQueryArgs: defaultSerializeQueryArgs,
      keepUnusedDataFor: 60,
      refetchOnMountOrArgChange: false,
      refetchOnFocus: false,
      refetchOnReconnect: false,
      ...options,
      tagTypes: [...(options.tagTypes || [])],
    }
    ```
  - `context`

    ```typescript
    const context: ApiContext<EndpointDefinitions> = {
      endpointDefinitions: {},
      batch(fn) // . . . . .
    ```

    ```typescript
    const initializedModules = modules.map((m) =>
      m.init(api as any, optionsWithDefaults, context)
    )
    ```

  - returns `api = { injectEndpoints, enhanceEndpoints }` (functions)


- `api = { injectEndpoints, enhanceEndpoints }`
  - `context.endpointDefinitions[endpointName] = definition`

### <a name="endpoint-definition">EndpointDefinition</a>

```typescript
export type EndpointDefinition<
  QueryArg,
  BaseQuery extends BaseQueryFn,
  TagTypes extends string,
  ResultType,
  ReducerPath extends string = string
> =
  | QueryDefinition<QueryArg, BaseQuery, TagTypes, ResultType, ReducerPath>
  | MutationDefinition<QueryArg, BaseQuery, TagTypes, ResultType, ReducerPath>
```

### Additional query result processing

```typescript
extraReducers: (builder) => {
    builder
      .addMatcher(
        api.endpoints.yourEndpoint.matchFulfilled,
        (state, { payload: { result } }) => {
              // do things with the result
        }
      );
  },
```

## Endpoints definitions

`EndpointDefinitionWithQuery`


