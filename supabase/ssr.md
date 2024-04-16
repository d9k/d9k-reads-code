# Supabase: SSR

- `@supabase/ssr@0.0.10`
- https://github.com/supabase/auth-helpers/tree/main/packages/ssr

## OTP doesn't work. Why?

### `src/createBrowserClient.ts`:

```ts
const cookieClientOptions = {
// . . .
	auth: {
		flowType: 'pkce',
```