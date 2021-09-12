
class: center, middle

# .twilio[Twilio] Console
### A Large Scale Migration to Jamstack

.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]

---

class: fifty-fifty

.left-panel[

# Intro

]

.right-panel[

- What is the .twilio[Twilio] Console
- Legacy Console and Issues
- New Console and Impact

]

---

class: middle

# What is the .twilio[Twilio] Console?

---

class: inverse, middle

# A SPA that enables customers to build, configure and monitor applications on the .twilio[Twilio] platform.

???

- Everything from SMS to Video to a full on contact center.

---

class: middle

<img src="/images/slides/twilio-console-jamstack-migration/twilio-console-screenshot.png" alt="Twilio Console Screenshot" width="90%" />

???

- lots of products the make up different parts of the Twilio platform

---

class: middle

<img src="/images/slides/twilio-console-jamstack-migration/twilio-console-app-screenshot.png" alt="Twilio Numbers App" width="90%" />

???

- each product is its own app, most of which are maintained by 1 or more teams

---

class: fifty-fifty

.left-panel[
# Scale
]

.right-panel[
- ~350k MAU
- ~4.5M Monthly Page Views
- xxx other metrics different countries
- xxx other scale metric, like bandwidth?
- xxx other scale metric, like time spent per session or total time in console?
]

???

- The take away is that this isn't a huge amount of traffic, but it is very high value traffic from all around the world

---

class: fifty-fifty

.left-panel[
# Complexity
]

.right-panel[
- xxx products
- xxx teams
- xxx engineers
- xxx LOC
]

???

- The take away that there are organizational challanges here due to the size and structure of the company

---

class: middle

# Legacy Console

---

# Legacy Console: Architecture

- SSR index.html with navigation shell and inject content in a few different ways
- Inject JS/CSS into head for client side rendered apps (React, JQuery, etc.)
- Inject downstream rendered HTML inot the content directly for SSR apps (PHP, etc.)

---

class: inverse, middle

# Essentially, legacy Console was a microfrontend architecture stitched together on the backend.

???

- This in of itself is not a bad thing, however the trade offs playing out over time were not ideal for our situation

TODO: would it make sense to put a diagram here instead?

---

# Legacy Console: Issues

- First request always had to go to a US based region to SSR index.html
- When crossing product boundaries a full page refresh is required
- Errors bubble up unpredictably
- Teams have near complete autonomy, all the way down to how to build their application

???

- Always having to go to a US region is slow, especially outside the us
- It's a pretty common flow to buy a number and then use another product like messaging, our customers quite often would have multiple tabs open to mitigate this
- Some applications have errors that are spread out in multiple projects, while others are handled by catch all logic
- 30+ teams means 30+ different ways of doing things
- Org structure did not encourage sharing methods and practices accross teams
- Less time to focus on solving problems for the customer

---

class: middle

# New Console

???

- Improve the customer experiences of the console, globally
- Enable teams to focus on what matters most, solving problems for their customers

---

# New Console: Architecture

- Jamstack hosted on Netlify
- Shared CI/CD pipeline
- Legacy code is iframed into new Console
- Everything is an "Application Package"
  - (code split bundles wrapped in Error Boundaries)

???

- Initial request and all static assets come from the nearest CDN node
- One CI/CD means that you don't need to roll your own or maintain your own system
- The iframe approach allows for incremental migration at the page level
- Code splitting applications keeps the intial payload small
- Error boundaries on the application allows error routing to go to the correct team
