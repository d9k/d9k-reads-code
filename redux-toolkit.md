# Redux-toolkit

## [] Исходники rtk query 1.6.0

https://redux-toolkit.js.org/rtk-query/overview

Doc: the data fetching and caching logic is built on top of Redux Toolkit's `createSlice` and `createAsyncThunk` APIs

- `coreModule()`
  - `buildSlice()`
    - аргумент `queryThunk: QueryThunk`
    - `createSlice()`
      - `querySlice`
      - `mutationSlice`
      - `invalidationSlice`
      - `subscriptionSlice`
      - `configSlice`

- `SliceActions`
