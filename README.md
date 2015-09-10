React
=====

*Mostly reasonable patterns for writing React on Rails*

## Table of Contents

1. [Scope](#scope)
1. Organization
  1. [Component Organization](#component-organization)
  1. [Formatting Props](#formatting-props)
1. Patterns
  1. [Computed Props](#computed-props)
  1. [Compound State](#compound-state)
  1. [prefer-ternary-to-sub-render](#prefer-ternary-to-sub-render)
  1. [View Components](#view-components)
  1. [Container Components](#container-components)
1. Anti-patterns
  1. [Compound Conditions](#compound-conditions)
  1. [Cached State in render](#cached-state-in-render)
  1. [Existence Checking](#existence-checking)
  1. [Setting State from Props](#setting-state-from-props)
1. Practices
  1. [Naming Handle Methods](#naming-handler-methods)
  1. [Naming Events](#naming-events)
  1. [Using PropTypes](#using-proptypes)
  1. [Using Entities](#using-entities)
1. Gotchas
  1. [Tables](#tables)
1. Libraries
  1. [classnames](#classnames)
1. Other
  1. [JSX](#jsx)
  1. [ES6 Harmony](#es6-harmony)
  1. [react-rails](#react-rails)
  1. [rails-assets](#rails-assets)
  1. [flux](#flux)

---

## Scope

This is how we write [React.js](https://facebook.github.io/react/) on Rails.
We've struggled to find the happy path. Recommendations here represent a good
number of failed attempts. If something seems out of place, it probably is;
let us know what you've found.

All examples written in ES2015 syntax now that the
[official react-rails gem](https://github.com/reactjs/react-rails) ships with
[babel](http://babeljs.io/).

**[⬆ back to top](#table-of-contents)**

---

## Component Organization

* class definition
  * constructor
    * event handlers
  * 'component' lifecycle events
  * getters
  * render
* defaultProps
* proptypes

```javascript
class Person extends React.Component {
  constructor (props) {
    super(props);

    this.state = { smiling: false };

    this.handleClick = () => {
      this.setState({smiling: !this.state.smiling});
    };
  }

  componentWillMount () {
    // add event listeners (Flux Store, WebSocket, document, etc.)
  },

  componentDidMount () {
    // React.getDOMNode()
  },

  componentWillUnmount () {
    // remove event listeners (Flux Store, WebSocket, document, etc.)
  },

  get smilingMessage () {
    return (this.state.smiling) ? "is smiling" : "";
  }

  render () {
    return (
      <div onClick={this.handleClick}>
        {this.props.name} {this.smilingMessage}
      </div>
    );
  },
}

Person.defaultProps = {
  name: 'Guest'
};

Person.propTypes = {
  name: React.PropTypes.string
};
```

**[⬆ back to top](#table-of-contents)**

## Formatting Props

Wrap props on newlines for exactly 2 or more.

```html
// bad
<Person
 firstName="Michael" />

// good
<Person firstName="Michael" />
```

```html
// bad
<Person firstName="Michael" lastName="Chan" occupation="Designer" favoriteFood="Drunken Noodles" />

// good
<Person
 firstName="Michael"
 lastName="Chan"
 occupation="Designer"
 favoriteFood="Drunken Noodles" />
```

**[⬆ back to top](#table-of-contents)**

---

## Computed Props

Use getters to name computed properties.

```javascript
  // bad
  firstAndLastName () {
    return `${this.props.firstName} ${this.props.lastname}`;
  }

  // good
  get fullName () {
    return `${this.props.firstName} ${this.props.lastname}`;
  }
```

See: [Cached State in render](#cached-state-in-render) anti-pattern

**[⬆ back to top](#table-of-contents)**

---

## Compound State

Prefix compound state getters with a verb for readability.

```javascript
// bad
happyAndKnowsIt () {
  return this.state.happy && this.state.knowsIt;
}
```

```javascript
// good
get isHappyAndKnowsIt () {
  return this.state.happy && this.state.knowsIt;
}
```

These methods *MUST* return a `boolean` value.

See: [Compound Conditions](#compound-conditions) anti-pattern

**[⬆ back to top](#table-of-contents)**

## Prefer Ternary to Sub-render

Keep login inside the `render`.

```javascript
// bad
renderSmilingStatement () {
  return <strong>{(this.state.isSmiling) ? " is smiling." : ""}</strong>;
},

render () {
  return <div>{this.props.name}{this.renderSmilingStatement()}</div>;
}
```

```javascript
// good
render () {
  return (
    <div>
      {this.props.name}
      {(this.state.smiling)
        ? <span>is smiling</span>
        : null
      }
    </div>
  );
}
```

**[⬆ back to top](#table-of-contents)**

## View Components

Compose components into views. Don't create one-off components that merge layout
and domain components.

```javascript
// bad
class PeopleWrappedInBSRow extends React.Component {
  render () {
    return (
      <div className="row">
        <People people={this.state.people} />
      </div>
    );
  }
}
```

```javascript
// good
class BSRow extends React.Component {
  render () {
    return <div className="row">{this.props.children}</div>;
  }
}

class SomeView extends React.createClass {
  render () {
    return (
      <BSRow>
        <People people={this.state.people} />
      </BSRow>
    );
  }
}
```

**[⬆ back to top](#table-of-contents)**

## Container Components

> A container does data fetching and then renders its corresponding
> sub-component. That's it. &mdash; Jason Bonta

#### Bad

```javascript
// CommentList.js

class CommentList extends React.Component {
  getInitialState () {
    return { comments: [] };
  }

  componentDidMount () {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: function(comments) {
        this.setState({comments: comments});
      }.bind(this)
    });
  }

  render () {
    return (
      <ul>
        {this.state.comments.map(({body, author}) => {
          return <li>{body}—{author}</li>;
        })}
      </ul>
    );
  }
}
```

#### Good

```javascript
// CommentList.js

class CommentList extends React.Component {
  render() {
    return (
      <ul>
        {this.props.comments.map(({body, author}) => {
          return <li>{body}—{author}</li>;
        })}
      </ul>
    );
  }
}
```

```javascript
// CommentListContainer.js

class CommentListContainer extends React.Component {
  getInitialState () {
    return { comments: [] }
  }

  componentDidMount () {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: function(comments) {
        this.setState({comments: comments});
      }.bind(this)
    });
  }

  render () {
    return <CommentList comments={this.state.comments} />;
  }
}
```

[Read more](https://medium.com/@learnreact/container-components-c0e67432e005)  
[Watch more](https://www.youtube.com/watch?v=KYzlpRvWZ6c&t=1351)

**[⬆ back to top](#table-of-contents)**

---

## Cached State in `render`

Do not keep state in `render`

```javascript
// bad
render () {
  let name = `Mrs. ${this.props.name}`;

  return <div>{name}</div>;
}

// good
render () {
  return <div>{`Mrs. ${this.props.name}`}</div>;
}
```

```javascript
// best
get fancyName () {
  return `Mrs. ${this.props.name}`;
}

render () {
  return <div>{this.fancyName}</div>;
}
```

*This is mostly stylistic and keeps diffs nice. I doubt that there's a significant perf reason to do this.*

See: [Computed Props](#computed-props) pattern

**[⬆ back to top](#table-of-contents)**

## Compound Conditions

Don't put compound conditions in `render`.

```javascript
// bad
render () {
  return <div>{if (this.state.happy && this.state.knowsIt) { return "Clapping hands" }</div>;
}
```

```javascript
// better
get isTotesHappy() {
  return this.state.happy && this.state.knowsIt;
},

render() {
  return <div>{(this.isTotesHappy) && "Clapping hands"}</div>;
}
```

The best solution for this would use a [container
component](#container-components) to manage state and
pass new state down as props.

See: [Compound State](#compound-state) pattern

**[⬆ back to top](#table-of-contents)**

## Existence Checking

Do not check existence of `prop` objects. Use `defaultProps`.

```javascript
// bad
render () {
  if (this.props.person) {
    return <div>{this.props.person.firstName}</div>;
  } else {
    return null;
  }
}
```

```javascript
// good
class MyComponent extends React.Component {
  render() {
    return <div>{this.props.person.firstName}</div>;
  }
}

MyComponent.defaultProps = {
  person: {
    firstName: 'Guest'
  }
};
```

This is only where objects or arrays are used. Use PropTypes.shape to clarify
the types of nested data expected by the component.

**[⬆ back to top](#table-of-contents)**

## Setting State from Props

Do not set state from props without obvious intent.

```javascript
// bad
getInitialState () {
  return {
    items: this.props.items
  };
}
```

```javascript
// good
getInitialState () {
  return {
    items: this.props.initialItems
  };
}
```

Read: ["Props in getInitialState Is an Anti-Pattern"](http://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html)

**[⬆ back to top](#table-of-contents)**

---

## Naming Handler Methods

Name the handler methods after their triggering event.

```javascript
// bad
punchABadger () { /*...*/ },

render () {
  return <div onClick={this.punchABadger} />;
}
```

```javascript
// good
handleClick () { /*...*/ },

render () {
  return <div onClick={this.handleClick} />;
}
```

Handler names should:

- begin with `handle`
- end with the name of the event they handle (eg, `Click`, `Change`)
- be present-tense

If you need to disambiguate handlers, add additional information between
`handle` and the event name. For example, you can distinguish between `onChange`
handlers: `handleNameChange` and `handleAgeChange`. When you do this, ask
yourself if you should be creating a new component.

**[⬆ back to top](#table-of-contents)**

## Naming Events

Use custom event names for ownee events.

```javascript
class Owner extends React.Component {
  handleDelete () {
    // handle Ownee's onDelete event
  }

  render () {
    return <Ownee onDelete={this.handleDelete} />;
  }
}

class Ownee extends React.Component {
  render () {
    return <div onChange={this.props.onDelete} />;
  }
}

Ownee.propTypes = {
  onDelete: React.PropTypes.func.isRequired
};
```

**[⬆ back to top](#table-of-contents)**

## Using PropTypes

Use PropTypes to communicate expectations and log meaningful warnings.

```javascript
MyValidatedComponent.propTypes = {
  name: React.PropTypes.string
};
```
`MyValidatedComponent` will log a warning if it receives `name` of a type other than `string`.


```html
<Person name=1337 />
// Warning: Invalid prop `name` of type `number` supplied to `MyValidatedComponent`, expected `string`.
```

Components may also require `props`.

```javascript
MyValidatedComponent.propTypes = {
  name: React.PropTypes.string.isRequired
}
```

This component will now validate the presence of name.

```html
<Person />
// Warning: Required prop `name` was not specified in `Person`
```

Read: [Prop Validation](http://facebook.github.io/react/docs/reusable-components.html#prop-validation)

**[⬆ back to top](#table-of-contents)**

## Using Entities

Use React's `String.fromCharCode()` for special characters.

```javascript
// bad
<div>PiCO · Mascot</div>

// nope
<div>PiCO &middot; Mascot</div>

// good
<div>{'PiCO ' + String.fromCharCode(183) + ' Mascot'}</div>

// better
<div>{`PiCO ${String.fromCharCode(183)} Mascot`}</div>
```

Read: [JSX Gotchas](http://facebook.github.io/react/docs/jsx-gotchas.html#html-entities)

**[⬆ back to top](#table-of-contents)**

## Tables

The browser thinks you're dumb. But React doesn't. Always use `tbody` in
`table` components.

```javascript
// bad
render () {
  return (
    <table>
      <tr>...</tr>
    </table>
  );
}

// good
render () {
  return (
    <table>
      <tbody>
        <tr>...</tr>
      </tbody>
    </table>
  );
}
```

The browser is going to insert `tbody` if you forget. React will continue to
insert new `tr`s into the `table` and confuse the heck out of you. Always use
`tbody`.

**[⬆ back to top](#table-of-contents)**

## classnames

Use [classNames](https://www.npmjs.com/package/classnames) to manage conditional classes.

```javascript
// bad
get classes () {
  let classes = ['MyComponent'];

  if (this.state.active) {
    classes.push('MyComponent--active');
  }

  return classes.join(' ');
}

render () {
  return <div className={this.classes} />;
}
```

```javascript
// good
render () {
  let classes = {
    'MyComponent': true,
    'MyComponent--active': this.state.active
  };

  <div className={classnames(classes)} />;
}
```

Read: [Class Name Manipulation](http://facebook.github.io/react/docs/class-name-manipulation.html)

**[⬆ back to top](#table-of-contents)**

## JSX

We used to have some hardcore CoffeeScript lovers is the group. The unfortunate
thing about writing templates in CoffeeScript is that it leaves you on the hook
when certain implementations changes that JSX would normally abstract.

We no longer recommend using CoffeeScript to write `render`.

For posterity, you can read about how we used CoffeeScript, when using CoffeeScript was
non-negotiable: [CoffeeScript and JSX](https://slack-files.com/T024L9M0Y-F02HP4JM3-80d714).

**[⬆ back to top](#table-of-contents)**

## ES2015

[react-rails](https://github.com/reactjs/react-rails) now ships with [babel](babeljs.io). Anything
you can do in Babel, you can do in Rails. See the documentation for additional config.

**[⬆ back to top](#table-of-contents)**

## react-rails

[react-rails](https://github.com/reactjs/react-rails) should be used in all
Rails apps that use React. It provides the perfect amount of glue between Rails
conventions and React.

**[⬆ back to top](#table-of-contents)**

## rails-assets
[rails-assets](https://rails-assets.org) should be considered for bundling
js/css assets into your applications. The most popular React-libraries we use
are registered on [Bower](http://bower.io) and can be easily added through
Bundler and react-assets.

**caveats: rails-assets gives you access to bower projects via Sprockets
requires. This is a win for the traditionally hand-wavy approach that Rails
takes with JavaScript. This approach does buy you modularity or the ability to
interop with JS tooling that requires modularity.**

**[⬆ back to top](#table-of-contents)**

## flux

Use [Alt](http://alt.js.org) for flux implementation. Alt is true to the flux
pattern with the best documentation available.

**[⬆ back to top](#table-of-contents)**
