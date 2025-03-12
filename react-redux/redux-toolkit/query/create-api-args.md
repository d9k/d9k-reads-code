# Redux-toolkit: query: createApi() args

## docs (rtk query 1.6.0)

https://redux-toolkit.js.org/rtk-query/api/createApi

- `baseQuery`
- `endpoints`
- `extractRehydrationInfo`
- `keepUnusedDataFor`
- `reducerPath`
- `refetchOnMountOrArgChange`
- `refetchOnFocus`
- `refetchOnReconnect`
- `serializeQueryArgs`
- `tagTypes`

### baseQuery

`(args, api: {signal, dispatch, getState, extra}, extraOptions)`

### serializeQueryArgs​

creates a cache key based on the combination of arguments + endpoint name (ignoring object key order)




## Runtime reflection (rtk query 1.6.0)

```typescript
> import { schema } from 'ts-transformer-json-schema';
> import {buildCreateApi} from 'src/query/react/index.ts'
> schema<Parameters<ReturnType<typeof buildCreateApi>>[0]>()
```

See doc on args: https://redux-toolkit.js.org/rtk-query/api/createApi

```typescript
> schema<Parameters<ReturnType<typeof buildCreateApi>>[0]>()
{
  baseQuery: {
    example: '```ts\n' +
      "import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'\n" +
      '\n' +
      'const api = createApi({\n' +
      '  // highlight-start\n' +
      "  baseQuery: fetchBaseQuery({ baseUrl: '/' }),\n" +
      '  // highlight-end\n' +
      '  endpoints: (build) => ({\n' +
      '    // ...endpoints\n' +
      '  }),\n' +
      '})\n' +
      '```'
  },
  tagTypes: {
    example: '```ts\n' +
      "import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'\n" +
      '\n' +
      'const api = createApi({\n' +
      "  baseQuery: fetchBaseQuery({ baseUrl: '/' }),\n" +
      '  // highlight-start\n' +
      "  tagTypes: ['Post', 'User'],\n" +
      '  // highlight-end\n' +
      '  endpoints: (build) => ({\n' +
      '    // ...endpoints\n' +
      '  }),\n' +
      '})\n' +
      '```',
    optional: true,
    type: 'array',
    items: { type: 'string' }
  },
  reducerPath: {
    example: '```ts\n' +
      '// codeblock-meta title="apis.js"\n' +
      "import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query';\n" +
      '\n' +
      'const apiOne = createApi({\n' +
      '  // highlight-start\n' +
      "  reducerPath: 'apiOne',\n" +
      '  // highlight-end\n' +
      "  baseQuery: fetchBaseQuery({ baseUrl: '/' }),\n" +
      '  endpoints: (builder) => ({\n' +
      '    // ...endpoints\n' +
      '  }),\n' +
      '});\n' +
      '\n' +
      'const apiTwo = createApi({\n' +
      '  // highlight-start\n' +
      "  reducerPath: 'apiTwo',\n" +
      '  // highlight-end\n' +
      "  baseQuery: fetchBaseQuery({ baseUrl: '/' }),\n" +
      '  endpoints: (builder) => ({\n' +
      '    // ...endpoints\n' +
      '  }),\n' +
      '});\n' +
      '```',
    optional: true,
    type: 'string'
  },
  serializeQueryArgs: { optional: true },
  endpoints: {},
  keepUnusedDataFor: { optional: true, type: 'number' },
  refetchOnMountOrArgChange: [ { type: 'number' }, { type: 'boolean' }, { type: 'forbidden' } ],
  refetchOnFocus: { type: 'boolean', optional: true },
  refetchOnReconnect: { type: 'boolean', optional: true }
}
```

interface `CreateApiOptions` from `src/query/createApi.ts`
