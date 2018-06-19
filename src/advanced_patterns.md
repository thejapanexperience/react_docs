# Advanced Patterns
[Frontend Masters Course](https://frontendmasters.com/courses/advanced-react-patterns/) |
[github](https://github.com/kentcdodds/advanced-react-patterns-v2/tree/frontend-masters)

- [Advanced Patterns](#advanced-patterns)
  - [Functional setState / Updater Function](#functional-setstate--updater-function)
  - [Compound Components](#compound-components)
    - [Flexible Compound Components using the Context API](#flexible-compound-components-using-the-context-api)
  - [Render Props](#render-props)
  - [State Initializers](#state-initializers)
  - [State Reducers](#state-reducers)
  - [Control Props](#control-props)
  - [Provider Pattern](#provider-pattern)
  - [Higher Order Components](#higher-order-components)
  - [Rendux](#rendux)


switch.js - This component is used in the below examples.
```javascript
import './switch.styles.css'
import React from 'react'

const noop = () => {}

class Switch extends React.Component {
  render() {
    const {
      on,
      className = '',
      'aria-label': ariaLabel,
      onClick,
      ...props
    } = this.props
    const btnClassName = [
      className,
      'toggle-btn',
      on ? 'toggle-btn-on' : 'toggle-btn-off',
    ]
      .filter(Boolean)
      .join(' ')
    return (
      <label
        aria-label={ariaLabel || 'Toggle'}
        style={{display: 'block'}}
      >
        <input
          className="toggle-input"
          type="checkbox"
          checked={on}
          onChange={noop}
          onClick={onClick}
          data-testid="toggle-input"
        />
        <span className={btnClassName} {...props} />
      </label>
    )
  }
}

export {Switch}
```
## Functional setState / Updater Function

Calling setState and using previous state in the update can be dangerous with async functions due to the way React works. [See here](https://medium.freecodecamp.org/functional-setstate-is-the-future-of-react-374f30401b6b) and [here](https://codedaily.io/tutorials/40/How-to-use-a-setState-Updater-Function-with-a-Reducer-Pattern).

E.g. In this case, the state is only actually updated once the final setState is called.
```javascript
state = {score : 0};
// multiple setState() calls
increaseScoreBy3 () {
 this.setState({score : this.state.score + 1});
 this.setState({score : this.state.score + 1});
 this.setState({score : this.state.score + 1});
}
```
We can get around this by calling setState with a function instead. setState below takes two parameters, the first is a function that updates the state, the second is an optional callback function that is invoked once the state is updated.
```javascript
import React from 'react'
import {Switch} from '../switch'

class Toggle extends React.Component {
  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => {
        this.props.onToggle(this.state.on)
      },
    )
  render() {
    const {on} = this.state
    return <Switch on={on} onClick={this.toggle} />
  }
}

//
function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return <Toggle onToggle={onToggle} />
}
Usage.title = 'Build Toggle'

export {Toggle, Usage as default}

```
Using an updater function in this way allows the state updating logic to live outside of the ES6 Class in which the state is being updated.


## Compound Components
The [static](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static) components that make up this Toggle component share implicit state with the parent component. This simplifies the implementation of the code.

In the example below, when the state of the toggle component is 'on', then the static components will display appropriately as well.
I.e. the `On` component will display when state `on = true` but not when `on = false`, etc.

[React.Children.map](https://reactjs.org/docs/react-api.html#reactchildrenmap) and [React.cloneElement](https://reactjs.org/docs/react-api.html#cloneelement) are used here to pass on the props from the parent component to the child components.

([this.props.children](https://learn.co/lessons/react-this-props-children))
```javascript
import React from 'react'
import {Switch} from '../switch'

class Toggle extends React.Component {
  static On = ({on, children}) => (on ? children : null)
  static Off = ({on, children}) => (on ? null : children)
  static Button = ({on, toggle, ...props}) => (
    <Switch on={on} onClick={toggle} {...props} />
  )
  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => this.props.onToggle(this.state.on),
    )
  render() {
    return React.Children.map(this.props.children, child =>
      React.cloneElement(child, {
        on: this.state.on,
        toggle: this.toggle,
      }),
    )
  }
}

function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return (
    <Toggle onToggle={onToggle}>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <Toggle.Button />
    </Toggle>
  )
}
Usage.title = 'Compound Components'

export {Toggle, Usage as default}
```
### Flexible Compound Components using the [Context API](https://reactjs.org/docs/context.html)
```javascript
import React from 'react'
import {Switch} from '../switch'

const ToggleContext = React.createContext()

class Toggle extends React.Component {
  static On = ({children}) => (
    <ToggleContext.Consumer>
      {({on}) => (on ? children : null)}
    </ToggleContext.Consumer>
  )
  static Off = ({children}) => (
    <ToggleContext.Consumer>
      {({on}) => (on ? null : children)}
    </ToggleContext.Consumer>
  )
  static Button = props => (
    <ToggleContext.Consumer>
      {({on, toggle}) => (
        <Switch on={on} onClick={toggle} {...props} />
      )}
    </ToggleContext.Consumer>
  )
  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => this.props.onToggle(this.state.on),
    )
  render() {
    return (
      <ToggleContext.Provider
        value={{on: this.state.on, toggle: this.toggle}}
      >
        {this.props.children}
      </ToggleContext.Provider>
    )
  }
}

function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return (
    <Toggle onToggle={onToggle}>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <div>
        <Toggle.Button />
      </div>
    </Toggle>
  )
}
Usage.title = 'Flexible Compound Components'

export {Toggle, Usage as default}
```

## Render Props

## State Initializers

## State Reducers

## Control Props

## Provider Pattern

## Higher Order Components

## Rendux
