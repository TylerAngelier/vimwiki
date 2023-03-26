# Interview

Questions for when giving an interview.

[Great](Great.md) set of questions: https://github.com/bregman-arie/devops-exercises

# Culture Questions

## Tell me a little about yourself.

Anything.

## What has you interested in this position at Oracle?

Anything.

## How would you describe a successful DevOps engineer or team?

The answer can focus on:
* Collaboration
* Communication
* Set up and improve workflows and processes (related to testing, delivery, ...)
* Dealing with issues

Things to think about:
* What DevOps teams or engineers should NOT focus on or do?
* Do DevOps teams or engineers have to be innovative or practice innovation as part of their role?

## What do you take into consideration when choosing a tool/technology?

A few ideas to think about:
  * mature/stable vs. cutting edge
  * community size
  * architecture aspects - agent vs. agentless, master vs. masterless, etc.
  * learning curve

## What is YAGNI? What is your opinion on it?

YAGNI - You aren't gonna need it. 
You must add functionality that will be used. No need to add functionality that is not directly needed.

## Describe an ideal day at work?

Looking for something that aligns with our culture. 
Things to think about:
- No laziness
- Are they self motivated?
- What kind of ownership do they have?
	- Do they start to work on things they know need to be done or pick from a backlog?

## What area of work in the DevOps space do you like the most?

Examples:
- Logging
- Monitoring
- Provisioning
- Process
- Load testing

## What technology are you interested in right now or were previously interested in?

Looking for how well the canidiate gets to know things they are interested in.
Looking for:
- Excitement
- Depth of knowledge
- _Having an answer_ regardless of what it is

# Technical Questions (DevOps focused)

## Explain the differences between Continuous Deployment and Continious Delivery?

Continuous Deployment:
> The deployment to the production environment is fully automated and does not require manual/ human intervention.
> Here, the application is run by following the automated set of instructions, and no approvals are needed.

Continuous Delivery
>In this process, some amount of manual intervention with the manager’s approval is needed for deployment to a production environment.
>Here, the working of the application depends on the decision of the team.

## What is a Blue/Green Deployment?

>A blue-green pattern is a type of continuous deployment, application release pattern which focuses on gradually transferring the user traffic from a previously working version of the software or service to an almost identical new release - both versions running on production.
>The blue environment would indicate the old version of the application whereas the green environment would be the new version.
>The production traffic would be moved gradually from blue to green environment and once it is fully transferred, the blue environment is kept on hold just in case of rollback necessity.
>In this pattern, the team has to ensure two identical prod environments but only one of them would be LIVE at a given point of time. Since the blue environment is more steady, the LIVE one is usually the blue environment.

## Kubernetes

### In your own words, explain what a Pod is?
TLDR: A Pod is similar to a set of containers with shared namespaces and shared filesystem volumes.

_Pods_ are the smallest deployable units of computing that you can create and manage in Kubernetes.

