# Redux-toolkit: query: endpoints

## rtk query 1.6.0 docs

### Endpoint definition

```
const api = createApi({
  // ...
  endpoints: (build) => ({
    getPosts: build.query<PostsResponse, void>({
      query: () => 'posts',
    }),
    flipCoin: build.query<'heads' | 'tails', void>({
      queryFn(arg, queryApi, extraOptions, baseQuery) {
        const randomVal = Math.random()
        if (randomVal < 0.45) {
          return { data: 'heads' }
        }
        if (randomVal < 0.9) {
          return { data: 'tails' }
        }
        return {
          error: {
            status: 500,
            statusText: 'Internal Server Error',
            data: "Coin landed on it's edge!",
          },
        }
      },
```

* arguments for `build.query<>`:
  - `query` | `queryFn`
  - `extraOptions`
  - `keepUnusedDataFor`
  - `onCacheEntryAdded`
  - `onQueryStarted`
  - `providesTags` (`query`) | `invalidatesTags` (`mutation`)
  - `transformResponse`

https://redux-toolkit.js.org/rtk-query/api/createApi#query-endpoint-definition

https://redux-toolkit.js.org/rtk-query/api/createApi#anatomy-of-an-endpoint

`extraOptions?: BaseQueryExtraOptions<BaseQuery>`

`QueryReturnValue<ResultType, BaseQueryError<BaseQuery>>`

## rtk query 1.6.0 sources

### EndpointBuilder at `src/query/endpointDefinitions.ts`

- `build.query<ResultType, QueryArg>(~QueryDefinition)`
- `build.mutation<ResultType, QueryArg>(~MutationDefinition)`

### Endpoint definitions at `src/query/endpointDefinitions.ts`

- `DefinitionType`: `'query'` | `'mutation'`

- `BaseEndpointDefinition<Arg, QueryFn, ResultType>`,
  - `resultType`, `baseQuery`

- `QueryDefinition`: `BaseEndpointDefinition` & `QueryExtraOptions`
  - `QueryExtraOptions`:  `providesTags`

- `MutationDefinition`:
  - `MutationExtraOptions`: `invalidatesTags`

- `EndpointDefinition`: `QueryDefinition` | `MutationDefinition `

- "phantom type"?
  - type notes
    - https://dev.to/busypeoples/notes-on-typescript-phantom-types-kg9
