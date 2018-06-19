# React Router #
[React Router](https://reacttraining.com/react-router/)

[Connected React Router - Redux binding for React Router v4](https://github.com/supasate/connected-react-router)

[Beginners Guide to React Router v4](https://medium.freecodecamp.org/beginners-guide-to-react-router-4-8959ceb3ad58)

[React Router v4 Tutorial](https://medium.com/@pshrmn/a-simple-react-router-v4-tutorial-7f23ff27adf)

Not a built in part of React. Used for making a single-page application.

There has been some API thrash in the past, especially moving from v3 to v4. (Something to consider...)

React router interacts with the dom's history API. This allows it to work with back, forward, etc.

```javascript
// Landing.jsx
import React from 'react';
import { Link } from 'react-router-dom';

const Landing = () => (
  <div className="landing">
    <h1>svideo</h1>
    <input type="text" placeholder="Search" />
    <Link to='/search'>or Browse All</Link>
  </div>
);

export default Landing;
```
`<Link to='/search'>or Browse All</Link>` `Link` auto generates an <a> tag to the correct route.
```javascript
// Search.jsx
import React from 'react';

const Search = () => <h1>Search!!</h1>;

export default Search;
```
```javascript
// FourOhFour.jsx
import React from 'react';

const FourOhFour = () => <h1>404!!</h1>;

export default FourOhFour;
```
```javascript
// ClientApp.jsx
import React from 'react';
import { render } from 'react-dom';
import { HashRouter, Route } from 'react-router-dom';
import Landing from './Landing';
import Search from './Search';

const App = () => (
  <HashRouter>
    <div className="app">
        <Route exact path="/" component={Landing} />
        <Route path="/search" component={Search} />
    </div>
  </HashRouter>
);

render(<App />, document.getElementById('app'));
```
`<Route exact path="/" component={Landing} />` `exact` means the component will load on only that route.

If the `exact` was removed, `/foo` would still load the Landing component.

`HashRouter` is a higher-order component. It doesn't render any markup itself. It only contains behaviour.

```javascript
// ClientApp.jsx
import React from 'react';
import { render } from 'react-dom';
import { BrowserRouter, Route, Switch } from 'react-router-dom';
import Landing from './Landing';
import Search from './Search';
import FourOhFour from './FourOhFour';

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
`BrowserRouter`. To use this instead of `HashRouter` (remove nasty '#'s from urls / more SEO friendly), add `historyApiFallback: true` to webpack.config.js devServer.

`<Switch>` acts as a switch to ensure that only one component will be rendered. This prevents the `FourOhFour` component also being loaded when hitting the other routes as it has no route requirement. `FourOhFour` will only be rendered if none of the other path conditions are met.

Dynamic routing can be done by adding a variable in the path, e.g. `path="/:language/search"` or `path="/users/:id"`

### Passing in props through React Router

```javascript
// @flow

import React from 'react';
import { BrowserRouter, Route, Switch } from 'react-router-dom';
import type { Match } from 'react-router-dom'; // This is
import Landing from './Landing';
import Search from './Search';
import Details from './Details';
import preload from '../data.json';

const FourOhFour = () => <h1>404</h1>;

const App = () => (
  <BrowserRouter>
    <div className="app">
      <Switch>
        <Route exact path="/" component={Landing} />
        <Route
          path="/search"
          component={props => <Search shows={preload.shows} {...props} />}
        />
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
  </BrowserRouter>
);

export default App;
```
In the following section of the above example:
```javascript
<Route
  path="/details/:id"
  component={(props: { match: Match }) => {
    const selectedShow = preload.shows.find(show => props.match.params.id === show.imdbID);
    return <Details show={selectedShow} {...props} />;
  }}
/>
```
`props: { match: Match }` `match` is an object on props and `Match` is imported as a Flow type from `react-router-dom` above.

### Routing on Events, e.g. Form Submit
This example is now using Redux. The `goToSearch` function will inject a new url into `history` which is now available on props as of React Router v4 and cause a redirect.

```javascript
// @flow

import React, { Component } from 'react';
import { connect } from 'react-redux';
import { Link } from 'react-router-dom';
import type { RouterHistory } from 'react-router-dom';
import { setSearchTerm } from './actionCreators';

class Landing extends Component {
  props: {
    searchTerm: string,
    handleSearchTermChange: Function,
    history: RouterHistory
  };
  goToSearch = (event: SyntheticEvent) => {
    event.preventDefault();
    this.props.history.push('/search');
  };
  render() {
    return (
      <div className="landing">
        <h1>svideo</h1>
        <form onSubmit={this.goToSearch}>
          <input
            onChange={this.props.handleSearchTermChange}
            value={this.props.searchTerm}
            type="text"
            placeholder="Search"
          />
        </form>
        <Link to="/search">or Browse All</Link>
      </div>
    );
  }
}

const mapStateToProps = state => ({ searchTerm: state.searchTerm });
const mapDispatchToProps = (dispatch: Function) => ({
  handleSearchTermChange(event) {
    dispatch(setSearchTerm(event.target.value));
  }
});

export default connect(mapStateToProps, mapDispatchToProps)(Landing);
```