A _Pod_ (as in a pod of whales or pea pod) is a group of one or more [containers](https://kubernetes.io/docs/concepts/containers/), with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. A Pod models an application-specific "logical host": it contains one or more application containers which are relatively tightly coupled. In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.

### What is the difference between a Deployment and a Daemonset?

DaemonSet manages the number of pod copies to run in a node. However, a deployment manages the number of pods and where they should be on nodes. Deployment selects nodes to place replicas using labels and other functions (e.g., tolerations).

A DaemonSet doesn’t need external resources like IP addresses or port numbers. However, you must provide them if you create a deployment.

Additionally, a DaemonSet doesn’t need to know the number of nodes in your [Kubernetes cluster.](https://loft.sh/blog/multi-cluster-kubernetes-part-one-defining-goals-and-responsibilities/?utm_medium=reader&utm_source=other&utm_campaign=blog_kubernetes-daemonset-what-it-is-and-5-ways-to-use-it) However, you must provide it if you create a deployment.

Last, a DaemonSet doesn’t need runtime state information, like the number of pods currently running on each node or the number of replicas running on each pod (although these things can still be specified).

### Write a `kubectl` command to get all pods with a label `foo=bar`?

```bash
kubectl get pods -l foo=bar -A
```

### When would you use a secret over a configmap?

# Austin Interview Document

Hackerrank: https://hr.gs/2ca0b3c

## Introductions

Avalara, tax compliance, tax calculations, in seattle, odd guy out. Another guy was remote, acquired by avalara.

## Generic

### Most fun

> What was your principal contribution?

Not a developer, work in infra, have worked in dev. SQL would...take data, read it from XML, use it to parse messages. In that company, jms, xml, parse, fix data. Grabbing data, xml, not straight forward in SQL. THen put the data back after filtered. Just a tool to clear out messages...

Code wise...when deployed bug fixes, basic stuff, statements. 2 to 2.5m lines of code, couldn't figure out where bad data came from. Teams gone, people replaced gone. Only way to fix, don't care...if it doesn't fit the format, don't write it.

> Recent thing

Avalara, tickets...most of this, maintaining infra. 100s of AWS accounts, teams to maintain. Have standups, mo, thu, go over tickets.

Big deploy a few months ago...on my end, it was access here, access there.

* Complains about merge requests

### Most complicated / challenging

> What was your principal contribution?

JC, part of dev team, team that fixed bugs. We fix bugs - job nobody wants, I thought it was cool. Take backsup, rebuild, rerun, reproduce. Sometimes...the upstream...problem is challenging to find. Spend months...dig and dig... People would say...just move on, don't spend too much time on it. Just have to figure out...

JSM app, messages would go in queues for processing. Used JBOSS, would go to journal

## Technical Competencies

## Core

> What is DNS? How does it work?

> Some issues?

> Issues with caching?

> Why implemented the way it is? Why not updated all the time?

> CAP theorem?

> Describe what a CDN is and how it works

> How is memory shared?

> Processes?

- Diff heap/stack

- What are the pros / cons for method/block synchronization?

> What do you have to worry about?

- What is deadlock?

- What is a race condition?

- How is a linked list different than an array?

> IaC

> Terraform, CFM

> How cross cloud is tf?

> Different resources?

## Programming

- Difference between a stack and a queue?

- When might you use a stack?

> Breadth first, depth first?

## Language Mechanics

- What is inheritance?

> Why can't you extend multiple classes?

- Difference between a class and an interface?

> When might you use one versus the other?

> What is polymorphism?

- What is composition?

> When would you use composition over inheritance?

## Searching / Sorting

- What is the difference between a stable and unstable sorting algorithm?

# Empathetic Competencies

## Ownership

Describe a time you...

- Made a commitment to something, like getting a feature or sprint work done, and realized part way through you wouldn't be able to.

> What was it you had committed too and how did you handle the situation?

> Communicate out the timeline? What was going on?

> Hard deadline, trying again...

## Bias for Action

Describe a time you...

- Went outside of your job role to fix something, build something, or improve something?

What was it you did and why did you do it?

## Deliver Results

Describe a time you...

## Earn Trust

Describe a time you...

# Whiteboard

## Common Phrases

Complete the function, below, that returns the most common word that appears between two provided arrays:

```java

public class WhiteboardQuestions {

    /**

     * Returns the most common word found in the provided list.

     * to, be, or, not, to, be, that, is, the, question, to, to, to

     * => to

     */

    public static String mostCommonWord(List<String> listOne) {

    }

}

```

# Being a watcher of an Interview

- Talked about himself first. 
- Asked interviewee to talk about themselves. 
- Most fun bit of code you ever have written.
- Asks a lot of follow-up questions based on the interviewe response
- Switches to "General Technical questions"
	- What is DNS?
		- Why don't we try to have one centralized DNS? (As a follow-up b/c the interviewer was talking a lot obout decentralized DNS.)
		- Does DNS have strong consistency or eventually consistent?
	- CAP Theorem 
	- Can you describe a CDN?
- Switches to development related technical questions
	- differences between process and a thread?
		- Can you share memory between processes
		- Difference between the HEAP and the stack?
			- Are you familar with stackoverflow exception? (trying to guide the interviewer to the right answer b/c he was close)
	- What is deadlock?
	- Tell me about Infrastructure as Code?
	- CloudFormation vs Terraform
	- How cross-cloud is Terraform?
		- Are they the same across clouds?
	- Weird debug problem: App code has been deployed in AWS, load balancer with autoscalling group, three instances running. Randomly CPU will spike - only way to fix it is to restart the Instance of the service. Usually quick build-up when it happens.
	- Difference between a Stack and Queue?
	- What is the difference between a stable and unstable unsorting algorithm?
- Tell me about a time when something happened
	- You made a commitment to something (like a ticket) - you get into it and there is no way that I can finish it on time - how did you deal with it?
		- Trying to get the interviewee to talk about how they communicate.
	- Tell me about a time when you went outside your job responsiblity (not becuase you were told to do but because you wanted to do it)?
- Coding session
	- Looking for performance + data structures insight
- Are you familar with Big O notation

* Says he will compile all the notes he wrote and send it up and you'll be contacted in the next few days.
