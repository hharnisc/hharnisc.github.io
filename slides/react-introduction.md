class: center, middle
# <img src="/images/slides/react-introduction/react-icon.png" width="50px"/> .react[React] Intro
.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]
---
# What's this talk about?
- .react[React] History and Background
- Data Flow
- Props vs. State
- Component Lifecycle
- Build A Component Together
---
# History
- Publicly Released In 2013
- Started as a client side port of XHP (PHP focused on mitigating XSS attacks)
- Facebook feed and entire Instagram web app built with .react[React]
---
# What Is .react[React]?
- The **V** in MVC
- Renders to a virtual DOM
- Allows you to describe how a component looks at a given state
- Follows a unidirectional data flow pattern (Von Neumann Model)
- Encourages separation of concerns
---
class: fifty-fifty
.left-panel[
# Typical App Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/initial-dom.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# Typical App Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/shared-step-1.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# Typical App Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/update-indicator.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# Typical App Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/direct-dom-step-2.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# Typical App Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/direct-dom-step-3.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# Typical App Data Flow
]
.right-panel[
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
]
---
class: fifty-fifty
.left-panel[
# .react[React] Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/initial-dom.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# .react[React] Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/shared-step-1.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# .react[React] Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/update-indicator.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# .react[React] Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/react-step-2.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# .react[React] Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/react-step-3.png" width="100%"/>
]
---
class: fifty-fifty
.left-panel[
# .react[React] Data Flow
]
.right-panel[
<img src="/images/slides/react-introduction/react-step-4.png" width="100%"/>
]
---
# .react[React] Data Flow
- Data flows predictably and deterministically
- Only need to reason about parts of the app
- Components can be written in complete isolation
- Testing gets easier, since components can be isolated
---
class: center, middle, inverse
# Let's build a *simple* .react[React] component
---
class: fifty-fifty
.left-panel[
# Extend Base .react[React] Component
]
.right-panel[
```javascript
import React, { Component } from 'react';

export default class MyTextComponent extends Component {};
```
]
---
class: fifty-fifty
.left-panel[
# Add render
]
.right-panel[
```javascript
import React, { Component } from 'react';

export default class MyTextComponent extends Component {
  render () {
    return(
        <div>Hello!</div>
    );
  }
};
```
]
---
class: fifty-fifty
.left-panel[
# Add A .green[Prop]
]
.right-panel[
```javascript
import React, { Component } from 'react';

export default class MyTextComponent extends Component {
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
# .green[Props] vs. .blue[State]
---
# Commonality
- Both are **plain JS objects**
- Both trigger a **render** when changed
- Both are **deterministic**, If your component generates different outputs for the same combination of .green[Props] + .blue[State] *you're doing it wrong*
---
class: center, middle, blue-slide
# So .green[Props] Or .blue[State]?
---
# .green[Props]
- A component's **configuration**
- They are received from the outside (parent component)
- **Immutable** within the component
---
# .blue[State]
- Starts with an initial value and goes through a series of mutations
- Mutations are typically generated from user events (text input, clicks, etc.)
- When reading the state you're getting a snapshot at a point in time
- A component manages it's own state internally
- Does not set the state of it's children
- Conceptually similar to *private* in other languages
---
# Should you use .blue[State]?
- .blue[State] increases complexity and reduces predictability
- Stateless components are preferable
- It's not always possible to avoid using .blue[State]
---
class: middle, center
- | _**.green[Props]**_ | _**.blue[State]**_ |
--- | --- | ---
Can get initial value from parent Component? | Yes | Yes
Can be changed by parent Component? | Yes | **No**
Can set default values inside Component?* | Yes | Yes
Can change inside Component? | **No** | Yes
Can set initial value for child Components? | Yes | Yes
Can change in child Components? | Yes | **No**
---
class: center, middle, inverse-slide
# Let's add some .blue[State] to our component!
---
class: fifty-fifty
.left-panel[
# Add Hover .blue[State]
]
.right-panel[
```javascript
import React, { Component } from 'react';

export default class MyTextComponent extends Component {
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
class: fifty-fifty
.left-panel[
# Add style
]
.right-panel[
```javascript
import React, { Component } from 'react';

export default class MyTextComponent extends Component {
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
class: fifty-fifty
.left-panel[
# Add Focus and Blur Handlers
]
.right-panel[
```javascript
import React, { Component } from 'react';

export default class MyTextComponent extends Component {
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
# Component Lifecycle
<img src="/images/slides/react-introduction/complete-lifecycle.png" width="100%"/>
---
# render
- Invoked when component is initially mounted
- Invoked when .green[Props] change
- Invoked when .blue[State] changes

<img src="/images/slides/react-introduction/render.png" width="75%"/>
---
# componentWillMount
- Invoked once before the initial **render** occurs
- Setting .blue[State] here will be observed in the initial render

<img src="/images/slides/react-introduction/componentWillMount.png" width="20%"/>
---
# componentDidMount
- Invoked once once after the initial **render** occurs
- Child **refs** are available in this part of the lifecycle
- Common to see integrations with other JS frameworks here and AJAX requests

<img src="/images/slides/react-introduction/componentDidMount.png" width="20%"/>
---
# componentWillReceiveProps
- Invoked when the component is receiving new .green[Props]
- Not called on the initial **render**
- Setting .blue[State] here does not trigger another render

<img src="/images/slides/react-introduction/componentWillReceiveProps.png" width="25%"/>
---
# shouldComponentUpdate
- Invoked before rendering when new .green[Props] or .blue[State] are received
- Not called on the initial **render**
- **Important**: Return false when you're certain a transition to the new .green[Props] and .blue[State] do not require a re-render

<img src="/images/slides/react-introduction/shouldComponentUpdate.png" width="65%"/>
---
# componentWillUpdate
- Invoked immediately before rendering when new .green[Props] or .blue[State] have been received
- Not called on the initial render
- Used when you need to perform any prep before a render

<img src="/images/slides/react-introduction/componentWillUpdate.png" width="65%"/>
---
# componentWillUnmount
- Invoked immediately before the component is removed from the DOM
- Used for cleanup tasks (things like invalidating timers)

<img src="/images/slides/react-introduction/componentWillUnmount.png" width="40%"/>
---
# Component Lifecycle
- render
- componentWillMount
- componentDidMount
- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- componentDidUpdate
- componentWillUnmount
---
class: center, middle, inverse-slide
# Questions?
---
# Sources
- https://github.com/uberVU/react-guide/blob/master/props-vs-state.md
- https://facebook.github.io/react/docs/component-specs.html
- http://tylermcginnis.com/reactjs-tutorial-a-comprehensive-guide-to-building-apps-with-react/
