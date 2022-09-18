# Laravel: active records

version: `laravel/lumen-framework": "^8.2"`

`Eloquent`

## Default table name from Model name

`illuminate/database/Eloquent/Model.php`:

```
public function getTable() {
  return $this->table ?? Str::snake(Str::pluralStudly(class_basename($this)));
}
```

## scope...Search?

`illuminate/database/Eloquent/Model.php`:

```
public function callNamedScope($scope, array $parameters = [])
{
  return $this->{'scope'.ucfirst($scope)}(...$parameters);
}
```

`illuminate/database/Eloquent/Builder.php`:

```
public function scopes($scopes) {
  $builder->callNamedScope($scope, Arr::wrap($parameters));
}
```
