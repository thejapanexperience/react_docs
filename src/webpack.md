# Webpack #
#### Links ####
[Docs](https://webpack.js.org/)

[Concepts](https://webpack.js.org/concepts/)

[Tutorial](http://webpack.js.org/get-started/)

#### Notes ####

Add `public/` to .eslintignore as bundle.js doesn't need to be linted

webpack.config.js
```javascript
const config = {
  context: __dirname,
  ```
`__dirname` means webpack can be run from anywhere and it will always refer to the root directory
```javascript
  entry: ['./js/ClientApp.jsx'],
  devtool: process.env.NODE_ENV === 'development' ? 'cheap-eval-source-map' : false,
  ```
`devtool: 'cheap-eval-source-map` allows errors to point to the location in the code instead of into the bundle.js
```javascript
  output: {
    path: path.resolve(__dirname, 'public'),
    filename: 'bundle.js',
    publicPath: '/public/'
  },
  devServer: {
    hot: true,
    publicPath: '/public/',
    historyApiFallback: true
  },
```
`historyApiFallback: true` will allow the webpack dev server to work with BrowserRouter.
```javascript
  resolve: {
    extensions: ['.js', '.jsx', '.json'],
    alias: {
      react: 'preact-compat',
      'react-dom': 'preact-compat'
    }
  },
  stats: {
    colors: true,
    reasons: true,
    chunks: false
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
```
`enforce: 'pre'` ensures code passes eslint before reaching babel. This is because we don't care about linting the code once it's converted to es5 by babel.
```javascript
      {
        test: /\.jsx?$/,
        loader: 'babel-loader',
        include: [path.resolve('js'), path.resolve('node_modules/preact-compat/src')]
      }
    ]
  }
};

if (process.env.NODE_ENV === 'development') {
  config.entry.unshift('webpack-hot-middleware/client?path=/__webpack_hmr&timeout=20000');
}

module.exports = config;

```
#### Webpack Dev Server ####
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
Make sure the path starts with a slash to set it as the relative path to the root of the project so the dev server will serve assets correctly.

[HTML File Paths](https://www.w3schools.com/html/html_filepaths.asp)
