# React: StrictMode

## code v17.0.2

### packages/react/src/React.js

```
import {
  REACT_STRICT_MODE_TYPE,
} from 'shared/ReactSymbols';

export {
  REACT_STRICT_MODE_TYPE as StrictMode,
}
```

### packages/shared/ReactSymbols.js

```
export let REACT_STRICT_MODE_TYPE = 0xeacc;


if (typeof Symbol === 'function' && Symbol.for) {
  REACT_STRICT_MODE_TYPE = symbolFor('react.strict_mode');
}
```

### packages/react-reconciler/src/ReactFiber.new.js

```
getTag: switch (type) {
  case REACT_STRICT_MODE_TYPE:
    fiberTag = Mode;
    mode |= StrictMode;
    break;
```

### packages/react-reconciler/src/ReactTypeOfMode.js

```
export const StrictMode = 0b00001;
```

### packages/react-reconciler/src/ReactChildFiber.new.js

```
(returnFiber.mode & StrictMode || warnAboutStringRefs) &&

// . . . . .
if (!didWarnAboutStringRefs[componentName]) {
  if (warnAboutStringRefs) {
    console.error(
      'Component "%s" contains the string ref "%s". Support for string refs ' +
```
