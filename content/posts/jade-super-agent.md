---
date: "2020-03-04"
title: "A Simple Parent Class for JADE Agents"
tags: ["java"]
author: "masoudkf"
excerpt: "The JADE Agent class&mdash;which your agents must extend in order to be recognized by the system as an actual agent&mdash;offers the minimum functionality. Although it's totally fine, there's no harm in adding a couple more features to it. The ones I think many agents would like to have right from the start. Here's a simple suggestion."
---

In this article, I will not talk about how JADE works. I assume that you are already familiar with the framework. If this is not the case, you can check out JADE <a target="_blank" href="https://jade.tilab.com/" rel="noopener noreferrer">website</a> first. Let's get to it.

{{< gist spf13 7896402 >}}

Communication is the heart of every agent-based system. If your agents do not talk to one another, what's the point of having them? In order to establish connections between your agents (a.k.a sending messages), you need to have the destination address. In JADE, this address is something called `AID` or Agent ID. The question to ask now is "How do we find the AID of our target agent?" I'm so glad you asked.

In the JADE world, the best practice for finding other agents is by using something called **Yellow Pages**. Yellow Pages in JADE, like their real-world equivalent, is a way to advertise your service and find the services offered by others that you're interested in using. So, as I'm sure you'd agree, this is a very essential thing for an agent: the ability to publish its services, and find the services it needs, or more accurately, find the agents that offer a particular service. The `Agent` class, however, does not provide these simple features. In this tutorial, we make a wrapper class around the `Agent` class to add these functionalities. Our agents, then, can extend this class instead and use the corresponding methods right from the beginning. Let's first have a look at the methods&mdash;we will wrap them together at the end.

In JADE, the yellow page service is handled by the `DF` agent (or Directory Facility). You do not need to create this agent manually. It's already running on the background and is being managed by the framework itself. Let's see our first method, `register`:

```java
protected void register(String serviceName) {
		DFAgentDescription dfd = new DFAgentDescription();
		dfd.setName(getAID());
		ServiceDescription sd = new ServiceDescription();
		sd.setName(getLocalName());
		sd.setType(serviceName.toLowerCase());
		dfd.addServices(sd);
		try {
			DFService.register(this, dfd);
		}
		catch (FIPAException ex) { ex.printStackTrace(); }
	}
```
<br/>

The method is very straightforward. It gets a `ServiceName` as a `String` and publishes it using the `DF` agent. Note that for doing that, we need a `DFAgentDescription` and a `ServiceDescription`. In the end, since the process might throw an exception, we wrapped it in a `try-catch` block. As you can see, `DFAgentDescription` works with `AID`s, whereas `ServiceDescription` works with the `ServiceName` we provide as a `String`. The `toLowerCase` method is just for making sure that we will not get into one of those situations where we have a service named `Buyer` and we are looking for some `buyers` to no avail. It's just a safety mechanism&mdash;and a rather simple one.

Now that we can register ourselves and publish our services, let's write a method to search for a particular service this time:

```java
protected Set<AID> searchForService(String serviceName) {
		Set<AID> foundAgents = new HashSet<>();
		DFAgentDescription dfd = new DFAgentDescription();
		ServiceDescription sd = new ServiceDescription();
		sd.setType(serviceName.toLowerCase());
		dfd.addServices(sd);

		SearchConstraints sc = new SearchConstraints();
		sc.setMaxResults(Long.valueOf(-1));

		try {
			DFAgentDescription[] results = DFService.search(this, dfd, sc);
			for(DFAgentDescription result : results) {
				foundAgents.add(result.getName());
			}
			return foundAgents;
		}
		catch (FIPAException ex) { ex.printStackTrace(); return null; }
	}
```
<br/>

This method will take a `String` for a service name and then search for it throughout the yellow pages. It will then return a `Set` of `AID`s of the agents that are providing the service. The reason that I used `Set` here is the fact that sometimes you might receive duplicates, especially when you poll this service periodically, and that might cause some problems. With `Set`s, on the other hand, there will be no duplicate; so, one less thing to worry about. The method is fairly clear and doesn't need much explanation. The only thing that might seem a little odd is `sc.setMaxResults(Long.valueOf(-1))`. The `setMaxResults` indicates how many results we are looking for: do you need only one agent to do your thing, or do you need all of them? If you want to go with the latter, you should pass `-1` to the `setMaxResults` method. Otherwise, just pass the number of results you're interested in: `1`, for instance. Just remember that this method expects a `Long` value; that's why I cast it first. Come to think of it, the number of results could also be another argument for this method. If you think this happens a lot in your program where you need a different number of results every time, just add another argument to the argument list. For me, it's fine this way.

And that's pretty much it. We now have a method to register our services, and another to find other services. For the last touch, however, I'd like to add another method. This method will deregister our services when our agent is about to die. That's a good thing, since we don't want to end up in a situation where there are some services available but the corresponding agents are not. Hence, we'll add another very short method:

```java
protected void takeDown() {
	try { DFService.deregister(this); }
	catch (Exception ex) { ex. printStackTrace(); }
}
```
<br/>

Here, we are overriding the `takeDown` method of the `Agent` class. It will get called by the framework when the agent is about to die. So, it's a very good place to handle things like deregistering. All we have to do is to call the `deregister` method of the `DFService`. That's it.


At the end, let's have a look at the complete class, with all the `import`s.

```java
import jade.domain.FIPAException;
import jade.domain.DFService;
import jade.domain.FIPAAgentManagement.SearchConstraints;
import jade.domain.FIPAAgentManagement.DFAgentDescription;
import jade.domain.FIPAAgentManagement.ServiceDescription;
import java.util.Set;
import java.util.HashSet;
import jade.core.Agent;
import jade.core.AID;

public class EnhancedAgent extends Agent {
	protected Set<AID> searchForService(String serviceName) {
		Set<AID> foundAgents = new HashSet<>();
		DFAgentDescription dfd = new DFAgentDescription();
		ServiceDescription sd = new ServiceDescription();
		sd.setType(serviceName.toLowerCase());
		dfd.addServices(sd);

		SearchConstraints sc = new SearchConstraints();
		sc.setMaxResults(Long.valueOf(-1));

		try {
			DFAgentDescription[] results = DFService.search(this, dfd, sc);
			for(DFAgentDescription result : results) {
				foundAgents.add(result.getName());
			}
			return foundAgents;
		}
		catch (FIPAException ex) { ex.printStackTrace(); return null; }
	}

	protected void takeDown() {
		try { DFService.deregister(this); }
		catch (Exception ex) { ex. printStackTrace(); }
	}

	protected void register(String serviceName) {
		DFAgentDescription dfd = new DFAgentDescription();
		dfd.setName(getAID());
		ServiceDescription sd = new ServiceDescription();
		sd.setName(getLocalName());
		sd.setType(serviceName.toLowerCase());
		dfd.addServices(sd);
		try {
			DFService.register(this, dfd);
		}
		catch (FIPAException ex) { ex.printStackTrace();
		}
	}

}
```
<br/>

I called this class `EnhancedAgent`, just because it does a little more than the `Agent` class. Now, all you have to do is just to extend this class instead and use the above methods:

```java
class MyAgent extends EnhancedAgent {
  protected void setup() {
      register("buyer");
      HashSet<AID> sellers = searchForService("sellers");
    }
}
```
<br/>

The `setup` method is a good place for such things since it gets called once your agent is created. If you're familiar with React, it's sort of like the `componentDidMount` function.
