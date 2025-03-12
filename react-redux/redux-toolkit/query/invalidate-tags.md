# RTK Query: invalidateTags

## source RTK 1.6.0

### invalidateTags()

#### src/query/core/module.ts

```
  util: {
      invalidateTags: ActionCreatorWithPayload<
        Array<TagTypes | FullTagDescription<TagTypes>>,
      string
    >
  }
```

#### src/query/core/buildMiddleware/invalidationByTags.ts

```
  function invalidateTags(
    tags: readonly FullTagDescription<string>[],
    api: SubMiddlewareApi
  ) {
```

multiple `invalidateTags()`, example:

```
  if (api.util.invalidateTags.match(action)) {
    invalidateTags(
      calculateProvidedBy(
        action.payload,
        undefined,
        undefined,
        undefined,
        assertTagType
      ),
      mwApi
    )
  }
```

```
  if (
    isAnyOf(
      isFulfilled(mutationThunk),
      isRejectedWithValue(mutationThunk)
    )(action)
  ) {
    invalidateTags(
      calculateProvidedByThunk(
        action,
        'invalidatesTags',
        endpointDefinitions,
        assertTagType
      ),
      mwApi
    )
  }
```

### calculateProvidedByThunk()

`src/query/core/buildThunks.ts`

```
export function calculateProvidedByThunk(
  action: UnwrapPromise<
    ReturnType<ReturnType<QueryThunk>> | ReturnType<ReturnType<MutationThunk>>
  >,
  type: 'providesTags' | 'invalidatesTags',
  endpointDefinitions: EndpointDefinitions,
  assertTagType: AssertTagTypes
) {
  return calculateProvidedBy(
    endpointDefinitions[action.meta.arg.endpointName][type],
    isFulfilled(action) ? action.payload : undefined,
    isRejectedWithValue(action) ? action.payload : undefined,
    action.meta.arg.originalArgs,
    assertTagType
  )
}
```

### calculateProvidedBy()

#### src/query/endpointDefinitions.ts

```
export function calculateProvidedBy<ResultType, QueryArg, ErrorType>(
  description:
    | ResultDescription<string, ResultType, QueryArg, ErrorType>
    | undefined,
  result: ResultType | undefined,
  error: ErrorType | undefined,
  queryArg: QueryArg,
  assertTagTypes: AssertTagTypes
): readonly FullTagDescription<string>[] {
  if (isFunction(description)) {
    return description(result as ResultType, error as undefined, queryArg)
      .map(expandTagDescription)
      .map(assertTagTypes)
  }
  if (Array.isArray(description)) {
    return description.map(expandTagDescription).map(assertTagTypes)
  }
  return []
}
```
