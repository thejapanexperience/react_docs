# JSX #
```javascript
function World() {
    return (
      <h1 color="red">Hello World</h1>
    )
}
```
compiles down to:
```javascript
function World() {
    return (
      React.createElement("h1", {color: "red"}, "Hello World")
    )
}
```
Inside JSX, anything inside curly braces will be treated as JavaScript.
```javascript
function Title(props) {
    return (
      <h1 style={ {color: props.color} }>{props.title}</h1>
    )
}
```
JSX is nearly the same as HTML

HTML                   | JSX
---------------------- | ---
`for`                  | `htmlFor`
`class`                | `className`
`<style color="blue">` | `<style={{color:'blue'}}>`
`<!-- Comment -->`     | `{*/Comment/*}`
