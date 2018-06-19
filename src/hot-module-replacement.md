# Hot Module Replacement

Enables a really tight feedback loop. Less waiting ...

Major advance over hot module reloading as it doesn't require a full page reload when you update a component. It also allows you to update components while maintaining values in state.

 .bablrc top level property (put it first if there are more than one):
 ```javascript
"plugins": [
  "react-hot-loader/babel"
],
 ```
webpack.config:

```javascript
const path = require('path');
const webpack = require('webpack'); // require webpack

module.exports = {
  context: __dirname,
  entry: [ // make into an array
    'react-hot-loader/patch', // put these three in this order
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/only-dev-server', // there are multiple options for type of hot reloader. Check docs
    './js/ClientApp.jsx'
  ],
  devtool: 'cheap-eval-source-map',
  output: {
    path: path.resolve(__dirname, 'public'),
    filename: 'bundle.js',
    publicPath: '/public/' // add this to match publicPath of devServer
  },
  devServer: {
    hot: true, // set dev-server to hot
    publicPath: '/public/',
    historyApiFallback: true
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json']
  },
  stats: {
    colors: true,
    reasons: true,
    chunks: false
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(), // add these plugins
    new webpack.NamedModulesPlugin() // adds the name of the module for debugging purposes
    ],
  module: {
    rules: [
      {
        enforce: 'pre',
        test: /\.jsx?$/,
        loader: 'eslint-loader',
        exclude: /node_modules/
      },
      {
        test: /\.jsx?$/,
        loader: 'babel-loader'
      }
    ]
  }
};
```

We need to split out our top level component to enable hmr. This will also come in handy for code-splitting and server-side rendering.

One file (ClientApp.jsx)...
```javascript
import React from 'react';
import { render } from 'react-dom';
import { BrowserRouter, Route, Switch } from 'react-router-dom';
import Landing from './Landing';
import Search from './Search';

const FourOhFour = () => <h1>404</h1>;

const App = () => (
  <BrowserRouter>
    <div className="app">
      <Switch>
        <Route exact path="/" component={Landing} />
        <Route path="/search" component={Search} />
        <Route component={FourOhFour} />
      </Switch>
    </div>
  </BrowserRouter>
);

render(<App />, document.getElementById('app'));
```
...will become two...

app.jsx
```javascript
import React from 'react';
import { BrowserRouter, Route } from 'react-router-dom';
import Landing from './Landing';
import Search from './Search';

const App = () => (
  <BrowserRouter>
    <div className="app">
      <Route exact path="/" component={Landing} />
      <Route path="/search" component={Search} />
    </div>
  </BrowserRouter>
);

export default App;
```

ClientApp.jsx (top level)
```javascript
import React from 'react';
import { render } from 'react-dom';
import App from './App';

const renderApp = () => {
  render(<App />, document.getElementById('app'));
};
renderApp();

if (module.hot) { // not enabled in production
  module.hot.accept('./App', () => {
    renderApp();
  });
}
```
`module` comes from webpack.

This split makes the top level component (ClientApp.jsx) a good place to do browser related stuff (e.g. Google Analytics).
