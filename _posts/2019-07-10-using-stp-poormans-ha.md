---
layout: post
title:  Leveraging Spanning Tree as A Poor Man's Redundancy - PVST+
date:   2019-07-10 15:19:25 -0600
image:  SDN-2.jpg
tags:   [Networking, Cisco]
---

Most people associate STP or Spanning Tree with Layer 2 Network Loops, and only see it as a safety net to prevent loops that would bring the network to its knees. What we should keep in mind is that if leveraged correctly, L2 redundancy can be achieved through STP. If we refer back to our fundamental triad of security - Confidentiality, Integrity and Availability, We can see that High availability should always be one of our main goals to achieve. Unfortunately, it doesn't matter what sector you are in or who your client is, the cost is always a factor which means that sometimes you aren't able to implement some of the more fancy HA configurations and you have to make do with what you have.

Let's take a look at the following diagram, which depicts something you would regularly see at a small organization that has seen some "organic" growth. (ie - poor network planning)

{:refdef: style="text-align: center;"}
![]({{site.baseurl}}/img/stp-ha/organicgrowth.png)
{: refdef}

> By just adding a few cables and without redesigning the network completely, we can add a bit of redundancy.

{:refdef: style="text-align: center;"}
![]({{site.baseurl}}/img/stp-ha/switchha.png)
{: refdef}

Now we could just leave it at that and let STP automagically figure itself out by letting it kill the network loop and finding a route for network traffic to flow through. But this is your time to shine by adding a bit of polish to this impromptu redundancy. 

## Taking it a step further

Now we could just leave it at that and let STP automagically figure itself out by letting it kill the network loop and finding a route for network traffic to flow through. But this is your time to shine by adding a bit of polish to this impromptu redundancy.

## Taking it a step further  
Before we start, we need to understand how the switches automagically choose which path to take. Switches are not routers, and as such, they don't care where your data is coming from or what path it needs to take to get to its destination.

At the very basic level of STP, a group of switches interconnected on the same network domain will elect a "root" switch, this switch is ideally one that is connected to everything and has the shortest path to your desired destination. In a non-configured environment this election takes place based on which switch has the lowest MAC address. Once the root switch is identified then the switches perform calculations on the connections and assigns them a cost, which simplified, are based on the speed of that connection. STP then chooses to block the slowest path and keep the faster one open. If the costs are identical for multiple connections, then again the lowest MAC address wins out.

Instead of allowing the switches to self elect the root, we can assign the switch a root designation with the following command on the switch you want to be root:  
##### <strong style="color: red;">DO NOT RUN ANY COMMANDS ON A PRODUCTION NETWORK WITHOUT KNOWING WHAT IT WILL DO</strong>
{% highlight cisco %}
#enable
#config terminal 
#spanning-tree vlan 1 root primary 
#exit 
#show spanning-tree summary
{% endhighlight %}

After the spanning-tree summary command you can verify that the switch is now the root bridge for VLAN1
{:refdef: style="text-align: center;"}
![]({{site.baseurl}}/img/stp-ha/sw1spanningtree.png)
{: refdef}

So now the other switches will know that they need to optimize their STP calculations for getting to the root bridge. <strong style="color: red;">One thing to note is that whenever STP calculates paths, all ports that are connected have to go through the Spanning Tree steps, which means there will be a 30-50 second disruption across the network.</strong>

Now that we know a little about the magic of how STP works, let's play out a scenario: Going back up to our diagram, let's say that STP has decided to block Connection 4 in order to prevent a loop from happening and Connection 3 was left open. Then Connection 3 fails in some way (cable cut, disconnected, etc)

Detecting a change in the topology, STP recalculates, the port that was initially being blocked will move through the listening, and learning states before entering forwarding. This can take some time, thus causing downtime. In some cases this can cause the entire topology to recalculate causing ALL of the switches to recalculate causing your network to go down for a bit.

## Enter PVST+
The difference with PVST+ is in the initial calculation when the loop-free topology is built, redundant ports facing upstream to the root bridge will be an alternate port. This means that it knows that the alternate port is a way to the root bridge, so should the primary link fail, it doesn't need to recalculate - it can immediately jump to the forwarding state and push traffic as normal, instead of being forced to run through all of the phases again. If you would like to change the active and alternate port, then all you need to is to run the following command on the interface you do not want to be primary:

{% highlight cisco %}
#spanning-tree [ vlan vlan-id ] cost 900

##To reset cost back to default 

no spanning-tree [ vlan vlan-id ] cost
{% endhighlight %}

The good thing is that out of the box recent switches come with PVST+ enabled and you'll only run into the aforementioned STP problems in switches that do not support it, if they are older switches that need to be configured in PVST+ or if you are running switches that don't support it.

## Taking it even further!
You could implement additional things such as [EtherChannel](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3750x_3560x/software/release/12-2_55_se/configuration/guide/3750xscg/swethchl.html){:target="_blank"} for additional throughput and redundancy, [Switch Clustering](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960/software/release/12-2_55_se/configuration/guide/scg_2960/swclus.html){:target="_blank"} for ease of management, or something that will greatly benefit this type of scenario - [Port Fast & BPDU Guard](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst4000/8-2glx/configuration/guide/stp_enha.html){:target="_blank"}. 
But I'll leave that as an excercise for the viewer. If you don't have the gear to test, I suggest you check out [eve-ng](https://www.eve-ng.net/){:target="_blank"} a great network simulator that is community driven and free to use. 

Feel free to leave questions or comments, if I'm completely wrong about anything let me know!

Until later - Kevin 

References:  
[Information about Rapid PVST+](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus5000/sw/configuration/guide/cli_rel_4_0_1a/CLIConfigurationGuide/RPVSpanningTree.html#16893){:target="_blank"}  
[STP Scenario](https://networkengineering.stackexchange.com/questions/12687/how-to-properly-configure-stp-in-this-simple-setup){:target="_blank"}  
[eve-ng: Virtual Environment for Networks](https://www.eve-ng.net/){:target="_blank"}  
