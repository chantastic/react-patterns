React
=====

*Mostly reasonable patterns for writing React in CoffeeScript.*

## Table of Contents

1. [Caveat](#caveat)
1. Organization
  1. [Component Organization](#component-organization)
  1. [Formatting Props](#formatting-props)
1. Patterns
  1. [Computed Props](#computed-props)
  1. [Compound State](#compound-state)
  1. [Sub-render](#sub-render)
  1. [Transclusion and Layouts](#transclusion-and-layouts)
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
1. Add-ons
  1. [ClassSet](#classset)
1. [JSX](#jsx)

---

## Caveat

These patterns and practices are birthed from our experience writing React on Rails.

We weight the trade-off of bloating components with `get` `is` and sub-`render` methods. While they clutter a components public interfaces, they are a huge maintainability win.

**[⬆ back to top](#table-of-contents)**

---

## Component Organization

Group methods into logical groups.

* mixins
* propTypes
* get methods
* state methods
* lifecycle events
* render
* event handlers
* "private" methods

```coffeescript
Person = React.createClass
  mixins: [MammalMixin]

  propTypes:
    name: React.PropTypes.string

  getInitialState: ->
    smiling: false

  getDefaultProps: ->
    name: ''

  componentWillMount: ->   # add event listeners (Flux Store, WebSocket, document)

  componentDidMount: ->    # data request (XHR)

  componentWillUnmount: -> # remove event listeners

  render: ->
      React.DOM.div
        className: 'Person'
        onClick:   @handleClick,
          @props.name
          " is smiling" if @state.smiling

   handleClick: ->
     @setState(smiling: !@state.smiling)

  # private

  _doSomethingUglyOrOutsideMyConcern: ->
    # Concerns with outside objects,
    # dirty or duplicated implementations,
    # etc.

```

Place `get` methods ([computed props](#computed-props)) after Reacts `getInitialState` and `getDefaultProps`.

Place `has` and `is` methods ([compound state](#compound-state)) after that, respectively.

```coffeescript
Person = React.createClass
  getInitialState: ->

  getDefaultProps: ->

  getFormattedBirthDate: ->

  hasHighExpectations: ->

  isLikelyToBeDissapointedWithSurprisePartyEfforts: ->
```

**[⬆ back to top](#table-of-contents)**

## Formatting Props

Wrap props on newlines for exactly 2 or more.

(*Hint*: Don't separate props with commas)

```coffeescript
# ok
Person({firstName: "Michael"})

# bad
Person({firstName: "Michael", lastName: "Chan", occupation: "Web Developer", favoriteFood: "Drunken Noodles"})

# good
Person
  firstName:    "Michael"
  lastName:     "Chan"
  occupation:   "Web Developer"
  favoriteFood: "Drunken Noodles"
  onChange:     @handleChange
```

**[⬆ back to top](#table-of-contents)**

---

## Computed Props

Name computed prop methods with the `get` prefix.

```coffeescript
# bad
firstAndLastName: ->
  "#{@props.firstName} #{@props.lastname}"

# good
getFullName: ->
  "#{@props.firstName} #{@props.lastname}"
```

See: [Cached State in render](#cached-state-in-render) anti-pattern

**[⬆ back to top](#table-of-contents)**

---

## Compound State

Name compound state methods with the `is` or `has` prefix.

```coffeescript
# bad
happyAndKnowsIt: ->
  @state.happy and @state.knowsIt

# good
isWillingSongParticipant: ->
  @state.happy and @state.knowsIt

hasWorrysomeBehavior: ->
  !@isWillingSongParticipant() and @props.punchesKittens
```

These methods should return a `boolean` value.

See: [Compound Conditions](#compound-conditions) anti-pattern

**[⬆ back to top](#table-of-contents)**

## Sub-render

Use sub-`render` methods to isolate logical chunks of component UI.

```coffeescript
# good
render: ->
  createItem = (itemText) ->
    React.DOM.li(null, itemText)

  React.DOM.ul(null, @props.items.map(createItem))

# better
render: ->
  React.DOM.ul(null, @renderItems())

renderItems: ->
  for itemText in @props.items
    React.DOM.li(null, itemText)
```

**[⬆ back to top](#table-of-contents)**

## Transclusion and Layouts

Use transclusion (i.e., passing children _through_ the component) to wrap components in layout. Don't create one-off components that merge layout and domain components.

``` coffeescript
# bad
PeopleWrappedInBSRow = React.createClass
  render: ->
    React.DOM.div className: 'row',
      People people: @state.people

# good
BSRow = React.createClass
  render: ->
    React.DOM.div
      className: 'row'
        @props.children

SomeHigherView = React.createClass
  render: ->
    React.DOM.div null,
      BSRow null,
        People(people: @state.people),
```

This works nicely for complex components—like Tabs or Tables—where you you might
need to iterate over children and place them within a complex layout.

**[⬆ back to top](#table-of-contents)**

---

## Cached State in `render`

Do not keep state in `render`

```coffeescript
# bad
render: ->
  name = 'Mr. ' + @props.name
  React.DOM.div(null, name)

# good
render: ->
  React.DOM.div(null, 'Mr. ' + @props.name)

# good (complex example)
getFormattedBirthDate: ->
  moment(@props.user.bday).format(LL);

render: ->
  React.DOM.div(null, @getFormattedBirthDate())
```

See: [Computed Props](#computed-props) pattern

**[⬆ back to top](#table-of-contents)**

## Compound Conditions

Do not put compound conditions in `render`.

```coffeescript
#bad
render: ->
  if @state.happy and @state.knowsIt
    React.DOM.div(null, "Knows what it's all about.")

#good
isLikeTotallyHappy: ->
  @state.happy and @state.knowsIt

render: ->
  if @isLikeTotallyHappy()
    React.DOM.div(null, "Knows what it's all about.")
```

See: [Compound State](#compound-state) pattern

**[⬆ back to top](#table-of-contents)**

## Existence Checking

Do not check existence of `prop` objects.

```coffeescript
#bad
render: ->
  if person?.firstName
    React.DOM.div(null, @props.person.firstName)
  else
    null

#good
getDefaultProps: ->
  person:
    firstName: ''

render: ->
  React.DOM.div(null, @props.person.firstName)
```

**[⬆ back to top](#table-of-contents)**

## Setting State from Props

Do not set state from props without obvious intent.

```coffeescript
#bad
getInitialState: ->
  items: @props.items

#good
getInitialState: ->
  items: @props.initialItems
```

Read: ["Props is getInitialState Is an Anti-Pattern"](http://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html)

**[⬆ back to top](#table-of-contents)**

---

## Naming Handler Methods

Name the handler methods after their triggering event.

```CoffeeScript
# bad
render: ->
  React.DOM.div
    onClick: @punchABadger

punchABadger: ->

# good
render: ->
  React.DOM.div
    onClick: @handleClick

handleClick: ->
```

Handler names should:

- begin with `handle`
- end with the name of the event they handle (eg, `Click`, `Change`)
- be present-tense

If you need to disambiguate handlers, add additional information between `handle` and the event name. For example, you can distinguish between `onChange` handlers:  `handleNameChange` and `handleAgeChange`. If you do this, check whether you should actually create another component class.

**[⬆ back to top](#table-of-contents)**

## Naming Events

Use custom event names for components Parent-Child event listeners.

```coffeescript
Parent = React.createClass
  render: ->
    React.DOM.div
      className: 'Parent'
        Child(onCry: handleCry) # custom event `cry`

  handleCry: ->
    # handle childs' cry

Child = React.createClass
  render: ->
    React.DOM.div
      className: 'Child'
      onChange:  @props.onCry # React DOM event
```

**[⬆ back to top](#table-of-contents)**

## Using PropTypes

Use PropTypes to communicate expectations and log meaningful warnings.

```coffeescript
MyValidatedComponent = React.createClass
  propTypes:
    name: React.PropTypes.string
```
This component will log a warning if it receives `name` of a type other than `string`.


```coffeescript
Person({name: 1337})
# Warning: Invalid prop `name` of type `number` supplied to `MyValidatedComponent`, expected `string`.
```

Components may require `props`

```coffeescript
MyValidatedComponent = React.createClass
  propTypes:
    name: React.PropTypes.string.isRequired
```

This component will now validate the presence of name.

```
Person()
# Warning: Required prop `name` was not specified in `Person`
```

Read: [Prop Validation](http://facebook.github.io/react/docs/reusable-components.html#prop-validation)

**[⬆ back to top](#table-of-contents)**

## Using Entities

Use Reacts `String.fromCharCode()` for special characters.

    # bad
    React.DOM.div(null, 'PiCO · Mascot')

    # nope
    React.DOM.div(null, 'PiCO &middot; Mascot')

    # good
    React.DOM.div(null, 'PiCO ' + String.fromCharCode(183) + ' Mascot')

Read: [JSX Gotchas](http://facebook.github.io/react/docs/jsx-gotchas.html#html-entities)

**[⬆ back to top](#table-of-contents)**

## classSet

Use the `classSet()` add-on to manage conditional classes in your app:


```coffeescript
# bad
render: ->
  React.DOM.div
    className: @getClassName()

getClassName: ->
  claasses = ['MyComponent']
  classes.push('MyComponent--active') if @state.active
  classes.join(' ')

# good
render: ->
  classes =
    'MyComponent': true
    'MyComponent--active': @state.active

  React.DOM.div
    className: React.addons.classSet(classes)
```

Read: [Class Name Manipulation](http://facebook.github.io/react/docs/class-name-manipulation.html)

**[⬆ back to top](#table-of-contents)**

## JSX

Don't use JSX or CJSX in CoffeeScript.

```coffeescript
# bad
render: ->
  `(
    <div
     className: "noJSX"
     orClick:   {@handleClick}>
      Save the children.
    </div>
   )`

#good
render: ->
  React.DOM.div
    className: "noJSX"
    onClick:   @handleClick,
      'Save the children.'
```

Read: [CoffeeScript and JSX](https://slack-files.com/T024L9M0Y-F02HP4JM3-80d714) for more on our decision to avoid JSX.

**[⬆ back to top](#table-of-contents)**
