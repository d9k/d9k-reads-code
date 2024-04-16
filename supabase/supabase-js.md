# Supabase-js

`v2.38.2`

- https://github.com/supabase/supabase-js

## How Auth works

### `src/SupabaseClient.ts`:

```ts
const DEFAULT_AUTH_OPTIONS: SupabaseAuthClientOptions = {
  autoRefreshToken: true,
  persistSession: true,
  detectSessionInUrl: true,
  flowType: 'implicit',
}

const defaultStorageKey = `sb-${new URL(this.authUrl).hostname.split('.')[0]}-auth-token`
```

```ts
_initSupabaseAuthClient()
```

### `src/lib/SupabaseAuthClient.ts`:

```ts
import { GoTrueClient } from '@supabase/gotrue-js'

export class SupabaseAuthClient extends GoTrueClient {
```

### `src/lib/types.ts`:

```ts
export interface SupabaseAuthClientOptions extends GoTrueClientOptions {
```