class: center, middle, inverse-slide

# Making The Case For .purple-text[Frontend Monorepos]

.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]

---

.left-column[

# About Me

]

.right-column-middle[

* Staff Software Engineer at Buffer
* Other things
  ]

---

class: middle, inverse-slide

## .purple-text[Monorepo] - A single repository containing multiple projects that may or may not be related

???

- Could be dependent of each other, ex. React and react-dom
- Might be completely unrelated, like Google's search algorithm and Angular

---

# Why Use A Monorepo?

???

- Tooling
- Cross project changes -- especially true for frontend development
- Simplified organization -- everything in an NPM package

---

# Who Is Using Frontend Monorepos?

- React - https://github.com/facebook/react
- Angular - https://github.com/angular/angular
- Babel - https://github.com/babel/babel
- Jest - https://github.com/facebook/jest
- Github Primer - https://github.com/primer/primer

???

- This list includes project backed by Google, Facebook, Github -- just to name a few

---

# Buffer Frontend Monorepos

- Publish - https://github.com/bufferapp/buffer-publish
- Analyze - https://github.com/bufferapp/buffer-analyze
- Account Managment - https://github.com/bufferapp/buffer-account

???

- We've got a suite products that are using the Monorepo pattern
- It allows us to share common code and tooling

---

class: center, middle

<img src="/images/slides/making-the-case-for-frontend-monorepos/sidebar.png" width="100%"/>
