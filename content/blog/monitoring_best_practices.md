---
title: "Monitoring_best_practices"
date: 2023-11-14T23:03:30+03:00
draft: true
tags:
    - monitoring
categories:
    - DevOps
ShowToc: true
---

## Introduction
This article is a summary of monitoring best practices as outlined in the book 
["Practical Monitoring"](https://www.oreilly.com/library/view/practical-monitoring/9781491957349/) 
by Mike Julian. I did not add  my original thoughts on monitoring, it's just 
a simple summary of core concepts from first 3 chapters. The whole book is 
tool-agnostic and focuses on common themes that occur in different kinds of 
monitoring systems, so the article will not be mentioning any specific tools.  

## Monitoring Anti-Patterns
Before jumping to design patterns it's important to understand what should be 
avoided.

### Tool Obsession
There is no perfect tool that will magically solve all issues with monitoring, 
and nor it should. Modern monitoring platforms offer a large number of loosely 
coupled tools which are supposed to work greatly together. Experiment with 
different tools and pick the ones that fit best to your infrastructure.  

Ideally, service owners should have a say in choosing the tools (if they don't 
have a direct way to modify monitoring system) because they know better how to 
monitor their services. Talk to people, ask about their preferences and needs 
and implement it.  

Don't incorporate tools just because other successful teams use it. Tools 
essentially are a result of a particular work style, which might not be 
suited for your current situation. That is also why sometimes it is neccessary 
to create custom tools, but only when it's a small specialzed tool, 
and not a whole monitoring platform from scratch.  

### Checkbox Monitoring
```
Checkbox monitoring is when you have monitoring systems for the sole sake
of saying you have them.
```
Checkbox monitoring is usually caused by misunderstanding what should be monitored. 
This is understandable, because responsibility on monitoring lays on every person 
in the organization. Simply put, one person could not possibly know how to monitor 
everything - networking gear, services, databases etc.  

It's OK to have a team be responsible for the monitoring platform generally, 
but creating alerts and using custom tools should be everyones responsibility.  

Here are some things that can be done to better understand what should be monitored:
- Understand what "working" actually means: Establishing high-level checks, such as an HTTP GET / for a webapp, provides valuable insights into its functionality, including response codes, expected content, and request latency. While these checks may not pinpoint specific problems, they serve as effective early indicators.
- OS metrics are not very useful for alerting. Alerts like 80% of CPU load are not actually a problem if the service is continuing to function properly.
- Collect metrics more often: some metrics can be collected once in 30-60 seconds (most modern networking devices can easily handle this kind of load).

### Misc
It is important to remember that monitoring is not a solution to a problem, but 
a way to detect problem when they happen. Is a service is poorly written and breaks 
often there is no point in adding additional alerts to it, the only solution is 
to fix the service itself.  

Finally, monitoring should be 100% automated. That means services and alerts should 
be self registered, as opposed to someone manually adding them. If this process is 
made as easy as possibly, it will be more enganging and will encourage people to add 
more checks to their services.  

## Monitoring Desing Patterns

## Tips on Alerting
