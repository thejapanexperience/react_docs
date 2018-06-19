# React #

- [React](#react)
    - [Links](#links)
    - [Notes](#notes)
    - [Different types of components](#different-types-of-components)
    - [Some Decisions to make with React](#some-decisions-to-make-with-react)
    - [Conditional Display Logic](#conditional-display-logic)
    - [Lifecycle Methods](#lifecycle-methods)
      - [componentDidMount is a good place to:](#componentdidmount-is-a-good-place-to)
      - [componentWillUnmount is a good place to tidy things up:](#componentwillunmount-is-a-good-place-to-tidy-things-up)
    - [Example of componentDidMount using Axios](#example-of-componentdidmount-using-axios)
    - [React Dev Tools](#react-dev-tools)
    - [Performance](#performance)
    - [Performance Optimisation](#performance-optimisation)
    - [Types of State: UI State vs Application State](#types-of-state-ui-state-vs-application-state)

### Links ####

[reactjs.org](https://reactjs.org/)

[reactjs.org Blog](https://reactjs.org/blog/)

[Tutorial](https://reactjs.org/tutorial/tutorial.html)

[React Workshop](https://github.com/ReactTraining/react-workshop)

[freeCodeCamp React](https://learn.freecodecamp.org/front-end-libraries/react)

[freeCodeCamp Redux](https://learn.freecodecamp.org/front-end-libraries/redux/)

[freeCodeCamp React and Redux](https://learn.freecodecamp.org/front-end-libraries/react-and-redux/)

### Notes ####

Separation of Concerns - Each component is a separate concern. One file contains MVC so is very easy to debug.

React is a library, not a framework.

Flexible: Can be used in web-apps, static sites, mobile, desktop, server-rendered, VR, etc.
Same code, different renderers (react-dom, react-native, react-vr, etc).

Used by Facebook so will be likely to keep being supported by browsers for a long time to come.

Simple, easy to learn API. Don't need to refer to docs.

Precise error messages and simple debugging.
`debugger;`

Huge and active community.

Virtual DOM leads to great performance.

Small library so good for bandwidth constrained situations, e.g. mobile.

Friendly to automated testing.

Majority of components can be plain pure functions. Easy and quick to test.

Uses JS to extend html. No need to learn special syntax like in Angular / Vue.

Performance inside render methods is key as this step will be run each time props change (i.e. a lot). So keep the render methods as performant as possible.

One way data flow; Parents can communicate to child components. Children have no concept of who their parents are :(
This helps a lot with debugging.

### Different types of components
1) Stateless functional components
```javascript
import React from 'react';

const HelloWorld = (props) => {
    return <div>Hello {props.name}</div>
}
```
2) ES6 Class component
```javascript
import React from 'react';

class HelloWorld extends React.Component {
    render() {
        return <div>Hello {props.name}</div>
    }
}
```
Class components must have a render method and the render method must return markup. In a stateless component, props are accessed as `props.foo`. In a class component, `this.props.bar`.


### Some Decisions to make with React ####

*Dev Environment?*
* [Build from scratch](https://btholt.github.io/complete-intro-to-react/)
* [Create React App](https://github.com/btholt/react-redux-workshop) / [github](https://github.com/facebookincubator/create-react-app)
* Create React App can be a great way to begin the project before ejecting it when / if you run into issues with configuration holding you back.

*Types*
* PropTypes
```javascript
import React from 'react'
import PropTypes from 'prop-types'

function Greeting(props) {
    return (
        <h1>Hello {props.name}</h1>
    )
}

Greeting.propTypes = {
    name: PropTypes.string
}
```
* TypeScript
```javascript
import * as React from 'react'
interface Props {
    name: string
}

function Greeting(props : Props) {
    return (
        <h1>Hello {props.name}</h1>
    )
}
```
* Flow
```javascript
// @flow
import React from 'react'

type Props = {
    name: string
}

function Greeting(props: Props) {
    return (
        <h1>Hello {props.name}</h1>
    )
}
```
*State Management*
* Plain React
* Flux
* Redux
* MobX

### Conditional Display Logic

You can't have `if` blocks inside the JSX ([at the moment](https://medium.freecodecamp.org/a-first-look-do-expressions-in-javascript-de-do-do-do-de-da-da-da-fc87f5fe238a)). You could use a ternary. Best practise is to put the conditional above the return and assign the variable outcomes to a variable that you reference inside the JSX.

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

### Lifecycle Methods
[React Docs](https://reactjs.org/docs/react-component.html)

Common
![React Lifecycle](/public/react-lifecycle.png "React Lifecycle")
Including Less Common
![React Lifecycle](/public/react-lifecycle-less-common.png "React Lifecycle")

#### componentDidMount is a good place to:
- do stuff with window
- do stuff with Document
- add event listeners
- make ajax calls
- use 3rd party libraries
```javascript
componentDidMount () {
  // do stuff with window, Document, ajax, other libraries (D3.js), etc
}
```
#### componentWillUnmount is a good place to tidy things up:
- remove event listeners
- disconnect from D3
```javascript
componentWillUnmount () {
  // tidy up
}
```
### Example of componentDidMount using Axios

Spinner will display while awaiting a response from the API.

Spinner.jsx
```javascript
// @flow

import React from 'react';
import styled, { keyframes } from 'styled-components';

const spin = keyframes`
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
`;

const Image = styled.img`
  animation: ${spin} 4s infinite linear;
`;

const Spinner = () => <Image src="/public/img/loading.png" alt="loading indicator" />;

export default Spinner;
```
Details.jsx
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

### [React Dev Tools](https://github.com/facebook/react-devtools)
Select a component with React Dev Tools then put `$r` in Console to access the highlighted component's data.

Then you can do `$r.setState({foo: 'bar'})` to interact with state.

### Performance
[React Docs](https://reactjs.org/docs/optimizing-performance.html#profiling-components-with-the-chrome-performance-tab)
[Debugging React Performance with React 16 and Chrome Devtools](https://building.calibreapp.com/debugging-react-performance-with-react-16-and-chrome-devtools-c90698a522ad)

### Performance Optimisation
If a component is performing poorly...
```javascript
// @flow

import React from 'react';
import styled from 'styled-components';
import { Link } from 'react-router-dom';

const Wrapper = styled((Link: any))`
  width: 32%;
  border: 2px solid #333;
  border-radius: 4px;
  margin-bottom: 25px;
  padding-right: 10px;
  overflow: hidden;
  color: black;
  text-decoration: none;
`;

const Image = styled.img`
  width: 46%;
  float: left;
  margin-right: 10px;
`;

const ShowCard = (props: Show) => (
  <Wrapper to={`/details/${props.imdbID}`}>
    <Image alt={`${props.title} Show Poster`} src={`/public/img/posters/${props.poster}`} />
    <div>
      <h3>{props.title}</h3>
      <h4>({props.year})</h4>
      <p>{props.description}</p>
    </div>
  </Wrapper>
);

export default ShowCard;
```
This component never needs to be updated so we can stop unnecessary renders by using `shouldComponentUpdate()`
```javascript
// @flow

import React, { Component } from 'react';
import styled from 'styled-components';
import { Link } from 'react-router-dom';

const Wrapper = styled((Link: any))`
  width: 32%;
  border: 2px solid #333;
  border-radius: 4px;
  margin-bottom: 25px;
  padding-right: 10px;
  overflow: hidden;
  color: black;
  text-decoration: none;
`;

const Image = styled.img`
  width: 46%;
  float: left;
  margin-right: 10px;
`;

class ShowCard extends Component {
  shouldComponentUpdate() {
    return false;
  }
  props: Show;
  render() {
    return (
      <Wrapper to={`/details/${this.props.imdbID}`}>
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

### Types of State: UI State vs Application State

UI State - state that is specific to how a component is presented, e.g. inFocus: true.
Application State - state that can be used application wide, e.g. data returned from API calls.

If you are using Redux, you don't have to put all state into the store. Just using it to store Application State can be sufficient. Then use setState for local component UI state.



