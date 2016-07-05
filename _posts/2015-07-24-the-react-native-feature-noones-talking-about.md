---
layout: post
title: The React Native Feature No One's Talking About
tags:
- React Native
---

Last week I started digging into the internals of the [React Native](https://github.com/facebook/react-native) project. I was looking to get a better understanding of how to debug and profile React Native apps with Chrome Developer tools. What I learned was a bit surprising and showed huge potential for future app developers.

Here's a little background on how React Native works, at a high level it's broken down into two isolated runtimes.

#### JS Runtime (Application)

The JavaScript runtime is responsible for driving the application logic. This code is executed asynchronously over their bridging mechanism and results are sent back to the native runtime.

#### Native Runtime (UI)

The native runtime is responsible for rendering UI efficiently and emitting events (touch, scroll, zoom etc.). The events are pushed over the bridging mechanism and the UI is updated based on the response.

<img src="/images/posts/react-native-bridge.png" width="70%">

In a typical configuration the JavaScript is being executed on the device (or simulator) with the local JavaScript engine. Due to the decoupled architecture of React Native the JavaScript can be executed either locally or remotely. You can activate remote execution *now* by using the Chrome Developer Tools. This is activated by pressing `⌘+D` in the simulator and then selecting **Debug In Chrome**.

<img src="/images/posts/react-native-debug-tools.png" width="30%">

This triggers the JavaScript application logic to be executed **remotely** inside of the newly opened Chrome tab. This is useful because it allows you to use all of the profiling and debugging tools already baked into Chrome. What's interesting is that React Native enables remotely executed JavaScript to be driving a thin native UI.

> The JavaScript is platform agnostic and can be ran nearly anywhere, from a browser tab to an instance of Node running on AWS.

#### Remote JS Runtime

Here's an overview on how this is achieved inside of a Chrome tab. Digging into React Native's [debugger.html](https://github.com/facebook/react-native/blob/master/packager/debugger.html) (opened when the **Debug In Chrome** is selected) you can see how it's implemented.

##### Establish A Connection Via Websocket

First a connection is established between the native runtime and the remote runtime via Websocket. In the initialization process a `sessionID` is exchanged and the JavaScript runtime is reset to a clean state.

```javascript
// handler for message type `prepareJSRuntime`
'prepareJSRuntime': function(message) {
  window.onbeforeunload = undefined;
  window.localStorage.setItem('sessionID', message.id);
  window.location.reload();
}
```

##### Load Application Logic

The scripts are injected into the runtime so they can be executed later.

```javascript
// handler for message type `executeApplicationScript`
'executeApplicationScript': function(message, sendReply) {
  for (var key in message.inject) {
    window[key] = JSON.parse(message.inject[key]);
  }
  loadScript(message.url, sendReply.bind(null, null));
}

function loadScript(src, callback) {
  var script = document.createElement('script');
  script.type = 'text/javascript';
  script.src = src;
  script.onload = callback;
  document.head.appendChild(script);
}
```

##### Asynchronously Run JavaScript Remotely

After the JavaScript runtime has all of the scripts injected it's ready to start running commands remotely. These commands are sent in batches and the results are reported back to the native runtime over the Websocket.

```javascript
// handler for message type `executeJSCall`
'executeJSCall': function(message, sendReply) {
  var returnValue = null;
  try {
    if (window && window.require) {
      var module = window.require(message.moduleName);
      returnValue = module[message.moduleMethod].apply(module, message.arguments);
    }
  } finally {
    sendReply(JSON.stringify(returnValue));
  }
}
```

When experimenting with the codebase a I updated the `debugger.html` page to visualize the batched JavaScript calls and show the UIs memory and CPU utilization - [PR Can Be Found Here](https://github.com/facebook/react-native/pull/2050)

##### Limitations

I'm sure there's someone out there thinking *you're proposing that I run application logic behind a websocket, isn't that slow?*. Of course there's overhead, there are multiple hops over the Websocket before you get a response from a function call. The point would not be to run everything remotely, just the bits the would get the most benefit from it.

<img src="/images/posts/react-native-bridge-remote.png" width="70%">

#### Why is this cool?

You're not limited to where the mobile application logic must be executed. This provides more options to build better mobile applications. From a Front End developer's perspective the fact that the code is running remotely is *completely abstracted*. They simply see the response from a function call and don't need to be concerned about where the code was executed. It's even conceivable to be executing code on another mobile device (these features are disabled at compile time when not in debug mode for security reasons), which makes possible a whole new class of applications that were either too difficult or not previously possible.

It's been a lot of fun playing around with the React Native codebase, and I think it shows that Facebook has invested time and energy into the project. Got to love open source.
