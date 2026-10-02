# PrimeVue: forms

## For lacking form data typing

 - :beginner: [Resolves | Vue Form Library](https://primevue.org/forms/#resolvers)

### :open_file_folder: `packages/forms/src/form/Form.d.ts`

```ts
export interface FormResolverOptions {
    /**
     * The values of the form fields.
     */
    values: Record<string, any>;
    /**
     * The names of the form fields.
     */
    names: string[] | undefined;
}
```

### What are names of form fields?