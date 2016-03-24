class: center, middle, inverse-slide
# <img src="/images/slides/react-introduction/react-icon.png" width="50px"/> .react-text[React] Intro
.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]
---
## What's this talk about?
- .react-text[React] History and Background
- Data Flow
- Props vs. State
- Component Lifecycle
- Build A Component Together
---
## History
- Publicly Released In 2013
- Started as a client side port of XHP (PHP focused on mitigating XSS attacks)
- Facebook feed and entire Instagram web app built with .react-text[React]
---
## What Is .react-text[React]?
- The **V** in MVC
- Renders to a virtual DOM
- Allows you to describe how a component looks at a given state
- Follows a unidirectional data flow pattern (Von Neumann Model)
- Encourages separation of concerns
---
.left-column[
## Typical App Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/initial-dom.png" width="100%"/>
]
---
.left-column[
## Typical App Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/shared-step-1.png" width="100%"/>
]
---
.left-column[
## Typical App Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/update-indicator.png" width="100%"/>
]
---
.left-column[
## Typical App Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/direct-dom-step-2.png" width="100%"/>
]
---
.left-column[
## Typical App Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/direct-dom-step-3.png" width="100%"/>
]
---
## Typical App Data Flow
- Requires you to understand entire system to make good decisions
- Common pattern is to trigger a change and catch the event somewhere else in the app
```javascript
  $(window).on('my-event', function () {
    // do a thing
  });
  $(window).trigger('my-event');
```
- Requires you to manage event handling yourself
  - Memory leaks
  - Obfuscated data flow patterns
---
.left-column[
## .react-text[React] Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/initial-dom.png" width="100%"/>
]
---
.left-column[
## .react-text[React] Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/shared-step-1.png" width="100%"/>
]
---
.left-column[
## .react-text[React] Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/update-indicator.png" width="100%"/>
]
---
.left-column[
## .react-text[React] Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/react-step-2.png" width="100%"/>
]
---
.left-column[
## .react-text[React] Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/react-step-3.png" width="100%"/>
]
---
.left-column[
## .react-text[React] Data Flow
]
.right-column-middle[
<img src="/images/slides/react-introduction/react-step-4.png" width="100%"/>
]
---
## .react-text[React] Data Flow
- Data flows predictably and deterministically
- Only need to reason about parts of the app
- Components can be written in complete isolation
- Testing gets easier, since components can be isolated
---
class: center, middle, inverse-slide
# Let's build a *simple* .react-text[React] component
---
.left-column[
## Build A Component
### Extend Base .react-text[React] Component
]
.right-column-middle[
```javascript
import React from 'react';

export default class MyTextComponent extends React.Component {};
```
]
---
.left-column[
## Build A Component
### Extend Base .react-text[React] Component
### Add render
]
.right-column-middle[
```javascript
import React from 'react';

export default class MyTextComponent extends React.Component {
  render () {
    return(
        <div>Hello!</div>
    );
  }
};
```
]
---
.left-column[
## Build A Component
### Extend Base .react-text[React] Component
### Add Render
### Add A .green-text[Prop]
]
.right-column-middle[
```javascript
import React from 'react';

export default class MyTextComponent extends React.Component {
  // This is our API
  default propTypes = {
    text: React.PropTypes.string
  };
  render () {
    return(
        <div>{this.props.text}</div>
    );
  }
};
```
]
---
class: center, middle, inverse-slide
# .green-text[Props] vs. .blue-text[State]
---
## Commonality
- Both are **plain JS objects**
- Both trigger a **render** when changed
- Both are **deterministic**, If your component generates different outputs for the same combination of .green-text[Props] + .blue-text[State] *you're doing it wrong*
---
class: center, middle, blue-slide
# So Props Or State?
---
## .green-text[Props]
- A component's **configuration**
- They are received from the outside (parent component)
- **Immutable** within the component
---
## .blue-text[State]
- Starts with an initial value and goes through a series of mutations
- Mutations are typically generated from user events (text input, clicks, etc.)
- When reading the state you're getting a snapshot at a point in time
- A component manages it's own state internally
- Does not set the state of it's children
- Conceptually similar to *private* in other languages
---
## Should you use .blue-text[State]?
- .blue-text[State] increases complexity and reduces predictability
- Stateless components are preferable
- It's not always possible to avoid using .blue-text[State]
---
class: middle
- | _**.green-text[Props]**_ | _**.blue-text[State]**_ |
--- | --- | ---
Can get initial value from parent Component? | Yes | Yes
Can be changed by parent Component? | Yes | **No**
Can set default values inside Component?* | Yes | Yes
Can change inside Component? | **No** | Yes
Can set initial value for child Components? | Yes | Yes
Can change in child Components? | Yes | **No**
---
class: center, middle, inverse-slide
# Let's add some .blue-text[State] to our component!
---
.left-column[
## Build A Component
### Extend Base React Component
### Add Render
### Add A .green-text[Prop]
### Add Hover .blue-text[State]
]
.right-column-middle[
```javascript
import React from 'react';

export default class MyTextComponent extends React.Component {
  default propTypes = {
    text: React.PropTypes.string
  };

  constructor() {
    // initialize state
    this.state = { isHovering: false };
  }

  render () {
    return(
        <div>
            {this.props.text}
        </div>
    );
  }
};
```
]
---
.left-column[
## Build A Component
### Extend Base React Component
### Add Render
### Add A .green-text[Prop]
### Add Hover .blue-text[State]
### Add style
]
.right-column-middle[
```javascript
import React from 'react';

export default class MyTextComponent extends React.Component {
  default propTypes = {
    text: React.PropTypes.string
  };

  constructor() {
    // initialize state
    this.state = { isHovering: false };
  }

  render () {
    let componentStyle = {
      background: this.state.isHovering ? "green" : ""
    };
    return(
        <div
            style={componentStyle}
        >
            {this.props.text}
        </div>
    );
  }
};
```
]
---
.left-column[
## Build A Component
### Extend Base React Component
### Add Render
### Add A .green-text[Prop]
### Add Hover .blue-text[State]
### Add style
### Add Focus and Blur Handlers
]
.right-column-middle[
```javascript
import React from 'react';

export default class MyTextComponent extends React.Component {
  default propTypes = {
    text: React.PropTypes.string
  };

  constructor() {
    // initialize state
    this.state = { isHovering: false };
  }

  handleFocus () {
    this.setState({ isHovering: true });
  }

  handleBlur () {
    this.setState({ isHovering: false });
  }

  render () {
    let componentStyle = {
      background: this.state.isHovering ? "green" : ""
    };
    return(
        <div
            style={componentStyle}
            onFocus={this.handleFocus.bind(this)}
            onBlur={this.handleBlur.bind(this)}
        >
            {this.props.text}
        </div>
    );
  }
};
```
]
---
class: center, middle, inverse-slide
# Component Lifecycle
---
## Component Lifecycle
<img src="/images/slides/react-introduction/complete-lifecycle.png" width="100%"/>
---
## render
- Invoked when component is initially mounted
- Invoked when .green-text[Props] change
- Invoked when .blue-text[State] changes

