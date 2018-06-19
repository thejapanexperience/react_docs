# Styled Components

[Styled Components](https://www.styled-components.com/)
| [github](https://github.com/styled-components/styled-components)

[5-minute Intro to Styled Components](https://medium.freecodecamp.org/a-5-minute-intro-to-styled-components-41f40eb7cd55)

[ES6 Template Literals](http://2ality.com/2016/11/computing-tag-functions.html)

Now we can have markup, logic and styling all in one place.

Mskes it much easier to delete unused styles. Really hard to determine what can be safely deleted when using centralised css.

Possible downsides: styles end up being parsed twice. This is non-trivial and this workflow is not fully optimised yet compared to importing an external stylesheet into an html file. Could lead to performance issue. (not an issue if doing server-side rendering)

This can be a full replacement for LESS/SASS.

[Polished](https://polished.js.org/docs/) can be used to add in some SASS-like functionality. (made by the same people as Styled Components)

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

### Keyframe Animations
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

### Link with Styled Components
```javascript
// @flow

import React from 'react';
import styled from 'styled-components';
import { Link } from 'react-router-dom';

// const Wrapper = styled(Link) if no Flow
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
Wrapper is now a Link component so takes a `to` address
```javascript
<Wrapper to={`/details/${props.imdbID}`}>
```
