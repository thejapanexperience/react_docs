# Flow #

[Flow](https://flow.org/en/) is a javascript type checker.

[Docs](https://flow.org/en/docs/)

[Flow Type Cheat Sheet](https://www.saltycrane.com/flow-type-cheat-sheet/latest/)

The main alternative to Flow is TypeScript, which is a language of its own based on javascript.

Main benefit of using Flow or Proptypes is in documentation. It makes it super obvious what props are required / in use by any component.

Will also help you to write better code with less errors. (great!)

Flow automatically infers type information by doing "flow analysis" so you don't have to declare the types.

### Getting Started

`npm install --save-dev babel-cli babel-preset-flow`


add `"flow"` to babel `"presets"` in `.babelrc`
```javascript
{
  "presets": [
    "flow"
    ]
}
```

add `"plugin:flowtype/recommended"` to `"extends"` and add `"flowtype"` to `"plugins"` in .eslintrc.json
```javascript
{
  "extends": [
    "plugin:flowtype/recommended",
    "airbnb",
    "prettier",
    "prettier/react"
  ],
  "plugins": [
    "flowtype",
    "prettier"
  ],
}
```
and in `"plugins"`, add `"flowtype"`

`npm i --save-dev flow-bin`

Add `"flow": "flow"` to scripts in `package.json`.

`npm run flow init` This will create a file called `.flowconfig`

`npm i -g flow-typed` Add flow-typed CLI.

`flow-typed install` to automatically pull in Flow support for installed libraries.

![flow-typed](/public/flow-typed.png "flow-typed")

This also created the flow-typed folder that contains files with type definitions for all of the installed libraries. [This should be commited to git](https://github.com/flowtype/flow-typed/wiki/FAQs).

We can also install specific versions of definitions: `flow-typed install styled-components@1.4` will install definitions for that version of styled-components.

`npm run flow` to run Flow.

### Ignoring Files
Add to .flowconfig
```javascript
[ignore]
<PROJECT_ROOT>/node_modules/styled-components/*
```

### Using Flow
Flow is 'opt-in'. Each file has to be opted in. This allows you to add Flow incrementally into a mature code-base.

Installing Flow support into your editor is a good idea. There are plugins for VSCode, Atom, Sublime, etc.

Flow type annotations are not valid javascript. `"react"` babel preset contains stuff needed for flow and since we are running babel with webpack, on compiling, the type annotations will be removed.

`// @flow` at top of any file that you want to use flow with.

#### Example

```javascript
handleSearchTermChange = event => {
  this.setState({ searchTerm: event.target.value })
}
```
becomes...
```javascript
handleSearchTermChange = (event: SyntheticKeyboardEvent & { target: HTMLInputElement }) => {
  this.setState({ searchTerm: event.target.value })
}
```
This is saying that `event` will be a React synthetic keyboard event AND that it will have a `target` that will be an HTML input element.

Not everything needs to be annotated in this way. E.g. React components will have their type inferred by Flow. Only things that Flow can't infer types for will need to be explicitly typed.

### Example

```javascript
import React from 'react';
import { string } from 'prop-types';
import styled from 'styled-components';

const Wrapper = styled.div`
  width: 32%;
  border: 2px solid #333;
  border-radius: 4px;
  margin-bottom: 25px;
  padding-right: 10px;
  overflow: hidden;
`;

const Image = styled.img`
  width: 46%;
  float: left;
  margin-right: 10px;
`;

const ShowCard = props => (
  <Wrapper>
    <Image alt={`${props.title} Show Poster`} src={`/public/img/posters/${props.poster}`} />
    <div>
      <h3>{props.title}</h3>
      <h4>({props.year})</h4>
      <p>{props.description}</p>
    </div>
  </Wrapper>
);

ShowCard.propTypes = {
  poster: string.isRequired,
  title: string.isRequired,
  year: string.isRequired,
  description: string.isRequired
};

export default ShowCard;
```
becomes...
```javascript
// @flow

import React from 'react';
import styled from 'styled-components';

const Wrapper = styled.div`
  width: 32%;
  border: 2px solid #333;
  border-radius: 4px;
  margin-bottom: 25px;
  padding-right: 10px;
  overflow: hidden;
`;

const Image = styled.img`
  width: 46%;
  float: left;
  margin-right: 10px;
`;

const ShowCard = (props: { poster: string, title: string, year: string, description: string }) => (
  <Wrapper>
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

[`// $FlowFixMe`](https://medium.com/@daveford/suppress-an-error-in-flow-68b38b032afa) will suppress individual Flow errors. (not recommended...)

### Example
```javascript
// @flow

import React, { Component } from 'react';
import ShowCard from './ShowCard';

class Search extends Component {
  state = {
    searchTerm: ''
  };
  props: {
    shows: Array<Show>
  };
```
`props` here is for typing purposes. We are saying that props will have `shows` which will be an array of things with a type of `Show`.
`Show` is defined in `flow-typed/types.js`
```javascript
// @flow

export type Show = {
  title: string,
  description: string,
  year: string,
  imdbID: string,
  trailer: string,
  poster: string
};

declare var module: {
  hot: {
    accept(path: string, callback: () => void): void
  }
};

```

```javascript
  handleSearchTermChange = (event: SyntheticKeyboardEvent & { target: HTMLInputElement }) => {
    this.setState({ searchTerm: event.target.value });
  };
  render() {
    return (
      <div className="search">
        <header>
          <h1>svideo</h1>
          <input
            onChange={this.handleSearchTermChange}
            value={this.state.searchTerm}
            type="text"
            placeholder="Search"
          />
        </header>
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

### Create Definitions
In `flow-types` directory, create `types.js`

```javascript
// @flow

declare var module: {
  hot: {
    accept(path: string, callback: () => void): void
  }
};
```
This is declaring a global variable `module` that is an object that has an object called `hot`, that has a function called `accept` that takes a path which is a string and a function that returns `void`, i.e. nothing. The `accept` function itself returns `void`.

Now we can use flow with `module.hot.accept` in the following example and not return a Flow error:
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


### Optional Props
If a type is set to optional for a prop (by adding `?`), then you have to include defaultProps (according to airbnb rules).
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

`Function` with a capital `F` is the type for functions in Flow. Lower-case `function` is a protected key word.
