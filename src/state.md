# State

Only a component can modify its own state.

Call `this.setState` to let React know that something has changed.

```javascript
import React, { Component } from 'react';
import preload from '../data.json';
import ShowCard from './ShowCard';

class Search extends Component {
  constructor(props) {
    super(props);

    this.state = {
      searchTerm: ''
    };

    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(event){
    this.setState({
      searchTerm: event.target.value
    })
  }

  render() {
    return (
      <div className="search">
        <header>
          <h1>{this.state.searchTerm}</h1>
          <input onChange={this.handleChange} value={this.state.searchTerm} type="text" placeholder="Search" />
        </header>
        <div>
          {preload.shows.map(show => <ShowCard key={show.imdbID} {...show} />)}
        </div>
      </div>
    );
  }
}

export default Search;
```

Include `this.handleChange = this.handleChange.bind(this);` in the constructor to set the correct context.

### A Better Way
We can avoid binding `this` and remove the constructor completely.

Add an additional plugin to `.babelrc` : `"babel-plugin-transform-class-properties"`

Add `"parser": "babel-eslint"` to `.eslintrc.json`

Now, we have an arrow function that doesn't create a new context so we don't need to bind it.

```javascript
import React, { Component } from 'react';
import preload from '../data.json';
import ShowCard from './ShowCard';

class Search extends Component {

  state = {
      searchTerm: ''
  }

  handleChange = event => {
    this.setState({
      searchTerm: event.target.value
    })
  }

  render() {
    return (
      <div className="search">
        <header>
          <h1>App Title</h1>
          <input onChange={this.handleChange} value={this.state.searchTerm} type="text" placeholder="Search" />
        </header>
        <div>
          {preload.shows
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
