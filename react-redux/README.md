# React-redux

## How Redux `<Provider>` works?

 - :beginner: [Todos | Examples | Redux](https://redux.js.org/introduction/examples#todos)

### Example source

- :open_file_folder: `index.js`

```tsx
 <Provider store={store}>
    <App />
  </Provider>,
```

- :open_file_folder: `src/containers/VisibleTodoList.js`

```tsx
import TodoList from '../components/TodoList'
import { connect } from 'react-redux'

/* ... */

const mapStateToProps = state => ({
  todos: getVisibleTodos(state.todos, state.visibilityFilter)
})

const mapDispatchToProps = dispatch => ({
  toggleTodo: id => dispatch(toggleTodo(id))
})

export default connect(mapStateToProps, mapDispatchToProps)(TodoList)
```

### Library source (9.2.8)

- :open_file_folder: `package.json`:

```json
  "peerDependencies": {
    "@types/react": "^18.2.25 || ^19",
    "react": "^18.0 || ^19",
```

- :open_file_folder: `src/components/Provider.tsx`

```
```tsx
const subscription = createSubscription(store)

if (previousState !== store.getState()) {
      subscription.notifyNestedSubs()
```

- :open_file_folder: `src/utils/Subscription.ts`

```tsx
export function createSubscription(store: any, parentSub?: Subscription) {
  function notifyNestedSubs() {
    listeners.notify()
  }
}

function createListenerCollection() {
	return {
	  subscribe(callback: () => void) {
```

:open_file_folder: `src/components/connect.tsx`

```tsx
function connect(
  mapStateToProps,
  mapDispatchToProps,
  mergeProps,
  options
) {

  const wrapWithConnect = (WrappedComponent) => {
    function ConnectFunction(props) {
      const subscribeForReact = React.useMemo(() => {
	    const subscribe = (reactListener: () => void) => {
	      /* . . . . .*/
	      // [see definition below]
	      return subscribeUpdates( /* . . . . .*/ );
          /* . . . . . */
        } // END subscribe()

      }) // END subscribeForReact()
	  /* . . . . .*/
      actualChildProps = React.useSyncExternalStore(
          subscribeForReact,
          /* . . . . . */
	  );

      const renderedWrappedComponent = React.useMemo(() => {
        return <WrappedComponent {...actualChildProps } { /* . . . . . */ } />
      } /* . . . . . */ )
      const renderedChild = React.useMemo(() => {
	      /* . . . . . */ return renderedWrappedComponent
	  } /* . . . . . */)
      /* . . . . . */
    } // END ConnectFunction()
	return renderedChild
  } // END wrapWithConnect()
  return wrapWithConnect
} // END connect()
```

```tsx
function subscribeUpdates(
  /* . . . . . */
  store, subscription, childPropsSelector,
  /* . . . . . */
) {
  const checkForUpdates = () => {
	/* . . . . . */
    const latestStoreState = store.getState()
    /* . . . . . */
	newChildProps = childPropsSelector(
      latestStoreState,
      lastWrapperProps.current,
    )
  } // END checkForUpdates
  /* . . . . . */
  subscription.onStateChange = checkForUpdates
  /* . . . . . */
  checkForUpdates()
```

 - :beginner: [useSyncExternalStore – React](https://react.dev/reference/react/useSyncExternalStore)
	 - React Hook that lets you subscribe to an external store

## How react-redux connect() works with React Component

- :newspaper: [React-Redux: understanding components, containers, actions, and reducers | by Jason Holtkamp | Medium](https://medium.com/@jasondotparse/react-redux-understanding-components-containers-actions-and-reducers-a2f9287bfb92), 2016

### Example source

```tsx
export default connect(mapStateToProps, mapDispatchToProps)(ButtonList)
```

### React version?

Hooks are a new addition in React 16.8

### Library source 6.0.1 (2019)

- :open_file_folder: `package.json`:

```json
  "peerDependencies": {
    "react": "^16.4.0-0",
```

- :open_file_folder: `src/connect/connect.js`

```js
export function createConnect({
  /* . . . . . */
}) {
  return function connect(
    connectHOC = connectAdvanced,
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
	options
  ) {
    return connectHOC(selectorFactory, {
      /* . . . . . */
	})
  }
}
```

- :open_file_folder: `src/components/connectAdvanced.js`

```js
export default function connectAdvanced( /* . . . . . */ ) {
  return function wrapWithConnect(WrappedComponent) {
    /* . . . . . */
    function makeChildElementSelector() {
      let lastChildElement /* . . . . . */
      function selectChildElement(WrappedComponent, childProps, forwardRef) {
        if (
          childProps !== lastChildProps ||
          forwardRef !== lastForwardRef ||
          lastComponent !== WrappedComponent
        ) {
          lastChildProps = childProps
          lastForwardRef = forwardRef
          lastComponent = WrappedComponent
          lastChildElement = (
            <WrappedComponent {...childProps} ref={forwardRef} />
          )
        } // END if
        return lastChildElement
      } // END selectChildElement()
    } // END makeChildElementSelector()

    class Connect extends OuterBaseComponent {
      constructor(props) {
	    /* . . . . . */
        this.selectChildElement = makeChildElementSelector()
	  }

      renderWrappedComponent(value) {
        const { storeState, store } = value
        /* . . . . . */
        let derivedProps = this.selectDerivedProps(
          storeState, wrapperProps,
		  /* . . . . . */
        )
        /* . . . . . */
		return this.selectChildElement(
		  WrappedComponent,
		  derivedProps,
		  forwardedRef
		)
      }

      indirectRenderWrappedComponent(value) {
        return this.renderWrappedComponent(value)
      }

      render() {
        const ContextToUse = /* . . . . . */ Context

        return (
          <ContextToUse.Consumer>
            {this.indirectRenderWrappedComponent}
          </ContextToUse.Consumer>
        )
      } // END render()
    } // END class Connect
  } // END wrapWithConnect()
} // END coonectAdvanced()
```

`<ContextToUse.Consumer>` forces React to rerender when store `state` changes

- :open_file_folder: `src/components/Context.js`

```ts
export const ReactReduxContext = React.createContext(null)
```

:open_file_folder: `src/components/Provider.js`

```js
class Provider extends Component {
  /* . . . . . */
  componentDidMount() {
    /* . . . . . */
    this.subscribe()
  }

  subscribe() {
    const { store } = this.props

    this.unsubscribe = store.subscribe(() => {
      const newStoreState = store.getState()
      this.setState(/* . . . . . */ { storeState: newStoreState })
    } // END unsubscribe()
  } // END subscribe()

   render() {
    const Context = this.props.context || ReactReduxContext

    return (
      <Context.Provider value={this.state}>
        {this.props.children}
      </Context.Provider>
    )
  }
```
