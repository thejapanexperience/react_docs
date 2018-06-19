# Props

The components below are idempotent. To maintain this, pass down variable data such as a date or `Math.random()` in props from a higher-order component or state. This will make these reusable components more deterministic, easier to test, etc.

`props` are immutable. You cannot reassign a value on `props`. E.g. `props.name = 'Bob'` won't work.

Example 1
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import MyTitle from './MyTitle';

const MyFirstComponent = () => {
  return (
    <div>
      <MyTitle title="Props are great!" color="rebeccapurple" />
      <MyTitle title="Use props everywhere!" color="mediumaquamarine" />
      <MyTitle title="Props are the best!" color="peru" />
    </div>
  );
};

ReactDOM.render(<MyFirstComponent />, document.getElementById('app'));
```
```javascript
import React from 'react';

const MyTitle = props => {
  const style = { color: props.color };
  return (
    <div>
      <h1 style={style}>
        {props.title}
      </h1>
    </div>
  );
};

export default MyTitle;
```

Example 2
```javascript
import React from 'react';

const ShowCard = props => (
  <div className="show-card">
    <img alt={`${props.show.title} Show Poster`} src={`/public/img/posters/${props.show.poster}`} />
    <div>
      <h3>{props.show.title}</h3>
      <h4>({props.show.year})</h4>
      <p>{props.show.description}</p>
    </div>
  </div>
);

export default ShowCard;
```
```javascript
import React from 'react'
import ShowCard from './ShowCard'
import data from '../public/data.json'

class Shows extends React.Component{
  render () {
    return (
      <div className='shows'>
        {data.shows.map(show => <ShowCard key={show.id} show={show} />)}
      </div>
    )
  }
}

export default Shows
```
When mapping an array, always include a `key` prop that will be unique to each mapped component. Use some kind of unique identifier like an id. React uses this key to help determine if things have changed and trigger a render. `key` is not passed through in props. It's just for use by React.

Don't use the index of the array. Using the index of an array could end up being screwy if an array was sorted, resulting in the key values not changing, even though the content has, and, therefore, no render being triggered.

#### Default Props
Sets a default value in case the prop isn't passed in. Not required by React but is required by airbnb eslint rules.
```javascript
ComponentName.defaultProps = {
  foo: 'bar'
}
```

