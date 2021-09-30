
class: center, middle

<h1 style="display: flex; align-items: center; justify-content: center; gap: 15px">
  <img
    src="/images/slides/twilio-console-jamstack-migration/twilio-logo.png"
    alt="Twilio Logo"
    width="9%"
    style="margin:-20px"
  />
  .twilio[Twilio] Console
</h1>
## A Large Scale Migration to Jamstack

.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]

---

class: fifty-fifty

.left-panel[

# Harrison Harnisch

]
.right-panel[

## Principal Software <br> Engineer @ .twilio[Twilio]

<img
  src="/images/slides/twilio-console-jamstack-migration/twilio-logo.png"
  alt="Twilio Logo"
  width="20%"
  style="margin-top: -20px; margin-bottom: -20px;"
/>

### [@hjharnis](https://twitter.com/hjharnis)

]

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

# Console: A Brief History

---

class: fifty-fifty

.left-panel[
# Console 1.0
### c. 2008
]

.right-panel[
- Initially 3 products
- One team building everything
- Relatively simple navigation
]

---

class: middle

<img src="/images/slides/twilio-console-jamstack-migration/twilio-console-original.png" alt="Twilio Console Screenshot" width="50%" />

---

class: inverse, middle

# The single team approach hit limits quickly

---

class: fifty-fifty

.left-panel[
# Console 2.0
### c. 2016
]

.right-panel[
- Dozens of products
- Dozens of teams working independently
- Complex navigation, pinning products
]

---

class: middle

<img src="/images/slides/twilio-console-jamstack-migration/twilio-console-2-pinning.gif" alt="Twilio Console Screenshot" width="90%" />

---

class: fifty-fifty

.left-panel[
# Scale
]

.right-panel[
- ~4.5M Monthly Page Views
- ~480 GB Monthly Bandwidth
- Customers in over 200 countries
]

???

- The take away is that this isn't a huge amount of traffic, but it is very high value traffic from all around the world

---

class: fifty-fifty

.left-panel[
# Complexity
]

.right-panel[
- 30+ products
- 30+ teams
- 300+ engineers
]

???

- The take away that there are organizational challanges here due to the size and structure of the company

---

class: fifty-fifty

.left-panel[
# Console 2.0: Architecture
]

.right-panel[
- SSR index.html and inject content
  - SSR content and inject on the server
  - SSR scripts and inject on the client
]

---

class: inverse, middle

# Console 2.0: a microfrontend architecture stitched together on the backend

???

- This in of itself is not a bad thing, however the trade offs playing out over time were not ideal for our situation

---

class: fifty-fifty

.left-panel[
# Console 2.0: Issues
]

.right-panel[
- First render always goes through US
- Crossing products ALWAYS needs full page refresh
- Share development environments `dev` -> `stage` -> `prod`
- Teams have complete autonomy (almost)
]

???

- Always having to go to a US region is slow, especially outside the us
- It's a pretty common flow to buy a number and then use another product like messaging, our customers quite often would have multiple tabs open to mitigate this
- Some applications have errors that are spread out in multiple projects, while others are handled by catch all logic
- 30+ teams means 30+ different ways of doing things
- Org structure did not encourage sharing methods and practices accross teams
- Less time to focus on solving problems for the customer

---

class: inverse, middle

# Dozens of independent teams eventually hit limits
---

class: middle

# Console 3.0
### c. 2021

???

- Improve the customer experiences of the console, globally
- Enable teams to focus on what matters most, solving problems for their customers

---

class: fifty-fifty

.left-panel[
# Architecture
]

.right-panel[
- Jamstack hosted on Netlify
- Monorepo!
  - Shared CI/CD pipeline
  - Console 2.0 code is iframed into new Console
  - Everything is an "Application Package"
]

???

- Initial request and all static assets come from the nearest CDN node
- One CI/CD means that you don't need to roll your own or maintain your own system
- The iframe approach allows for incremental migration at the page level
- Code splitting applications keeps the intial payload small
- Application package = code split bundles wrapped in Error Boundaries
- Error boundaries on the application allows error routing to go to the correct team

---

class: middle

<img src="/images/slides/twilio-console-jamstack-migration/twilio-console-screenshot.png" alt="Twilio Console Screenshot" width="90%" />

???

- lots of products the make up different parts of the .twilio[Twilio] platform

---

class: middle

<img src="/images/slides/twilio-console-jamstack-migration/twilio-console-app-screenshot.png" alt="Twilio Numbers App" width="90%" />

???

- each product is its own app, most of which are maintained by 1 or more teams

---

class: middle

# Console 3.0: Impact

---

class: fifty-fifty

.left-panel[
# Increased Collaboration
]

.right-panel[
- Preview deploys allow for quicker validation on code reviews
- Direct feedback from product and design

<img src="/images/slides/twilio-console-jamstack-migration/twilio-console-featurepeek.png" alt="Twilio Preview Deployment FeaturePeek" width="50%" />
]

???

- To get a review from a teammate you used to have to check out the PR branch and spin up the stack locally
- Teams were either not getting feedback from product and design or would batch it up and do all the reviews right before launch

---


class: fifty-fifty

.left-panel[
# Performance Improvements
]

.right-panel[
- TTFB (global)
- Initial Page Load (global)
- Eliminated full page refresh
]

---

class: fifty-fifty

.left-panel[
# Shared Governance
]

.right-panel[
- Many decisions impact multiple teams, sometimes everyone
- Key stakeholders from teams help make decisions
- Document decisions as we go
]

---

class: fifty-fifty

.left-panel[
# Increased Development Velocity
]

.right-panel[
- Weekly bulk changes < small changes multiple times per day
- Preview deployments = ∞ staging environments
- **More focus on solving customer problems**
]

---

class: middle

# Closing

---

class: middle, inverse

# The Jamstack architecture has caused a shift in how .twilio[Twilio] thinks about frontend applications.

???

- Other teams working outside the Console are starting to plan their migration to a Jamstack architecture
- The Console is moving towards a truely global presence to meet our customers where they are
- It a deeper focus on solving problems for our customers

---

class: middle

# More time to focus on our customers!
