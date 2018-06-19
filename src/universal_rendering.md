# Universal Rendering

aka Isomorphic Rendering

- [Universal Rendering](#universal-rendering)
  - [Server-Side Rendering](#server-side-rendering)
  - [Separating Client and Server Code](#separating-client-and-server-code)
  - [Hot Module Reloading with Server-Side Rendering](#hot-module-reloading-with-server-side-rendering)
  - [Code Splitting](#code-splitting)
    - [Async Routing](#async-routing)
  - [Building for Production](#building-for-production)
    - [Build Step](#build-step)
    - [Preact](#preact)

## Server-Side Rendering

Not required. But can improve perceived load time.

Full markup is sent from the server then React will work behind the scenes to make the page interactive.

It's about perception, not real-world performance.

Hopefully, by the time the user makes a decision to click on something, the button will be active...

This requires Node to handle the JavaScript. For non-Node servers, can use a Node 'Middle-End' to handle this.

Server-side rendering helps with Google rankings / SEO.

Server-side rendering with vanilla react is pretty simple.

Adding routing makes it more complex as you don't want to have to re-write the client-side routing (react-router) on the server-side.

We want React-Router to handle both.

## Separating Client and Server Code

`<BrowserRouter>` below depends on a browser. This can't be run in Node. There's no `Document` or `Window` etc.

We want to move `<BrowserRouter>` into the client app.

App.jsx
```javascript
// @flow

import React from 'react';
import { BrowserRouter, Route, Switch } from 'react-router-dom';
import type { Match } from 'react-router-dom';
import { Provider } from 'react-redux';
import store from './store';
import Landing from './Landing';
import Search from './Search';
import Details from './Details';
import preload from '../data.json';

const FourOhFour = () => <h1>404</h1>;

const App = () => (
  <BrowserRouter>
    <Provider store={store}>
      <div className="app">
        <Switch>
          <Route exact path="/" component={Landing} />
          <Route path="/search" component={props => <Search shows={preload.shows} {...props} />} />
          <Route
            path="/details/:id"
            component={(props: { match: Match }) => {
              const selectedShow = preload.shows.find(show => props.match.params.id === show.imdbID);
              return <Details show={selectedShow} {...props} />;
            }}
          />
          <Route component={FourOhFour} />
        </Switch>
      </div>
    </Provider>
  </BrowserRouter>
);

export default App;
```
ClientApp.jsx
```javascript
// @flow

import React from 'react';
import { render } from 'react-dom';
import App from './App';

const renderApp = () => {
  render(<App />, document.getElementById('app'));
};
renderApp();

if (module.hot) {
  module.hot.accept('./App', () => {
    renderApp();
  });
}
```
becomes...

ClientApp.jsx
```javascript
// @flow

import React from 'react';
import { render } from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

const renderApp = () => {
  render(
    <BrowserRouter>
      <App />
    </BrowserRouter>,
    document.getElementById('app')
  );
};
renderApp();

if (module.hot) {
  module.hot.accept('./App', () => {
    renderApp();
  });
}
```
App.jsx
```javascript
// @flow

import React from 'react';
import { Route, Switch } from 'react-router-dom';
import type { Match } from 'react-router-dom';
import { Provider } from 'react-redux';
import store from './store';
import Landing from './Landing';
import Search from './Search';
import Details from './Details';
import preload from '../data.json';

const FourOhFour = () => <h1>404</h1>;

const App = () => (
  <Provider store={store}>
    <div className="app">
      <Switch>
        <Route exact path="/" component={Landing} />
        <Route path="/search" component={props => <Search shows={preload.shows} {...props} />} />
        <Route
          path="/details/:id"
          component={(props: { match: Match }) => {
            const selectedShow = preload.shows.find(show => props.match.params.id === show.imdbID);
            return <Details show={selectedShow} {...props} />;
          }}
        />
        <Route component={FourOhFour} />
      </Switch>
    </div>
  </Provider>
);

export default App;
```
Add `<div id="app"><%= body %></div>` to index.html. This is [Lodash-specific templating](https://lodash.com/docs#template). This templating can be done in many ways.

index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>svideo</title>
  <link rel="stylesheet" href="/public/style.css">
</head>
<body>
	<div id="app"></div>
	<script src="/public/bundle.js"></script>
</body>
</html>
```
becomes...
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>svideo</title>
  <link rel="stylesheet" href="/public/style.css">
</head>
<body>
	<div id="app"><%= body %></div>
	<script src="/public/bundle.js"></script>
</body>
</html>
```

Add a server plugin to .babelrc to that Node can understand ES6 modules.

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
    "react-hot-loader/babel",
    "babel-plugin-transform-class-properties"
  ],
  "env": {
    "test": {
      "plugins": [
        "transform-es2015-modules-commonjs"
      ]
    }
  }
}
```
becomes...
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
    "react-hot-loader/babel",
    "babel-plugin-transform-class-properties"
  ],
  "env": {
    "test": {
      "plugins": [
        "transform-es2015-modules-commonjs"
      ]
    },
    "server": {
      "plugins": [
        "transform-es2015-modules-commonjs"
      ]
    }
  }
}
```
Make a new file called `server.js` that exists outside of the React code. In this instance, React code is inside a folder called `js` so `server.js` can go in the root. This will create our Node server. We use lodash to process our template that we created above.

server.js
```javascript
/* eslint no-console:0 */
require('babel-register');
// This means that this file won't be run through Babel but everything that is required will be.
// 'babel-node' is an alternative.

const express = require('express');
const React = require('react');
const ReactDOMServer = require('react-dom/server');
const ReactRouter = require('react-router-dom');
const _ = require('lodash');
const fs = require('fs');
const App = require('./js/App').default; // Remember to include the '.default' as we have exported default from this file.

const StaticRouter = ReactRouter.StaticRouter;
// StaticRouter will replace BrowserRouter on the Node Server
const port = 8080;
const baseTemplate = fs.readFileSync('./index.html');
// readFileSync will wait until the file has been read (unlike fs.readFile). This is okay on startup as it only happens once.
// This would be a bad idea if it was happening multiple times...
const template = _.template(baseTemplate);
// template is a function that will return our markup inside our html file.

const server = express(); // We don't have to use express. Could use HAPI, etc.

server.use('/public', express.static('./public'));

server.use((req, res) => {
  console.log(req.url);
  const context = {};
  const body = ReactDOMServer.renderToString( // this will return our React code as a string of html
    React.createElement(StaticRouter, { location: req.url, context }, React.createElement(App)) // can't use JSX here so vanilla React code instead...
  );

// handle redirects...
  if (context.url) {
    res.redirect(context.url);
  }

  res.write(template({ body }));
  res.end();
});

console.log(`listening on ${port}`);
server.listen(port);
```

## Hot Module Reloading with Server-Side Rendering

Note - Generally, it may be better to do server-side rendering just in production and to use Webpack Dev Server for development. In that case, HMR doesn't need to be added to the Node server.


Change the entry point of Webpack so it reads from the Node server instead of the Webpack Dev Server.

webpack.config
```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  context: __dirname,
  entry: [
    'react-hot-loader/patch',
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/only-dev-server',
    './js/ClientApp.jsx'
  ],
  ...
};
```
becomes...
```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  context: __dirname,
  entry: ['webpack-hot-middleware/client?path=__webpack_hmr&timeout=2000', './js/ClientApp.jsx'],
  ...
};
```
server.js
```javascript
/* eslint no-console:0 */
require('babel-register');

const express = require('express');
const React = require('react');
const ReactDOMServer = require('react-dom/server');
const ReactRouter = require('react-router-dom');
const _ = require('lodash');
const fs = require('fs');
// Add these 3 imports for webpack
const webpackDevMiddleware = require('webpack-dev-middleware');
const webpackHotMiddleware = require('webpack-hot-middleware');
const webpack = require('webpack');
const App = require('./js/App').default;
// Add this config
const config = require('./webpack.config');

const StaticRouter = ReactRouter.StaticRouter;
const port = 8080;
const baseTemplate = fs.readFileSync('./index.html');
const template = _.template(baseTemplate);

const server = express();

// Add compiler to instatiate Webpack
const compiler = webpack(config);
server.use(
  webpackDevMiddleware(compiler, {
    publicPath: config.output.publicPath
  })
);
server.use(webpackHotMiddleware(compiler));

server.use('/public', express.static('./public'));

server.use((req, res) => {
  console.log(req.url);
  const context = {};
  const body = ReactDOMServer.renderToString(
    React.createElement(StaticRouter, { location: req.url, context }, React.createElement(App))
  );

  if (context.url) {
    res.redirect(context.url);
  }

  res.write(template({ body }));
  res.end();
});

console.log(`listening on ${port}`);
server.listen(port);
```

## Code Splitting

Note - There is an issue with Webpack when doing server-side rendering, hot module replacement and code splitting at the same time. Pick two. (need to check if this is still an issue) Also, getting code splitting to work with server-side rendering is difficult to do. It's important to think about the benefits of each and decide depending on your needs.

Code splitting allows us to only load the javascript for the components that are needed for a particular page. This cuts down on the bundle size and improves load times. This isn't necessary in development but can be useful in production.

Code splitting is useful if you have multiple routes that are large enough to see some benefit. You can reduce the initial bundle size significantly in such a use-case.

If this isn't the case, then there's not much point in doing code splitting.

### Async Routing

We are going to create a new higher-order component. This will fetch a component that we ask for and then render itself (it can also display a spinner while it's waiting so there is some markup).

It's going to take two things, the props we want to pass to the component, and a loading promise.

The loading promise resolves with an object that has a 'default' property which has a value of a React ES6 Class component.
```javascript
loadingPromise: Promise<{ default: Class<React.Component<*, *, *>> }>
```
The `*`s here are for typing React components with Flow. They allow any kind of generic component to pass the Flow typing.
It takes

AsyncRoute.jsx
```javascript
// @flow

import React, { Component } from 'react';
import Spinner from './Spinner';

class AsyncRoute extends Component {
  state = {
    loaded: false
  };
  componentDidMount() {
    this.props.loadingPromise.then(module => {
      this.component = module.default;
      this.setState({ loaded: true });
    });
  }
  props: {
    props: mixed,
    loadingPromise: Promise<{ default: Class<React.Component<*, *, *>> }>
  };
  component = null;
  render() {
    if (this.state.loaded) {
      return <this.component {...this.props.props} />;
    }
    return <Spinner />;
  }
}

export default AsyncRoute;
```
We can use this Async Route component inside our router so that components are only loaded when they are needed.

.babelrc
```javascript
 "plugins": [
    "react-hot-loader/babel",
    "babel-plugin-transform-class-properties"
  ],
```
becomes...
```javascript
"plugins": [
    "react-hot-loader/babel",
    "babel-plugin-syntax-dynamic-import", // this lets Babel understand the import syntax
    "babel-plugin-dynamic-import-webpack", // this transforms the import statements to require.ensure
    "babel-plugin-transform-class-properties"
  ],
```
We can delete the components from App.jsx and add our AsyncRoute instead.
App.jsx
```javascript
// @flow

import React from 'react';
import { Route, Switch } from 'react-router-dom';
import type { Match } from 'react-router-dom';
import { Provider } from 'react-redux';
import store from './store';
import Landing from './Landing';
import Search from './Search';
import Details from './Details';
import preload from '../data.json';

const FourOhFour = () => <h1>404</h1>;

const App = () => (
  <Provider store={store}>
    <div className="app">
      <Switch>
        <Route exact path="/" component={Landing} />
        <Route path="/search" component={props => <Search shows={preload.shows} {...props} />} />
        <Route
          path="/details/:id"
          component={(props: { match: Match }) => {
            const selectedShow = preload.shows.find(show => props.match.params.id === show.imdbID);
            return <Details show={selectedShow} {...props} />;
          }}
        />
        <Route component={FourOhFour} />
      </Switch>
    </div>
  </Provider>
);

export default App;
```
becomes...
```javascript
// @flow

import React from 'react';
import { Route, Switch } from 'react-router-dom';
import type { Match } from 'react-router-dom';
import { Provider } from 'react-redux';
import store from './store';
import AsyncRoute from './AsyncRoute';
import preload from '../data.json';

const FourOhFour = () => <h1>404</h1>;

const App = () => (
  <Provider store={store}>
    <div className="app">
      <Switch>
        <Route exact path="/" component={props => <AsyncRoute props={props} loadingPromise={import('./Landing')} />} />
        <Route
          path="/search"
          component={props => (
            <AsyncRoute props={Object.assign({ shows: preload.shows }, props)} loadingPromise={import('./Search')} />
          )}
        />
        <Route
          path="/details/:id"
          component={(props: { match: Match }) => {
            const selectedShow = preload.shows.find(show => props.match.params.id === show.imdbID);
            return (
              <AsyncRoute
                props={Object.assign({ show: selectedShow, match: {} }, props)}
                loadingPromise={import('./Details')}
              />
            );
          }}
        />
        <Route component={FourOhFour} />
      </Switch>
    </div>
  </Provider>
);

export default App;
```
## Building for Production

We want to reduce our bundle size for production. Minify, etc.

webpack.config - Refactor to remove hmr functionality if `NODE_ENV === 'production'`.
```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  context: __dirname,
  entry: [
    'react-hot-loader/patch',
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/only-dev-server',
    './js/ClientApp.jsx'
  ],
  devtool: 'cheap-eval-source-map',
  output: {
    path: path.join(__dirname, 'public'),
    filename: 'bundle.js',
    publicPath: '/public/'
  },
  devServer: {
    hot: true,
    publicPath: '/public/',
    historyApiFallback: true
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json']
  },
  stats: {
    colors: true,
    reasons: true,
    chunks: true
  },
  plugins: [new webpack.HotModuleReplacementPlugin(), new webpack.NamedModulesPlugin()],
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
becomes...
```javascript
const path = require('path');
const webpack = require('webpack');

const config = {
  context: __dirname,
  entry: [
    'react-hot-loader/patch',
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/only-dev-server',
    './js/ClientApp.jsx'
  ],
  devtool: 'cheap-eval-source-map',
  output: {
    path: path.join(__dirname, 'public'),
    filename: 'bundle.js',
    publicPath: '/public/'
  },
  devServer: {
    hot: true,
    publicPath: '/public/',
    historyApiFallback: true
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json']
  },
  stats: {
    colors: true,
    reasons: true,
    chunks: true
  },
  plugins: [new webpack.HotModuleReplacementPlugin(), new webpack.NamedModulesPlugin()],
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

console.log(process.env.NODE_ENV);

if (process.env.NODE_ENV === 'production') {
  config.entry = './js/ClientApp.jsx';
  config.devtool = false;
  config.plugins = [];
}

module.exports = config;
```
This step will add gzip [compression](https://github.com/expressjs/compression). This is more typically done at the apache or nginx level, not directly in the node server. Adding it here will let you see the degree of compression that can be achieved.

server.js - add compression
```javascript
/* eslint no-console:0 */
require('babel-register');

const express = require('express');
const React = require('react');
const ReactDOMServer = require('react-dom/server');
const ReactRouter = require('react-router-dom');
const _ = require('lodash');
const fs = require('fs');
// require compression
const compression = require('compression');
const App = require('./js/App').default;

const StaticRouter = ReactRouter.StaticRouter;
const port = 8080;
const baseTemplate = fs.readFileSync('./index.html');
const template = _.template(baseTemplate);

const server = express();

// use compression
server.use(compression());
server.use('/public', express.static('./public'));

server.use((req, res) => {
  console.log(req.url);
  const context = {};
  const body = ReactDOMServer.renderToString(
    React.createElement(StaticRouter, { location: req.url, context }, React.createElement(App))
  );

  if (context.url) {
    res.redirect(context.url);
  }

  res.write(template({ body }));
  res.end();
});

console.log(`listening on ${port}`);
server.listen(port);
```

### Build Step
`npm run build -p`

Running `build` with a `-p` tag will tell webpack to do everything it can to make the code as small as possible.

As a general point, you should try to make your main bundle as small as possible. E.g. move any actions that use Axios into a separate file and require that in any components that need to use that action. This will mean that the Axios library is only included in components that need it, instead of being made available as part of the main bundle.

asyncActions.js
```javascript
// @flow

import axios from 'axios';
import { addAPIData } from './actionCreators';

export default function getAPIDetails(imdbID: string) {
  return (dispatch: Function) => {
    axios
      .get(`http://localhost:3000/${imdbID}`)
      .then(response => {
        dispatch(addAPIData(response.data));
      })
      .catch(error => {
        console.error('axios error', error); // eslint-disable-line no-console
      });
  };
}
```

### Preact
The [Preact](https://preactjs.com/) library is compatible with react but is much smaller (3k vs 45k approximately).

We can set up webpack.config to use Preact instead of React by using an alias.

webpack.config
```javascript
const path = require('path');
const webpack = require('webpack');

const config = {
  context: __dirname,
  entry: [
    'react-hot-loader/patch',
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/only-dev-server',
    './js/ClientApp.jsx'
  ],
  devtool: 'cheap-eval-source-map',
  output: {
    path: path.join(__dirname, 'public'),
    filename: 'bundle.js',
    publicPath: '/public/'
  },
  devServer: {
    hot: true,
    publicPath: '/public/',
    historyApiFallback: true
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json'],
    // add this alias to use preact instead of react
    alias: {
      react: 'preact-compat',
      'react-dom': 'preact-compat'
    }
  },
  stats: {
    colors: true,
    reasons: true,
    chunks: true
  },
  plugins: [new webpack.HotModuleReplacementPlugin(), new webpack.NamedModulesPlugin()],
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
        loader: 'babel-loader',
        // include Preact
        include: [path.resolve('js'), path.resolve('node_modules/preact-compat/src')]
      }
    ]
  }
};

console.log(process.env.NODE_ENV);

if (process.env.NODE_ENV === 'production') {
  config.entry = './js/ClientApp.jsx';
  config.devtool = false;
  config.plugins = [];
}

module.exports = config;
```

