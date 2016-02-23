
# redux-subscribe

[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](https://github.com/feross/standard)

Subscribe to a path in your redux state atom

## Installation

    $ npm install redux-subscribe

## Usage

redux-subscribe is a modular primitive to providing a context-like abstraction in a component framework. It gives you the ability to dispatch an action in response to the change of a path in your global state atom. This allows you to create a `Context` component which will dispatch an action that updates its own local state in response to a change in the global state atom. The context component may then pass down that piece of state to its children, e.g.

```javascript
function render ({props}) {
  return (
    <Context path='url'>
      {
        url => <a href={props.href} class={{active: url === href}}>{props.text}</a>
      }
    </Context>
  )
}
```

## Example `<Context/>` implementation (in [vdux](https://github.com/vdux/vdux))


```javascript
import {subscribe, unsubscribe} from 'redux-subscribe'

/**
 * Setup the subscription on create, so that it dispatches a local STORE_VALUE
 * action anytime the path we're interested in changes.
 */

function onCreate ({props, key, local}) {
  return subscribe(props.path, key, local(storeValue))
}

/**
 * Render all children that are functions by passing them the current value
 * of the path we're watching
 */

function render ({children, state}) {
  return children.map(child => isFunction(child) ? child(state.value) : child)
}

/**
 * If the path we want to watch changes, cancel the old subscription
 * and create a new one
 */

function onUpdate (prev, next) {
  return [
    unsubscribe(prev.props.path, prev.key),
    subscribe(next.props.path, next.key, local(storeValue))
  ]
}

/**
 * Preserve the global value in our local state
 */

function reducer (state, action) {
  if (action.type === 'STORE_VALUE') {
    return {
      ...state,
      value: action.payload
    }
  }

  return state
}

function storeValue (path, prev, next) {
  return {
    type: 'STORE_VALUE',
    payload: next
  }
}

/**
 * Remove the subscription when the component is removed
 */

function onRemove ({props, key}) {
  return unsubscribe(props.path, key)
}

export default {
  onCreate,
  render,
  reducer,
  onRemove
}
```

## License

MIT