<img src="/images/slides/react-introduction/render.png" width="75%"/>
---
## componentWillMount
- Invoked once before the initial **render** occurs
- Setting .blue-text[State] here will be observed in the initial render

<img src="/images/slides/react-introduction/componentWillMount.png" width="30%"/>
---
## componentDidMount
- Invoked once once after the initial **render** occurs
- Child **refs** are available in this part of the lifecycle
- Common to see integrations with other JS frameworks here and AJAX requests

<img src="/images/slides/react-introduction/componentDidMount.png" width="30%"/>
---
## componentWillReceiveProps
- Invoked when the component is receiving new .green-text[Props]
- Not called on the initial **render**
- Setting .blue-text[State] here does not trigger another render

<img src="/images/slides/react-introduction/componentWillReceiveProps.png" width="30%"/>
---
## shouldComponentUpdate
- Invoked before rendering when new .green-text[Props] or .blue-text[State] are received
- Not called on the initial **render**
- **Important**: Return false when you're certain a transition to the new .green-text[Props] and .blue-text[State] do not require a re-render

<img src="/images/slides/react-introduction/shouldComponentUpdate.png" width="75%"/>
---
## componentWillUpdate
- Invoked immediately before rendering when new .green-text[Props] or .blue-text[State] have been received
- Not called on the initial render
- Used when you need to perform any prep before a render

<img src="/images/slides/react-introduction/componentWillUpdate.png" width="75%"/>
---
## componentWillUnmount
- Invoked immediately before the component is removed from the DOM
- Used for cleanup tasks (things like invalidating timers)

<img src="/images/slides/react-introduction/componentWillUnmount.png" width="40%"/>
---
.left-column[
## Component Lifecycle
]
.right-column-middle[
- render
- componentWillMount
- componentDidMount
- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- componentDidUpdate
- componentWillUnmount
]
---
class: center, middle, inverse-slide
# Questions?
---
## Sources
- https://github.com/uberVU/react-guide/blob/master/props-vs-state.md
- https://facebook.github.io/react/docs/component-specs.html
- http://tylermcginnis.com/reactjs-tutorial-a-comprehensive-guide-to-building-apps-with-react/
