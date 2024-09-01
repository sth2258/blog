---
title: "Adventures with Keda"
layout: post
date: 2024-09-01
---

If we have spoken about all things technical, you'd know how much of a lover of Keda I am. For me, this simple piece of middleware is the one brings developers down to the lowest levels of infrastructure, allowing them control for almost everything. 

### For Job Scheduling

In my day job, we have very heavy users of job scheduling - platforms like Informatica PowerCenter, Microsoft Flow, Azure Fabric / Azure Data Factory and others allow for the low/no-code solutions that are good, but come with **lots** of challenges. 

- Dedicated infra to handle peak loads (especially for Informatica)
- Specialized roles for ETL development
- Environment specialization (restricted to what the platform offers)
- Costly licensing / overhead

For job scheduling -- lets imagine you want to execute something like a database backup. You do already have an enterprise scheduler (MControl, AutoSys, etc.) with robust scheduling and calendaring features, but don't want to rely fully on this for handling the execution. 

### Event-Driven Data Processing

Using middleware such as message queues to exchange data between services is very common -- but what are the downstream implications of that kind of design? It means I then need to have something listening to that queue all the time, and that stinks! Instead, let Keda listen to the queue -- if there are items to parse, scale a pod and let that take control.

### Modern CI/CD

I want to scale my CI/CD, leverage the elasticity of my Kubernetes cluster, and not have a bunch of scale-set images. This is an absolutely killer example of when Keda can shine. CI/CD pipelines have lots of spikes in activity during development and release cycles. Platforms such as GitHub can be directly integrated with (such as query latest commit on a repo), and scale appropriately. 

### Simple Autoscaler

The [Kubernetes cluster-autoscaler](https://github.com/kubernetes/autoscaler) can work great for most applications based on limited attributes, but what about those where it doesn't? If I wanted to scaled based on specific metrics from inside my application, how can I do that? What about based on external APM? 

### Triggering

For workloads that are less critical or ones that distinct periods of low utilization, a time-based trigger might be appropriate. But what about those workloads that are not predictable? Or even when we want to ensure completion within a particular window? Could Keda do this for us? 

Triggers for Keda can also include dynamic scaling of nodes used to complete a backup based upon the size of database, current load, etc. to ensure that the RPO is kept within the acceptable window.

### Execution

This is where Keda really shines. Since industry wide, developers are more than accustomed to containerizing their applications, they have direct control of exactly what is happening within this context of this job. If their environment is a simple Python script with vew dependencies, how very easy it would be for them to build up a `dockerfile` with exactly what they need to perform the case at hand. 

Gone are the days where you needed a specific role to perform this development work - The batch requirements can pivot based on the existing skillset within the team - If you're a Python shop, do that; if you're a Java/Spring team, ez. As long as your triggers are setup appropriately and you've handled all of your environmental considerations (thread safety, file locking, etc.), you're good! Just remember to write your tasks in such a way that allows them to be distributed across multiple nodes; less your one pod is handling it all. 

### Scale Down

After our job is done and nothing else is in queue, why keep those resources around? Scale them down. Keda scales pods back down to zero.....or doesn't if you don't want it to! If you want to have a listener queue and with a rather large container image performing the execution, set a `minReplica` -- and allow those nodes to keep running and pickup items as needed; your `listLength` attribute can then be used to trigger your scale-up (i.e., if the queue is too big, scale it).


The bottom line is - Keda is a pretty amazing solution to almost all of the challenges we've had with legacy infrastructure and middleware solutions, and brings developers closer to the problems they are trying to solve. In future posts I will go into some details to some of the scripts I've been using to help manage it. 