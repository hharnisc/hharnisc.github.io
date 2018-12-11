---
layout: slides
title: Getting The Most Out Of Kubernetes
slides: getting-the-most-out-of-kubernetes.md
---

So you've carefully crafted your first Kubernetes service, and you're ready to deploy it to production. Well, not quite: there are still some important unknowns to understand before your service will be ready for production traffic. It's still unclear how the new service behaves when it's being pushed, and it's possible that Kubernetes will kill the service before serving a single request. While at Buffer, we developed a technique to optimize Kubernetes deployment limits by using load testing to identify optimal values for resource limits. When the service is under heavy load there are a few key metrics to watch to identify bottlenecks. These key metrics can be used to adjust resource limits. This real world approach allowed us to safely and efficiently switch our production traffic to our Kubernetes cluster and can be applied to any application.
