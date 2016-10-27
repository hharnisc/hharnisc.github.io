class: center, middle, inverse-slide
## Load Testing and Optimizing .kubernetes-text[K8S] Deployment Memory/CPU Limits
### _So you built a service, now what?_
<img src="/images/slides/load-testing-and-kubernetes/kubernetes.svg" width="100px"/>

.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]
---
## This Talk *Might* Be Enhanced If You're Familiar These
- `kubectl` command line tool
- Kubernetes [Deployments](http://kubernetes.io/docs/user-guide/deployments/)
- Kubernetes [Services](http://kubernetes.io/docs/user-guide/services/)

*But it's certainly not required*
---
class: middle, inverse-slide
## You’ve shipped your first deployment to .kubernetes-text[Kubernetes] 🎉🎉🎉
### ... _and you’re done right?_
---
class: middle, inverse-slide
## Well, .purple-text[not exactly] 😬
---
class: middle, inverse-slide
## There are still some things we .blue-text[don’t know]
---
.left-column[
## What We Don’t Know
]
.right-column-middle[
- What’s the failure point of the system?
    - Quantitative
      - Requests per second
]
---
.left-column[
## What We Don’t Know
]
.right-column-middle[
- What does a failure look like?
    - Qualitative
      - 500s
      - Segmentation fault
]
---
.left-column[
## What We Don’t Know
]
.right-column-middle[
- What does it look like when the system approaches failure?
]
---
.left-column[
## What We Don’t Know
]
.right-column-middle[
- How do you recover from a failure?
]
---
class: middle, inverse-slide
## That’s a pretty .red-text[risky] list of unknowns
- What’s the failure point of the system?
- What does a failure look like?
- What does it look like when the system approaches failure?
- How do you recover from a failure?
---
class: middle, inverse-slide
## We can reduce risk by <br> .green-text[increasing predictability]
---
class: middle
## Predictability

> Given a load, the system achieves the **same results** from test to test

---
class: middle, inverse-slide
## How can we increase predictability with .kubernetes-text[Kubernetes]?
---
class: middle, inverse-slide
## We can start by setting .blue-text[resource limits]
---
.left-column[
## Setting Limits
]
.right-column-middle[
- Kubernetes can run the maximum number of simultaneous tasks
]
---
.left-column[
## Setting Limits
]
.right-column-middle[
- Each container has enough resources to complete a task
]
---
.left-column[
## Setting Limits
]
.right-column-middle[
- Kubernetes can manage resources efficiently
]
---
class: middle, inverse-slide
## .green-text[over]/.blue-text[under]/.purple-text[even] resource allocation
---
.left-column[
## Setting Limits
### Underallocation
]
.right-column-middle[
- Containers are exceeding resource limits
- Kubernetes constantly kills containers
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/under-allocation.png" width="60%"/>
]
---
.left-column[
## Setting Limits
### Underallocation
### Overallocation
]
.right-column-middle[
- Containers can never fully utilize resources
- Kubernetes lets containers run, but each container is allocated extra resources
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/over-allocation.png" width="60%"/>
]
---
class: middle, inverse-slide
## Overallocation is a .purple-text[tricky] problem to detect
---
class: middle, inverse-slide
## But it becomes a problem when you _scale up your replicas_
---
class: center, middle, inverse-slide
<img src="/images/slides/load-testing-and-kubernetes/over-allocation.png" width="40%"/>
## vs
<img src="/images/slides/load-testing-and-kubernetes/just-right.png" width="40%"/>
---
class: middle, inverse-slide
## That's an .green-text[extra container] you could be running
---
.left-column[
## Setting Limits
### Underallocation
### Overallocation
### Even
]
.right-column-middle[
- Containers have enough resources to complete the task
- Kubernetes resources are allocated efficiently
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/even-steven.jpg" width="60%"/>
]
---
class: middle, inverse-slide
## How much a container needs depends on the type of load <br> .purple-text[Bursty] vs. .blue-text[Constant]
---
.left-column[
## .purple-text[Bursty] vs. .blue-text[Constant]
]
.right-column-middle[
- If migrating from another service
    - Take a look at the traffic data (if you've got it)
      - Periodic Peaks
      - Random Peaks
      - Constant
]
---
.left-column[
## .purple-text[Bursty] vs. .blue-text[Constant]
]
.right-column-middle[
- Constant
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/constant.png" width="80%"/>
]
---
.left-column[
## .purple-text[Bursty] vs. .blue-text[Constant]
]
.right-column-middle[
- Bursty
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/bursty.png" width="80%"/>
]
---
class: middle, inverse-slide
## Lets set some .purple-text[limits]!
---
.left-column[
## Setting Limits
### Testing Setup
]
.right-column-middle[
- Goal: figure out what **one container** can handle
  - Set the number replicas to 1
  - Start with a conservative level of resources
  ```yaml
  # for node might be something like
  cpu: 100m
  memory: 100Mi
  ```
]
---
class: middle, inverse-slide
## Now lets find the .red-text[breaking point] 👹
---
.left-column[
## Setting Limits
### Find The Breaking Point
]
.right-column-middle[
- Run a test where we slowly increase the load over time until we hit the breaking point
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/breakingpoint.png" width="80%"/>
]
---
.left-column[
## Setting Limits
### Find The Breaking Point
]
.right-column-middle[
- If container performed poorly, identify the bottleneck
  - While running a test you can `exec` into a container
      - Check memory consumption with something like
      ```bash
      ps -eo rss,pid,euser,args:100 --sort %mem
      ```
      - Check cpu and system load with
      ```bash
      top
      ```
  - Once you identify a bottleneck try raising the limit and repeat
  - Worst case you have to optimize some code paths
]
---
.left-column[
## Setting Limits
### Find The Breaking Point
]
.right-column-middle[
- When the test is too easy or the resources are overallocated, you’ll likely see no change in the system performance over time
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/overallocated.png" width="80%"/>
]
---
.left-column[
## Setting Limits
### Find The Breaking Point
]
.right-column-middle[
- When resource limits are at a good level you’ll likely see the performance degrade slowly over time
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/goodtest.png" width="80%"/>
]
---
.left-column[
## Setting Limits
### Find The Breaking Point
]
.right-column-middle[
- When a test fails observe and take note
    - Did it recover on it’s own? (probably did with Kubernetes 😎)
    - How did it behave right before it broke
        - Useful for monitoring
]
---
class: middle, inverse-slide
## Does It Handle Load .blue-text[Over Time]?
---
.left-column[
## Setting Limits
### Consistent Load Over Time
]
.right-column-middle[
- Set the load to something just under the breaking point
- The test duration depends on your use case
    - At least 10 minutes as a general guide
    - Longer tests are better!
]
---
.left-column[
## Setting Limits
### Consistent Load Over Time
]
.right-column-middle[
- Problem signs (not an exhaustive list)
  - Memory is slowly increasing
  - CPU is pegged at 100%
  - 500s in response codes
  - High response times
  - Large variance in response times
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/bad-duration-load-test.png" width="80%"/>
]
---
.left-column[
## Setting Limits
### Consistent Load Over Time
]
.right-column-middle[
- When things are looking good you *should* see consistent performance over time
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/good-duration-load-test.png" width="80%"/>
]
---
class: middle, inverse-slide
## At this point we should have an .green-text[answer] to our previous questions
- What’s the failure point of the system?
- What does a failure look like?
- What does it look like when the system approaches failure?
- How do you recover from a failure?
---
class: middle, blue-slide
# What Now?
---
.left-column[
## What Now?
]
.right-column-middle[
- **Setup monitoring**
- Slow rollout (from existing system)
  - 1% -> 10% -> 50% -> 100%
- Build an autoscaling service 😎
]
---
.left-column[
## What Now?
### Autoscaling <br> Service 😎
]
.right-column-middle[
- Monitor all containers in an app
- Scale up replicas when load exceeds a certain threshold
- Could even automate the load testing process and *auto set* thresholds
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/metrics.png" width="100%"/>
]
---
class: middle
## Just A Thought...
<img src="http://imgs.xkcd.com/comics/barrel_part_5.jpg" width="50%" />
---
class: middle
# Questions?
.footnote[Made with [remark](https://github.com/gnab/remark)]
