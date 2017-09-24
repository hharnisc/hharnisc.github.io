---
layout: post
title: GraphQL Is Architecture
tags:
- GraphQL
- Architecture
---

GraphQL has been gaining popularity since it was open sourced in 2015. It is an interesting implementation of the [Backend For Frontend](http://samnewman.io/patterns/architectural/bff/) pattern built on a custom query/filtering language. It has a strong community behind it ([Github uses it](https://githubengineering.com/the-github-graphql-api/)) and has a powerful debug tool [GraphiQL](https://github.com/graphql/graphiql). Most importantly GraphQL is an architectural choice.

## What Is Architecture?

There are so many different ways to define Architecture, so I'll borrow one of them from [Who Needs An Architect](http://files.catwell.info/misc/mirror/2003-martin-fowler-who-needs-an-architect.pdf):

> Architecture is the set of design decisions that must be made early in a project

When you take a moment to think about what types of choices you make in the early stages in a project, these are usually things like

- Programming Language (Python, Ruby, GO, JS, Etc.)
- Framework/Library (Django, Rails, Express, JQuery, React, Etc.)

These are decisions that would be difficult, but not necessarily impossible to reverse. Each of them have an _irreversibly_ cost that must be weighed in vs the problem that they solve.

With this in mind architecture could also be defined as **a set of difficult to reverse design decisions**.

## Irreversibility Drives Complexity

When something is difficult to reverse, it tends to be an anchor point that others will build off of as set of constraints. Decisions are made (for better or worse) that limit the number of choices available. An example of this is ordered function arguments vs keyed arguments:

### Complex

```js
const add = (a, b) => a + b
```

This adds hidden complexity because the order of the arguments matters. The arguments are effectively a list. Adding another argument requires appending it or modifying all of the function callers.

### Simple

```js
const add = ({ a, b }) => a + b
```

This is simpler because argument order doesn't matter. Adding another argument does not necessarily require changing argument order for other function callers.

In the *complex* example with ordered arguments, it is more complex because it is more difficult to change.

## Why Is GraphQL Architecture?

Choosing to use GraphQL is choosing to utilize the Query and Mutation domain specific language (DSL):

```GraphQL
{
  Movie(id: "cixos5gtq0ogi0126tvekxo27") {
    id
    title
    actors {
      name
    }
  }
}
```

Because GraphQL is a DSL it has many of the same trade-offs as other DSLs:

- Incompatible with other systems
- Learning cost
- Someone must maintain it
- Increased productivity within the domain

Once GraphQL is used, it requires a significant amount of effort to reverse the decision. If **a set of difficult to reverse design decisions** is architecture, then GraphQL is an architectural decision.

Since difficult to reverse decisions drive complexity in systems, GraphQL adds complexity. Understanding the complexity that GraphQL adds to project is going to a critical part of weighing the trade-offs.
