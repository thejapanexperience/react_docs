# Snippets #
#### Misc. Useful Snippets
Do second thing if first is true.
```javascript
{is Admin && <h1> Hi Admin</h1>}
```
Map over array to create markup.
```javascript
users.map(user => <h3>{user.name}</h3>)
```
Click event.
```javascript
<button onClick={delete}>Delete</button>
```
Visualise JSON data nicely on the page.
```javascript
<pre><code>{JSON.stringify(jsonData, null, 4)}</code></pre>
```
Filter and map an array to only display components matching a search string.
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
```

#### Stateless Functional Component ####
```javascript
import React from 'react';

const HelloWorld = (props) => {
    return <div>Hello {props.name}</div>
}
```
#### ES6 Class Component ####
```javascript
import React from 'react';

class HelloWorld extends React.Component {
    render() {
        return <div>Hello {props.name}</div>
    }
}
```

#### createClass vs ES Class ####

createClass is old syntax for React components.
```javascript
var createReactClass = require('create-react-class');

var Greeting = createReactClass({
    render: function() {
        return <h1>Hello World</h1>
    }
})
```
```javascript
import React from 'react'

class Greeting extends React.Component {
    render() {
        return <h1>Hello World</h1>
    }
}
```
