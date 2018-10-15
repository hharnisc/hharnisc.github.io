class: center, middle
# Load Testing and Optimizing .kubernetes[K8S] Deployment Memory/CPU Limits
## _So you built a service, now what?_
<img src="/images/slides/load-testing-and-kubernetes/kubernetes.svg" width="100px"/>

.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]
---
# This Talk *Might* Be Enhanced If You're Familiar These
- `kubectl` command line tool
- Kubernetes [Deployments](http://kubernetes.io/docs/user-guide/deployments/)
- Kubernetes [Services](http://kubernetes.io/docs/user-guide/services/)

*But it's certainly not required*
---
class: middle
# You’ve shipped your first deployment to .kubernetes[Kubernetes] 🎉🎉🎉
## ... _and you’re done right?_
---
class: middle
# Well, .purple[not exactly] 😬
---
class: middle
# There are still some things we .blue[don’t know]
---
class: fifty-fifty
.left-panel[
# What We Don’t Know
]
.right-panel[
- What’s the failure point of the system?
    - Quantitative
      - Requests per second
]
---
class: fifty-fifty
.left-panel[
# What We Don’t Know
]
.right-panel[
- What does a failure look like?
    - Qualitative
      - 500s
      - Segmentation fault
]
---
class: fifty-fifty
.left-panel[
# What We Don’t Know
]
.right-panel[
- What does it look like when the system approaches failure?
]
---
class: fifty-fifty
.left-panel[
# What We Don’t Know
]
.right-panel[
- How do you recover from a failure?
]
---
class: middle
# That’s a pretty .red[risky] list of unknowns
- What’s the failure point of the system?
- What does a failure look like?
- What does it look like when the system approaches failure?
- How do you recover from a failure?
---
class: middle
# We can reduce risk by <br> .green[increasing predictability]
---
class: middle
# Predictability

> Given a load, the system achieves the **same results** from test to test

---
class: middle
# How can we increase predictability with .kubernetes[Kubernetes]?
---
class: middle
# We can start by setting .blue[resource limits]
---
class: fifty-fifty
.left-panel[
# Setting Limits
]
.right-panel[
- Kubernetes can run the maximum number of simultaneous tasks
]
---
class: fifty-fifty
.left-panel[
# Setting Limits
]
.right-panel[
- Each container has enough resources to complete a task
]
---
class: fifty-fifty
.left-panel[
# Setting Limits
]
.right-panel[
- Kubernetes can manage resources efficiently
]
---
class: middle
# .green[over]/.blue[under]/.purple[even] resource allocation
---
class: fifty-fifty
.left-panel[
# Setting Limits: Underallocation
]
.right-panel[
- Containers are exceeding resource limits
- Kubernetes constantly kills containers
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/under-allocation.png" width="60%"/>
]
---
class: fifty-fifty
.left-panel[
# Setting Limits: Overallocation
]
.right-panel[
- Containers can never fully utilize resources
- Kubernetes lets containers run, but each container is allocated extra resources
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/over-allocation.png" width="60%"/>
]
---
class: middle
# Overallocation is a .purple[tricky] problem to detect
---
class: middle
# But it becomes a problem when you _scale up your replicas_
---
class: center, middle
<img src="/images/slides/load-testing-and-kubernetes/over-allocation.png" width="40%"/>
# vs
<img src="/images/slides/load-testing-and-kubernetes/just-right.png" width="40%"/>
---
class: middle
# That's an .green[extra container] you could be running
---
class: fifty-fifty
.left-panel[
# Setting Limits: Even
]
.right-panel[
- Containers have enough resources to complete the task
- Kubernetes resources are allocated efficiently
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/even-steven.jpg" width="60%"/>
]
---
class: middle
# How much a container needs depends on the type of load <br> .purple[Bursty] vs. .blue[Constant]
---
class: fifty-fifty
.left-panel[
# .purple[Bursty] vs. .blue[Constant]
]
.right-panel[
- If migrating from another service
    - Take a look at the traffic data (if you've got it)
      - Periodic Peaks
      - Random Peaks
      - Constant
]
---
class: fifty-fifty
.left-panel[
# .purple[Bursty] vs. .blue[Constant]
]
.right-panel[
- Constant
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/constant.png" width="80%"/>
]
---
class: fifty-fifty
.left-panel[
# .purple[Bursty] vs. .blue[Constant]
]
.right-panel[
- Bursty
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/bursty.png" width="80%"/>
]
---
class: middle
# Lets set some .purple[limits]!
---
class: fifty-fifty
.left-panel[
# Setting Limits: Testing Setup
]
.right-panel[
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
class: middle
# Now lets find the .red[breaking point] 👹
---
class: fifty-fifty
.left-panel[
# Setting Limits: Find The Breaking Point
]
.right-panel[
- Run a test where we slowly increase the load over time until we hit the breaking point
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/breakingpoint.png" width="80%"/>
]
---
class: fifty-fifty
.left-panel[
# Setting Limits: Find The Breaking Point
]
.right-panel[
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
class: fifty-fifty
.left-panel[
# Setting Limits: Find The Breaking Point
]
.right-panel[
- When the test is too easy or the resources are overallocated, you’ll likely see no change in the system performance over time
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/overallocated.png" width="80%"/>
]
---
class: fifty-fifty
.left-panel[
# Setting Limits: Find The Breaking Point
]
.right-panel[
- When resource limits are at a good level you’ll likely see the performance degrade slowly over time
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/goodtest.png" width="80%"/>
]
---
class: fifty-fifty
.left-panel[
# Setting Limits: Find The Breaking Point
]
.right-panel[
- When a test fails observe and take note
    - Did it recover on it’s own? (probably did with Kubernetes 😎)
    - How did it behave right before it broke
        - Useful for monitoring
]
---
class: middle
# Does It Handle Load .blue[Over Time]?
---
class: fifty-fifty
.left-panel[
# Setting Limits: Consistent Load Over Time
]
.right-panel[
- Set the load to something just under the breaking point
- The test duration depends on your use case
    - At least 10 minutes as a general guide
    - Longer tests are better!
]
---
class: fifty-fifty
.left-panel[
# Setting Limits: Consistent Load Over Time
]
.right-panel[
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
class: fifty-fifty
.left-panel[
# Setting Limits: Consistent Load Over Time
]
.right-panel[
- When things are looking good you *should* see consistent performance over time
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/good-duration-load-test.png" width="80%"/>
]
---
class: middle
# At this point we should have an .green[answer] to our previous questions
- What’s the failure point of the system?
- What does a failure look like?
- What does it look like when the system approaches failure?
- How do you recover from a failure?
---
class: middle
# What Now?
---
class: fifty-fifty
.left-panel[
# What Now?
]
.right-panel[
- **Setup monitoring**
- Slow rollout (from existing system)
  - 1% -> 10% -> 50% -> 100%
- Build an autoscaling service 😎
]
---
class: fifty-fifty
.left-panel[
# What Now?: Autoscaling <br> Service 😎
]
.right-panel[
- Monitor all containers in an app
- Scale up replicas when load exceeds a certain threshold
- Could even automate the load testing process and *auto set* thresholds
<br><br>
<img src="/images/slides/load-testing-and-kubernetes/metrics.png" width="100%"/>
]
---
class: middle
# Just A Thought...
<img src="http://imgs.xkcd.com/comics/barrel_part_5.jpg" width="40%" />
---
class: middle
# Questions?
.footnote[Made with [remark](https://github.com/gnab/remark)]
