# Redux

- [Redux](#redux)
- [Reducers](#reducers)
- [Store](#store)
- [Actions](#actions)
- [Connecting to React](#connecting-to-react)
- [Redux with Flow](#redux-with-flow)
- [Refactoring the Reducers](#refactoring-the-reducers)
- [Redux Dev Tools](#redux-dev-tools)
- [Async Redux / Redux Thunk](#async-redux--redux-thunk)
- [Testing Redux](#testing-redux)
  - [React Components with Redux](#react-components-with-redux)
  - [Testing Reducers](#testing-reducers)
  - [Testing Actions](#testing-actions)
  - [Testing Thunk](#testing-thunk)

Redux is a state management framework.

Vanilla React components have their own state that can be manipulated and we can send data down to child components through props if we want to share state between different components.

So, Redux doesn't need to be included in a project right from the off.

You should introduce it at the point at which it makes your life easier.

Redux adds complexity to any project so should be considered carefully before bringing it in.

At the point where many components are concerned with the same state, consider introducing Redux.

Redux alleviates the '[data tunneling](http://mitchlloyd.com/posts/tunnel-vision)' problem of having to pass data down through props many times to get from a top-level to a bottom-level component.

Redux is based on Flux but is different in that it has only one store. This is awesome and allows cool stuff like Redux Dev Tools.

![Redux Flow](/public/redux_flow.png "Redux Flow")


# Reducers

Reducers are pure functions that take state and an action, then return a new state.

At the top level of a Redux store is a 'root reducer'. This is where we will add all of our separate reducers.

reducers.js
```javascript
import { SET_SEARCH_TERM } from './actions';

const DEFAULT_STATE = {
  searchTerm: ''
};

// This is our first reducer
const setSearchTerm = (state, action) => Object.assign({}, state, { searchTerm: action.payload });

// rootReducer delegates to the separate reducers
const rootReducer = (state = DEFAULT_STATE, action) => {
  switch (action.type) {
    case SET_SEARCH_TERM:
      return setSearchTerm(state, action);
    default:
      return state;
  }
};

export default rootReducer;
```
`state = DEFAULT_STATE` will set state to DEFAULT_STATE if no state is provided.

`Object.assign({}, state, { searchTerm: action.payload })` Object.assign can be used to return an entirely new object so that state itself is never mutated. In this instance, `{}` is being merged with `state` and then the new object is mutated so that `searchTerm` = `action.payload`.

If the payload carried the value of 'foo', then State will now be
```javascript
{
  searchTerm: 'foo'
}
```

# Store

The store is usually pretty lightweight. You can add middleware, etc.

store.js
```javascript
import { createStore } from 'redux';
import reducer from './reducers';

const store = createStore(reducer);

export default store;
```

# Actions

Actions are objects that have a type and a payload. These are Flux Standard Actions. They don't have to be done in this way but it is convention.
```javascript
{
  type: 'foo',
  payload: 'bar'
}
```

One convention is to have a file of constants that are exported instead of using the string literal in each instance. This is not required.

actions.js
```javascript
export const SET_SEARCH_TERM = 'SET_SEARCH_TERM';
```

The action creators will take in some data and return an action that can be consumed by Redux.

This is where we can make calls to external APIs when using Redux.

actionCreators.js
```javascript
import { SET_SEARCH_TERM } from './actions';

export function setSearchTerm(searchTerm) {
  return { type: SET_SEARCH_TERM, payload: searchTerm };
}
```

# Connecting to React

Import `react-redux` and your new store and wrap everything in `router` in `Provider`.
Also import `store` and pass it into `Provider`.

`Provider` puts Redux on `Context`, which makes it available globally inside the React app.
```javascript
import { Provider } from 'react-redux';
import store from './store';

render () {
  return (
    <BrowserRouter>
      <Provider store={store}>
        […]
      </Provider>
    </BrowserRouter>
  )
}
```
It will all look a bit like this:
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
Now lets connect a component to Redux
```javascript
// @flow

import React from 'react';
import { Link } from 'react-router-dom';

const Landing = () => (
  <div className="landing">
    <h1>svideo</h1>
    <input type="text" placeholder="Search" />
    <Link to="/search">or Browse All</Link>
  </div>
);

export default Landing;
```
becomes...
```javascript
// @flow

import React from 'react';
import { connect } from 'react-redux';
import { Link } from 'react-router-dom';
import { setSearchTerm } from './actionCreators';

const Landing = (props: { searchTerm: string, handleSearchTermChange: Function }) => (
  <div className="landing">
    <h1>{props.searchTerm}</h1>
    <input onChange={props.handleSearchTermChange} value={props.searchTerm} type="text" placeholder="Search" />
    <Link to="/search">or Browse All</Link>
  </div>
);

const mapStateToProps = state => ({ searchTerm: state.searchTerm });
const mapDispatchToProps = (dispatch: Function) => ({
  handleSearchTermChange(event) {
    dispatch(setSearchTerm(event.target.value));
  }
});

export default connect(mapStateToProps, mapDispatchToProps)(Landing);
```
`mapStateToProps` allows us to access the parts of the store that we want to access. In this case we return an object. A common mistake is to try to return from the arrow function like this:
```javascript
const mapStateToProps = state => { searchTerm: state.searchTerm };
```
But, this isn't valid javascript as the curly braces indicate a function body, and don't define an object literal. That's why we wrap the curly braces in parentheses like this:
```javascript
const mapStateToProps = state => ({ searchTerm: state.searchTerm });
```

`mapDispatchToProps` makes our actionCreators available to props so that we can use them in the component.

`connect` connects the Redux store to the component's state.

Lets make the same data available in another component.
```javascript
// @flow

import React, { Component } from 'react';
import ShowCard from './ShowCard';
import Header from './Header';

class Search extends Component {
  state = {
    searchTerm: ''
  };
  props: {
    shows: Array<Show>
  };
  handleSearchTermChange = (event: SyntheticKeyboardEvent & { target: HTMLInputElement }) => {
    this.setState({ searchTerm: event.target.value });
  };
  render() {
    return (
      <div className="search">
        <Header searchTerm={this.state.searchTerm} showSearch handleSearchTermChange={this.handleSearchTermChange} />
        <div>
          {this.props.shows
            .filter(
              show =>
                `${show.title} ${show.description}`.toUpperCase().indexOf(this.state.searchTerm.toUpperCase()) >= 0
            )
            .map(show => <ShowCard key={show.imdbID} {...show} />)}
        </div>
      </div>
    );
  }
}

export default Search;
```
becomes...
```javascript
// @flow

import React from 'react';
import { connect } from 'react-redux';
import ShowCard from './ShowCard';
import Header from './Header';

const Search = (props: {
  searchTerm: string, // eslint-disable-line react/no-unused-prop-types
  shows: Array<Show>
}) => (
  <div className="search">
    <Header showSearch />
    <div>
      {props.shows
        .filter(show => `${show.title} ${show.description}`.toUpperCase().indexOf(props.searchTerm.toUpperCase()) >= 0)
        .map(show => <ShowCard key={show.imdbID} {...show} />)}
    </div>
  </div>
);

const mapStateToProps = state => ({
  searchTerm: state.searchTerm
});

export default connect(mapStateToProps)(Search);
```
Now that we don't need to manipulate state, we can make this a stateless functional component instead of an ES6 Class component.

One more...
```javascript
// @flow

import React from 'react';
import { Link } from 'react-router-dom';

const Header = (props: { showSearch?: boolean, handleSearchTermChange?: Function, searchTerm?: string }) => {
  let utilSpace;
  if (props.showSearch) {
    utilSpace = (
      <input onChange={props.handleSearchTermChange} value={props.searchTerm} type="text" placeholder="Search" />
    );
  } else {
    utilSpace = (
      <h2>
        <Link to="/search">
          Back
        </Link>
      </h2>
    );
  }
  return (
    <header>
      <h1>
        <Link to="/">
          svideo
        </Link>
      </h1>
      {utilSpace}
    </header>
  );
};

Header.defaultProps = {
  showSearch: false,
  handleSearchTermChange: function noop() {},
  searchTerm: ''
};

export default Header;
```
becomes...
```javascript
// @flow

import React from 'react';
import { connect } from 'react-redux';
import { Link } from 'react-router-dom';
import { setSearchTerm } from './actionCreators';

const Header = (props: { showSearch?: boolean, handleSearchTermChange: Function, searchTerm: string }) => {
  let utilSpace;
  if (props.showSearch) {
    utilSpace = (
      <input onChange={props.handleSearchTermChange} value={props.searchTerm} type="text" placeholder="Search" />
    );
  } else {
    utilSpace = (
      <h2>
        <Link to="/search">
          Back
        </Link>
      </h2>
    );
  }
  return (
    <header>
      <h1>
        <Link to="/">
          svideo
        </Link>
      </h1>
      {utilSpace}
    </header>
  );
};

Header.defaultProps = {
  showSearch: false
};

const mapStateToProps = state => ({ searchTerm: state.searchTerm });
const mapDispatchToProps = (dispatch: Function) => ({
  handleSearchTermChange(event) {
    dispatch(setSearchTerm(event.target.value));
  }
});

export default connect(mapStateToProps, mapDispatchToProps)(Header);
```
We have removed the `maybe` types from `handleSearchTermChange` and `searchTerm` as they are always going to be read from Redux instead.

# Redux with Flow

Flow can install a number of redux types via flow-typed.

But, we still want to manually add our action creators.

Add to flow-typed/types.js
```javascript
declare type ActionType = 'SET_SEARCH_TERM';

declare type ActionT<A: ActionType, P> = {|
  type: A,
  payload: P
|};

export type Action = ActionT<'SET_SEARCH_TERM', string>;
```
You can keep adding actions here as you create them in the app to work with Redux.

Then, opt-in the redux files with `// @flow`

```javascript
import { SET_SEARCH_TERM } from './actions';

export function setSearchTerm(searchTerm) {
  return { type: SET_SEARCH_TERM, payload: searchTerm };
}
```
becomes...
```javascript
// @flow

import { SET_SEARCH_TERM } from './actions';

export function setSearchTerm(searchTerm: string) {
  return { type: SET_SEARCH_TERM, payload: searchTerm };
}

```

# Refactoring the Reducers

We are going to use the `combineReducers` function which is part of Redux to split apart our reducers from the rootReducer. It also silos the information in the store so that each reducer manages its own piece of state and only has access to that piece.

```javascript
import { SET_SEARCH_TERM } from './actions';

const DEFAULT_STATE = {
  searchTerm: ''
};

const setSearchTerm = (state, action) => Object.assign({}, state, { searchTerm: action.payload });

const rootReducer = (state = DEFAULT_STATE, action) => {
  switch (action.type) {
    case SET_SEARCH_TERM:
      return setSearchTerm(state, action);
    default:
      return state;
  }
};

export default rootReducer;
```
becomes...
```javascript
// @flow

import { combineReducers } from 'redux';
import { SET_SEARCH_TERM } from './actions';

const searchTerm = (state = '', action: Action) => {
  if (action.type === SET_SEARCH_TERM) {
    return action.payload;
  }
  return state;
};

const rootReducer = combineReducers({ searchTerm });

export default rootReducer;
```
We have removed the switch and now we make our reducers with an if statement.

Default state is set inside each reducer for the bit of state we are referencing. `const searchTerm = (state = '', action: Action) => {`

If we name the reducer the same as the bit of state we are changing, then we can use ES6 to do:
```javascript
const rootReducer = combineReducers({ searchTerm })
```
This is equivalent to:
```javascript
const rootReducer = combineReducers({ searchTerm: searchTerm })
```

# Redux Dev Tools
[Docs](http://extension.remotedev.io/) / [Chrome](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=fr)

Time travelling debugging!

Import `compose` and add the compose line below to:

store.js
```javascript
// @flow

import { createStore, compose } from 'redux';
import reducer from './reducers';

const store = createStore(
  reducer,
  compose(
    typeof window === 'object' && typeof window.devToolsExtension !== 'undefined' ? window.devToolsExtension() : f => f
  )
);

export default store;
```

# Async Redux / Redux Thunk

Redux can't handle asynchronous actions out of the box. We can use Redux Thunk among other things to enable this ability.

A thunk is a special term for a function that's returned by another function.

store.js
```javascript
// @flow

import { createStore, compose, applyMiddleware } from 'redux'; // applyMiddleware
import thunk from 'redux-thunk'; //thunk
import reducer from './reducers';

const store = createStore(
  reducer,
  compose(
    applyMiddleware(thunk), // allow redux to use thunks (or other middleware)
    typeof window === 'object' && typeof window.devToolsExtension !== 'undefined' ? window.devToolsExtension() : f => f
  )
);

export default store;
```

actions.js
```javascript
// @flow

export const SET_SEARCH_TERM = 'SET_SEARCH_TERM';
export const ADD_API_DATA = 'ADD_API_DATA';
```

flow-typed/types.js
```javascript
// @flow

export type Show = {
  title: string,
  description: string,
  year: string,
  imdbID: string,
  trailer: string,
  poster: string,
  rating?: string // Added this as a maybe type
};

declare var module: {
  hot: {
    accept(path: string, callback: () => void): void
  }
};

declare type ActionType = 'SET_SEARCH_TERM' | 'ADD_API_DATA'; // put our new action here

declare type ActionT<A: ActionType, P> = {|
  type: A,
  payload: P
|};

export type Action = ActionT<'SET_SEARCH_TERM', string> | ActionT<'ADD_API_DATA', Show>; // and here
```
`ActionT<'ADD_API_DATA', Show>` is an action type of type `ADD_API_DATA` with payload of type `Show`

reducers.js
```javascript
// @flow

import { combineReducers } from 'redux';
import { SET_SEARCH_TERM, ADD_API_DATA } from './actions'; // added ADD_API_DATA

const searchTerm = (state = '', action: Action) => {
  if (action.type === SET_SEARCH_TERM) {
    return action.payload;
  }
  return state;
};

const apiData = (state = {}, action: Action) => { // added our new reducer
  if (action.type === ADD_API_DATA) {
    return Object.assign({}, state, { [action.payload.imdbID]: action.payload });
  }
  return state;
};

const rootReducer = combineReducers({ searchTerm, apiData }); // added new reducer to rootReducer

export default rootReducer;
```
`[action.payload.imdbID]: action.payload` is an ES6 way of doing dynamic key names.

actionCreators.js
```javascript
// @flow

import axios from 'axios'; // axios
import { SET_SEARCH_TERM, ADD_API_DATA } from './actions';

export function setSearchTerm(searchTerm: string) {
  return { type: SET_SEARCH_TERM, payload: searchTerm };
}

export function addAPIData(apiData: Show) { // action creator that updates the store
  return { type: ADD_API_DATA, payload: apiData };
}

export function getAPIDetails(imdbID: string) { // our new asynchronous action creator that returns a thunk
  return (dispatch: Function) => {
    // the thunk
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
Using our async redux from a component
```javascript
// @flow

import React, { Component } from 'react';
import axios from 'axios';
import Header from './Header';
import Spinner from './Spinner';

class Details extends Component {
  state = {
    apiData: { rating: '' }
  };
  componentDidMount() {
    axios.get(`http://localhost:3000/${this.props.show.imdbID}`).then((response: { data: { rating: string } }) => {
      this.setState({ apiData: response.data });
    });
  }
  props: {
    show: Show
  };
  render() {
    const { title, description, year, poster, trailer } = this.props.show;
    let ratingComponent;
    if (this.state.apiData.rating) {
      ratingComponent = <h3>{this.state.apiData.rating}</h3>;
    } else {
      ratingComponent = <Spinner />;
    }
    return (
      <div className="details">
        <Header />
        <section>
          <h1>{title}</h1>
          <h2>({year})</h2>
          {ratingComponent}
          <img src={`/public/img/posters/${poster}`} alt={`Poster for ${title}`} />
          <p>{description}</p>
        </section>
        <div>
          <iframe
            src={`https://www.youtube-nocookie.com/embed/${trailer}?rel=0&amp;controls=0&amp;showinfo=0`}
            frameBorder="0"
            allowFullScreen
            title={`Trailer for ${title}`}
          />
        </div>
      </div>
    );
  }
}

export default Details;
```
becomes...
```javascript
// @flow

import React, { Component } from 'react';
import { connect } from 'react-redux'; // import connect
import { getAPIDetails } from './actionCreators'; // import our async action creator
import Header from './Header';
import Spinner from './Spinner';

class Details extends Component {
  // we removed state as we are now using Redux
  componentDidMount() {
    if (!this.props.rating) { //get data if there's no data
      this.props.getAPIData();
    }
  }
  props: {
    show: Show,
    rating: string, // added from mapStateToProps
    getAPIData: Function // added from mapDispatchToProps
  };
  render() {
    const { title, description, year, poster, trailer } = this.props.show;
    let ratingComponent;
    if (this.props.rating) { // data is in props now, not state as before
      ratingComponent = <h3>{this.props.rating}</h3>;
    } else {
      ratingComponent = <Spinner />;
    }
    return (
      <div className="details">
        <Header />
        <section>
          <h1>{title}</h1>
          <h2>({year})</h2>
          {ratingComponent}
          <img src={`/public/img/posters/${poster}`} alt={`Poster for ${title}`} />
          <p>{description}</p>
        </section>
        <div>
          <iframe
            src={`https://www.youtube-nocookie.com/embed/${trailer}?rel=0&amp;controls=0&amp;showinfo=0`}
            frameBorder="0"
            allowFullScreen
            title={`Trailer for ${title}`}
          />
        </div>
      </div>
    );
  }
}

// ownProps refers to props being handed down from the parent component
// in this case, the parent passes down 'show'
const mapStateToProps = (state, ownProps) => {
  const apiData = state.apiData[ownProps.show.imdbID] ? state.apiData[ownProps.show.imdbID] : {};
  return {
    rating: apiData.rating
  };
};

const mapDispatchToProps = (dispatch: Function, ownProps) => ({
  getAPIData() {
    dispatch(getAPIDetails(ownProps.show.imdbID));
  }
});

export default connect(mapStateToProps, mapDispatchToProps)(Details);
```

# Testing Redux

## React Components with Redux
When we export a component with connect
```javascript
export default connect(mapStateToProps, mapDispatchToProps)(Search);
```
we are exporting the component (Details) inside a connect wrapper. This isn't something we want to do while testing. We don't want to test Redux, we want to test the component itself.

To work around this we can do:
```javascript
export const Unwrapped = Search;
export default connect(mapStateToProps, mapDispatchToProps)(Search);
```
In our spec file...
```javascript
import Search from '../Search';
```
becomes...
```javascript
import Search, { Unwrapped as UnwrappedSearch } from '../Search';
```
We can now refer to the component as `UnwrappedSearch` in the spec.
```javascript
// @flow

import React from 'react';
import { shallow } from 'enzyme';
import preload from '../../data.json';
import Search from '../Search';
import ShowCard from '../ShowCard';

test('Search renders correctly', () => {
  const component = shallow(<Search shows={preload.shows} />);
  expect(component).toMatchSnapshot();
});

test('Search should render correct amount of shows', () => {
  const component = shallow(<Search shows={preload.shows} />);
  expect(component.find(ShowCard).length).toEqual(preload.shows.length);
});

test('Search should render correct amount of shows based on search term', () => {
  const searchWord = 'black';
  const component = shallow(<Search shows={preload.shows} />);
  component.find('input').simulate('change', { target: { value: searchWord } });
  const showCount = preload.shows.filter(
    show => `${show.title} ${show.description}`.toUpperCase().indexOf(searchWord.toUpperCase()) >= 0
  ).length;
  expect(component.find(ShowCard).length).toEqual(showCount);
});
```
becomes...
```javascript
// @flow

import React from 'react';
import { shallow, render } from 'enzyme'; // add 'render' for test 4 below
import { Provider } from 'react-redux'; // add for test 4 below
import { MemoryRouter } from 'react-router-dom'; // add for test 4 below
import store from '../store'; // add for test 4 below
import { setSearchTerm } from '../actionCreators'; // add for test 4 below
import preload from '../../data.json';
import Search, { Unwrapped as UnwrappedSearch } from '../Search';
import ShowCard from '../ShowCard';

test('Search renders correctly', () => {
  const component = shallow(<UnwrappedSearch shows={preload.shows} searchTerm="" />);
  expect(component).toMatchSnapshot();
});

test('Search should render correct amount of shows', () => {
  const component = shallow(<UnwrappedSearch shows={preload.shows} searchTerm="" />);
  expect(component.find(ShowCard).length).toEqual(preload.shows.length);
});

test('Search should render correct amount of shows based on search term – without Redux', () => {
  const searchWord = 'black';
  const component = shallow(<UnwrappedSearch shows={preload.shows} searchTerm={searchWord} />);
  const showCount = preload.shows.filter(
    show => `${show.title} ${show.description}`.toUpperCase().indexOf(searchWord.toUpperCase()) >= 0
  ).length;
  expect(component.find(ShowCard).length).toEqual(showCount); //
});

test('Search should render correct amount of shows based on search term – with Redux', () => {
  const searchWord = 'black';
  store.dispatch(setSearchTerm(searchWord));
  const component = render( // this is now render, not shallow
    <Provider store={store}>
      <MemoryRouter>
        <Search shows={preload.shows} searchTerm={searchWord} />
      </MemoryRouter>
    </Provider>
  );
  const showCount = preload.shows.filter(
    show => `${show.title} ${show.description}`.toUpperCase().indexOf(searchWord.toUpperCase()) >= 0
  ).length;
  expect(component.find('.show-card').length).toEqual(showCount); // search by class due to render instead of shallow
});
```
on ShowCard, add `className="show-card"` to the top level div so our `component.find` above will be able to count the number of components with that class.
```javascript
class ShowCard extends Component {
  shouldComponentUpdate() {
    return false;
  }
  props: Show;
  render() {
    return (
      <Wrapper className="show-card" to={`/details/${this.props.imdbID}`}>
        <Image alt={`${this.props.title} Show Poster`} src={`/public/img/posters/${this.props.poster}`} />
        <div>
          <h3>{this.props.title}</h3>
          <h4>({this.props.year})</h4>
          <p>{this.props.description}</p>
        </div>
      </Wrapper>
    );
  }
}

export default ShowCard;
```
Test 4 ('Search should render correct amount of shows based on search term – with Redux') is a complex test and isn't necessarily a good unit test. In terms of testing the integration with Redux, it is more valid. Test 3 ('Search should render correct amount of shows based on search term – without Redux') is a better option as a unit test as it  tests what that component is doing in a more direct way.

Also, test 4 takes much longer to run as it uses `render` instead of `shallow`.

The takeaway - Adding redux makes React components tougher to test. But, state management becomes easier to test. (see below...)

## Testing Reducers
Redux Dev Tools can write tests for you!

1) In the app you are making, trigger a redux action
2) Open the `Redux` tools in Chrome Dev Tools
3) Open `Inspector`
4) Select the action you want to test
5) Select `Test` to see your generated Jest spec!

```javascript
import reducers from '../../reducers';

test('reducers', () => {
  let state;
  state = reducers({ searchTerm: '', apiData: {} }, { type: 'SET_SEARCH_TERM', payload: 'black' });
  expect(state).toEqual({ searchTerm: 'black', apiData: {} });
});

```
Add these tests to a file, fix lint and path errors and you now have a set of tests for your reducers!
`reducer.spec.js`
```javascript
// @flow

import reducers from '../reducers';

test('SET_SEARCH_TERM', () => { // change test name to name of the action being tested
  const state = reducers({ searchTerm: '', apiData: {} }, { type: 'SET_SEARCH_TERM', payload: 'black' });
  expect(state).toEqual({ searchTerm: 'black', apiData: {} });
});

test('ADD_API_DATA', () => {
  const state = reducers(
    { searchTerm: '', apiData: {} },
    {
      type: 'ADD_API_DATA',
      payload: {
        rating: '1.1',
        title: 'Westworld',
        year: '2016–',
        description: 'Set at the intersection of the near future and the reimagined past, explore a world in which every human appetite, no matter how noble or depraved, can be indulged without consequence.',
        poster: 'ww.jpg',
        imdbID: 'tt0475784',
        trailer: 'eX3u0IlBBO4'
      }
    }
  );
  expect(state).toEqual({
    searchTerm: '',
    apiData: {
      tt0475784: {
        rating: '1.1',
        title: 'Westworld',
        year: '2016–',
        description: 'Set at the intersection of the near future and the reimagined past, explore a world in which every human appetite, no matter how noble or depraved, can be indulged without consequence.',
        poster: 'ww.jpg',
        imdbID: 'tt0475784',
        trailer: 'eX3u0IlBBO4'
      }
    }
  });
});

test('ADD_API_DATA with two shows', () => {
  const state = reducers(
    {
      searchTerm: '',
      apiData: {
        tt0475784: {
          rating: '1.1',
          title: 'Westworld',
          year: '2016–',
          description: 'Set at the intersection of the near future and the reimagined past, explore a world in which every human appetite, no matter how noble or depraved, can be indulged without consequence.',
          poster: 'ww.jpg',
          imdbID: 'tt0475784',
          trailer: 'eX3u0IlBBO4'
        }
      }
    },
    {
      type: 'ADD_API_DATA',
      payload: {
        rating: '6.2',
        title: 'Game of Thrones',
        year: '2011–',
        description: 'Nine noble families fight for control over the mythical lands of Westeros, while a forgotten race returns after being dormant for thousands of years.',
        poster: 'got.jpg',
        imdbID: 'tt0944947',
        trailer: 'giYeaKsXnsI'
      }
    }
  );
  expect(state).toEqual({
    searchTerm: '',
    apiData: {
      tt0475784: {
        rating: '1.1',
        title: 'Westworld',
        year: '2016–',
        description: 'Set at the intersection of the near future and the reimagined past, explore a world in which every human appetite, no matter how noble or depraved, can be indulged without consequence.',
        poster: 'ww.jpg',
        imdbID: 'tt0475784',
        trailer: 'eX3u0IlBBO4'
      },
      tt0944947: {
        rating: '6.2',
        title: 'Game of Thrones',
        year: '2011–',
        description: 'Nine noble families fight for control over the mythical lands of Westeros, while a forgotten race returns after being dormant for thousands of years.',
        poster: 'got.jpg',
        imdbID: 'tt0944947',
        trailer: 'giYeaKsXnsI'
      }
    }
  });
});
```
## Testing Actions
Jest snapshots are json representations of components. We can use this when testing our actionCreators.
`actionCreators.spec.js`
```javascript
// @flow

import { setSearchTerm, addAPIData } from '../actionCreators';

const strangerThings = {
  title: 'Stranger Things',
  year: '2016–',
  description: 'When a young boy disappears, his mother, a police chief, and his friends must confront terrifying forces in order to get him back.',
  poster: 'st.jpg',
  imdbID: 'tt4574334',
  trailer: '9Egf5U8xLo8',
  rating: '8.6'
};

test('setSearchTerm', () => {
  expect(setSearchTerm('New York')).toMatchSnapshot();
});

test('addAPIData', () => {
  expect(addAPIData(strangerThings)).toMatchSnapshot();
});
```

## Testing Thunk
The test below is testing that the correct API is being called and the correct action is being dispatched.
```javascript
// @flow

import moxios from 'moxios';
import { setSearchTerm, addAPIData, getAPIDetails } from '../actionCreators';

const strangerThings = {
  title: 'Stranger Things',
  year: '2016–',
  description: 'When a young boy disappears, his mother, a police chief, and his friends must confront terrifying forces in order to get him back.',
  poster: 'st.jpg',
  imdbID: 'tt4574334',
  trailer: '9Egf5U8xLo8',
  rating: '8.6'
};

test('setSearchTerm', () => {
  expect(setSearchTerm('New York')).toMatchSnapshot();
});

test('addAPIData', () => {
  expect(addAPIData(strangerThings)).toMatchSnapshot();
});

test('getAPIDetails', (done: Function) => {
  const dispatchMock = jest.fn();
  moxios.withMock(() => {
    getAPIDetails(strangerThings.imdbID)(dispatchMock);
    moxios.wait(() => {
      const request = moxios.requests.mostRecent();
      request
        .respondWith({
          status: 200,
          response: strangerThings
        })
        .then(() => {
          expect(request.url).toEqual(`http://localhost:3000/${strangerThings.imdbID}`);
          expect(dispatchMock).toBeCalledWith(addAPIData(strangerThings));
          done();
        });
    });
  });
});
```

