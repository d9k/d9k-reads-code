# Teal

## Five Years of Teal: minimalism versus growth in language design / Hisham

- https://www.youtube.com/watch?v=68F5Qs91izQ

- https://github.com/teal-language/tl/tree/v0.24.4

### :open_file_folder: `tl.tl`:

```lua
local enum VarianceMode
  "covariant"
  "contravariant"
  "bivariant"
  "invariant"
end
```

```lua
local table_types <total>: {TypeName:boolean} = {
   ["array"] = true,
   ["map"] = true,
   ["record"] = true,
   ["interface"] = true,
   ["self"] = true,
   ["emptytable"] = true,
   ["tupletable"] = true,
```

### :open_file_folder: `spec/lang/assignment/to_function_spec.lua`

`"bivariant type checking for functions with optional arguments`

```lua
local fcts: {string:function(val: any, opt?: string)}

fcts['foo2'] = function (val: string, val2: string)  --  in assignment: incompatible number of arguments: got 2 (string, string), expected at least 1 and at most 2 (<any type>, string)
-- . . . . .
fcts['bar2'] = function (val: number, val2: string)  -- in assignment: incompatible number of arguments: got 2 (number, string), expected at least 1 and at most 2 (<any type>, string)
-- . . . . .
```