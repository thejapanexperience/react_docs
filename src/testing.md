# Testing / Jest

Jest is well suited to testing React.

Cool jest stuff:

- snapshots

- smart finding of files

- automatic running with Babel

- can run tests in parallel for increased performance

- run last failing test first so you fail faster

`__tests__` folder in root is convention for airbnb. They can also live alongside the files they are testing. (as far as Jest is concerned)

Filenames Jest will pick up automatically: `ComponentName.test.jsx` or `ComponentName.spec.jsx`

### Running Jest
If Jest is globally installed:

`jest`

That's it. Jest will find all of the test/spec files to run by itself.

`jest --watch` to run in watch mode.

`package.json` scripts:
```javascript
"test": "jest",
"test:update": "jest -u"
```

### Snapshot Testing
`npm i --save-dev react-test-renderer`

`Search.test.jsx`
```javasctipt
import React from 'react';
import renderer from 'react-test-renderer';
import Search from '../Search';

test('Search renders correctly', () => {
  const component = renderer.create(<Search />);
  let tree = component.toJSON();
  expect(tree).toMatchSnapshot();
});
```
The first time you run this test, Jest will save a 'snapshot' to compare future tests against. If the markup changes, this test will fail. If you changed the markup on purpose, run the test with a `-u` flag and it will write out a new snapshot.

Commit snapshots to git as they are part of your testing suite.

### Make it work with Eslint
Put `"jest"` in `.eslintrc.json`
```javascript
"env": {
    "es6": true,
    "jest": true,
    "browser": true,
    "node": true
  },
```

### Make it work with Babel
These tests run in node but node doesn't understand `import` so we need to enable Babel for the tests.
.babelrc
```javasctipt
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
  "env": {
    "test": {
      "plugins": ["transform-es2015-modules-commonjs"]
    }
  }
}

```
This tells Babel to run the `transform-es2015-modules-commonjs` plugin when `env` is set to `test`.

At this stage, if you change something in a child component, the test will fail. To fix this, we can use:

### Enzyme for Shallow Rendering

`npm i --save-dev enzyme`

In `package.json` top level:
```javascript
 "jest": {
    "snapshotSerializers": [
      "jest-serializer-enzyme"
    ]
  },
```

Search.test.jsx
```javasctipt
import React from 'react';
import { shallow } from 'enzyme';
import Search from '../Search';

test('Search renders correctly', () => {
  const component = shallow(<Search />);
  expect(component).toMatchSnapshot();
});
```

Enzyme will stub out all of the child components instead of rendering them. It will only fail tests if there is a problem inside the specific component.

### Example
```javascript
import React from 'react';
import { Provider } from 'react-redux';
import { MemoryRouter } from 'react-router-dom';
import { shallow, render } from 'enzyme';
import Search, { Unwrapped as UnwrappedSearch } from './Search';
import store from './store';
import { setSearchTerm } from './actionCreators';
import preload from '../data.json';
import ShowCard from './ShowCard';

describe('Search', () => {

  it('renders correctly', () => {
    const component = shallow(<UnwrappedSearch searchTerm="" shows={preload.shows} />);
    expect(component).toMatchSnapshot();
  });

  it('should render correct amount of shows', () => {
    const component = shallow(<UnwrappedSearch searchTerm="" shows={preload.shows} />);
    expect(component.find(ShowCard).length).toEqual(preload.shows.length);
  });

  it('should render correct amount of shows based on search term', () => {
    const searchWord = 'black';
    const component = shallow(<Search />);
    component.find('input').simulate('change', { target: { value: searchWord } });
    const showCount = preload.shows.filter(
      show => `${show.title} ${show.description}`.toUpperCase().indexOf(searchWord.toUpperCase()) >= 0
    ).length;
    expect(component.find(ShowCard).length).toEqual(showCount);
  })
});
```
<!-- ```javascript
  it('should render correct amount of shows based on search', () => {
    const searchWord = 'New York';
    store.dispatch(setSearchTerm(searchWord));
    const component = render(
      <Provider store={store}>
        <MemoryRouter>
          <Search shows={preload.shows} />
        </MemoryRouter>
      </Provider>
    );
    const showCount = preload.shows.filter(show =>
      `${show.title.toUpperCase()} ${show.description.toUpperCase()}`.includes(searchWord.toUpperCase())
    ).length;
    expect(showCount).toEqual(component.find('.show-card').length);
  });
``` -->

If you don't want to run a test or a whole suite, add an `x` to the start of the line:
`xit` , `xtest`, `xdescribe`.

With Enzyme, `component.find(ShowCard)` and `component.find('input')` both work. It can search for React components and CSS selectors.

### Test Coverage
Included as part of jest. `--coverage`

![picture alt](/public/test-coverage.png "Test Coverage")

This shows the number of times each line of code is run. Lines that are run many times should be considered for optimisation. (e.g. line 28 is run 60x in only two renders!)

Some thoughts on testing (from btholt [Frontend Masters Complete Intro to React v3](https://btholt.github.io/complete-intro-to-react/)):
* If the UI is changing regularly, you may not want to write so many tests (or if there's lots of A/B testing going on). Business logic should be tested thoroughly though.
* Coverage is not the be-all end-all. Low coverage is probably bad. High coverage is good but doesn't necessarily mean you have good tests...


