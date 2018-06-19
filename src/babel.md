# Babel #
.babelrc
```javascript
{
  "presets": [
    "react",
    ["env", {
      "targets": {
        "browsers": "last 2 versions"
      },
      "loose": true,
      "modules": false
    }]
  ],
  "plugins": [
    "babel-plugin-transform-class-properties"
  ],
}
```
`"presets"` install a number of plugins at once.

`"env"` allows you to stay recent and not be transpiling for too many old browser versions.

`"loose"` if set to true allows for more efficient transpiling but may run into errors with edge cases.

`"modules": false` prevents babel from changing es6 module imports into require statements and therefore allows webpack to do its 'tree-shaking' live-code inclusion.

`"babel-plugin-transform-class-properties"` means you don't have to bind functions to `this` inside the constructor in ES6 Class Components.
