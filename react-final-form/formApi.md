# react-final-form

## Как обновляется formApi `6.5.3`

`ReactFinalForm.js:`

```ts
function ReactFinalForm({
    // . . . . .
      form: alternateFormApi,
}): {
    // . . . . .

    const form: FormApi<FormValues> = useConstant(() => {
      const f = alternateFormApi || createForm<FormValues>(config)
      // pause validation until children register all fields on first render (unpaused in useEffect() below)
      f.pauseValidation()
      return f
    })

    useWhenValueChanges(onSubmit, () => {
        form.setConfig('onSubmit', onSubmit)
    })
}
```