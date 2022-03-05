# Redux-toolkit

## [ ] Исходники rtk query 1.6.0

https://redux-toolkit.js.org/rtk-query/overview

Doc: the data fetching and caching logic is built on top of Redux Toolkit's `createSlice` and `createAsyncThunk` APIs

### `createApi()`

`function buildCreateApi(modules: Modules): CreateApi`

```typescript
const createApi = /* @__PURE__ */ buildCreateApi(
  coreModule(),
  reactHooksModule()
)
```

- see [buildCreateApi()](#build-create-api) definition below

### `coreModule()`

- `buildSlice()`

  - arg `queryThunk: QueryThunk`
  - `createSlice()`
    - 1 `querySlice`
    - 2 `mutationSlice`
    - 3 `invalidationSlice`
    - 4 `subscriptionSlice`
    - 5 `configSlice`
  - return `{ reducer, actions }`
    - `actions: SliceActions`

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

### `reactHooksModule()`

- uses `batch(fn: Function)` - ensure that multiple actions dispatched outside of React only result in a single render update,
- returns `{ injectEndpoint(endpointName, definition) {`

```typescript
injectEndpoint(
  endpointName: string,
  definition: EndpointDefinition<any, any, any, any>
): void
```

See [EndpointDefinition](#endpoint-definition) below.

### <a name="build-create-api">buildCreateApi()</a>

- `baseCreateApi()`

- `optionsWithDefaults`

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

- `injectEndpoints()`
* `defaultSerializeQueryArgs`

```ts
> defaultSerializeQueryArgs({'endpointName': 'endpointName', queryArgs: {arg1: 1, arg2: 'test'}, endpointDefinition: null as any})
'endpointName({"arg1":1,"arg2":"test"})'
```

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
