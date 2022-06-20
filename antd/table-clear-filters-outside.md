# antd table: Clear filters from outside

## antd code v. 4.18.8

## `components/table/hooks/useFilter/FilterDropdown.tsx`:

```
dropdownContent = column.filterDropdown({
  // . . . .
  clearFilters: onReset,
```

```
  const onReset = () => {
    setSearchValue('');
    setFilteredKeysSync([]);
  };
```
