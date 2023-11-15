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
Now that it is clear what to avoid, lets take a look at design patterns that are 
known to be successful.

### Composable Monitoring
Composable monitoring is when 
[UNIX philosophy](https://en.wikipedia.org/wiki/Unix_philosophy) 
is applied to monitoring - loosely couple multiple tools together according to 
your needs, and form a monitoring platform. 

In other words, there should be a dedicated tool for each part of a monitoring system. 
Here is a breakdown of all parts:
- Data collection: 2 primary methods - pull and push metrics.
- Data storage: metrics are usually stored in a TSDB, where they might be summarized (reduced resolution) to save space. Logs are stored in a flat file (rsyslog) or in a search engine (Elasticsearch). It is also common to storage data in remote storage, like S3.
- Visualization: Good dashboards should be answering questions you have at given moment. They can also vary greatly in functionality, some are giving high level overview, and others are providing details about a single service.
- Analytics and reporting: Mostly for determining and reporting on SLAs.
- Alerting: see [Tips on Alerting](#tips-on-alerting)

### Think About the User
Users don't care about the server load or available storage on servers, they care 
if the service is available and works. So the first order of business when configuring 
monitoring is to make sure that HTTP status codes, request latency and time are being 
monitored very well.  

### Continual Improvement
Monitoring system is a complex project that quickly evolves and requires constant 
maintanence. Be patient, always look for improvements, get feedback from users and 
optimize it.  


## Tips on Alerting
Alert is supposed to cause an immediate reaction from the on-call engineer. 
If immediate reaction is not required, then it's a message. Message might cause an 
alert in future if it is not handled in-time, so it still has be sent out to internal 
chats or filed as an auto-generated ticket (or even resolved automatically).  

### What Makes a Good Alert
Here are some qualities that make up a good alert:
- Deliver it with appropriate tools: Pager if it's an alert, chatroom if it's a message.
- Write runbooks: Link to documentation about a particular issue which can be embedded into the alert describtion.
- Be flexible in alert triggers: Arbitrary static thresholds are not always a best option for triggering alert, in some cases percentage is better (like in disk usage).
- Delete and tune alerts: If alert is not causing an immediate action - tune or delete it.
- Attempt self-healing: Some tasks can be automated. In this case alert should be sent only if automation failed to fix the issue.

### Runbooks
Runbooks helps on-call engineer to quickly understand the context of an alert, and 
improve the resolution speed. It is usually not a list of commands for copy-pasting, 
becuase in that case it could just be automated via self-healing, but rather is a 
context and extra information for the alert to assist in forming a human judgement.  

A runbook should be able to answer following questions about a service:
- What is the servie and what is its functionality?
- Who/What team is responsible for it?
- Does it have dependencies?
- What infrastructure it uses?
- Where are its logs?
- What alerts are set up for it and why?

Information about the runbook can be provided via a link to documentation service 
that is attached to the alert description. This way, on-call engineer can just 
follow the link and save a lot of time debuggin the issue.  


## Conclusion
This article described some general concepts that make a up a good monitoring system 
without dwelving into particular tools. Some anti-patterns were covered, as well as 
good practices. To summarize:
- Composable monitoring architecture is more efficient
- Monitor things by thinking from user's perspective
- Keep improving and gather feedback
- There is no silver bullet when it comes to tools
- Monitoring is everyones responsibility
- Think carefully if something actually needs to monitored
- Monitoring does not fix bad code
- Automate everying to improve usability and ensure consistenct

We also covered some basics in alerting, since it is an integral part of any 
monitoring system. In a nutshell, every alert has to be carefully though out 
to reduce noize, and should provide essential information via runbooks for 
the on-call engineers to have a context about the issue.  
