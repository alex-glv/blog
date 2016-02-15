---
title: "Containers are not real"
description: "The illustratory example of creating your own containers in Linux"
date: "2016-02-16"
categories:
  - "containers"
tags:
  - "docker"
  - "containers"
  - "linux"
---

# The advent of containers

Containers seem to have turned the industry upside down and marching full ahead to become the dominant way of shipping software.
To uninitiated mortals, Docker seems to be kind of a strange child.
Where does it come from, what's it good at?
It looks like VM, but everyone keeps telling us not to compare it to VMs.
Even though it gained a massive traction, it still has a long way to go to be a de-facto standard every developer should be familiar with.

In this post I'll go through the elements that make up the software container, what underlying infrastructure allows it to exist and how to make a DIY container on a clean ubuntu machine.

We will be creating our own busybox container!

# Namespaces
To create illusion of a container, Docker (and alike) utilizes Linux kernel feature - namespaces.

As the name suggests, namespaces allow isolation of global resources, similar to programming language namespaces that allow the namespace functionality to stay encapsulated from the rest of the codebase. 
It appears as if a  process or a group of processes are in control of these resources.

There are 6 different namespaces that allow isolation of different types of system resources:

- UTS 
- Mount
- User
- Network
- Proc
- IPC

I will not go too deeply in the discusssion of namespaces, you can read more about them in a brilliant series of articles by Michael Kerrisk on linux kernel namespaces. https://lwn.net/Articles/531114/

So, by utilizing resource isolation processes create an illusion of a standalone container.
Interesting thing that if you search for words "containers" in linux kernel, you will find nothing related to actual concept of containerization. 
This is it, the containers don't actually exist, they are not real, we make them real.
Since the genie is out of the bottle now, let's recreate a busybox container from scratch.

# Prerequisites

I am using DigitalOcean with a clean Ubuntu 15.10 virtual machine, and nothing more, so you can easily reproduce the results.

# Network interfaces

Containers wouldn't be very interesting if they didn't have access to the internet.
It's like lego bricks without the studs ("stud" seems to be an official term to call those little pimples on Lego bricks http://thebrickblogger.com/2010/11/lego-disctionary-basic-term/).

Luckily, linux kernel provides an option of network namespaces that allow a process to have its own isolated network stack. We connect that stack with the host network namespace with virtual ethernet devices.

First, let's create our network namespace:
```ip netns add container```

Ensure that kernel allows packet forwarding:
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Now, we'll create a pair of virtual ethernet devices. Think of them as a pipe where interfaces are on both ends.
I like to think of them as portals to the alternate universe.

```
ip link add veth0 type veth peer name veth1
```

We created a pair of interfaces veth0 and veth1.
We'll add a tap device that will bridge our virtual device with the real (or not) hardware adapter:

```
ip tuntap add tapm mode tap 
ip link set dev tapm up
ip link add brm type bridge
ip link set dev tapm master brm
ip link set dev veth0 master brm

ip addr add 10.0.0.1/24 dev brm
ip link set dev veth0 up
ip link set dev brm up
```

And iptables rules to allow traffic forward:
```

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE 
```

Now, we'll move one of them into a separate network namespace which we created before:

```
ip link set veth1 netns container 
```

Let's add the IP address and bring them up:

```
ip netns exec container ip addr add dev veth1 10.0.0.2/24
ip netns exec container ip link set dev veth1 up
ip netns exec container ip route add  default via 10.0.0.1
```


Let's try out if our container network interface has access to the internet:
```
ip netns exec container ping -c1 8.8.8.8
```
If our set-up went well, you should get a successful ping response.

http://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/

OK, we've managed the hard part, let's create our container's file system and do the rest of the housekeeping

# Enter the namespace

Let's enter the configured network namespace:
```
ip netns exec container bash
```
By executing this command, we only separate network namespace. Everything else is still shared with the parent process.
That's right, you can share one or many namespaces with other processes.

Now, let's move forward and unshare other namespaces. The only namespace we will not unshare is user namespace, because it makes the whole configuration more complex and is it's not necessary for our demonstrational purposes.

To unshare namespaces, we'll be using ```unshare``` command.
Unshare command will detach from the namespaces specified by flags ```-m``` (mount namespace), ```-p``` (PID namespace), etcetera.

```
unshare -m -p -i -f --mount-proc
```

Let's also mark mount point as private, so non of our mounts leak:
```
mount / --make-rprivate
```

We will install busybox chroot environment using the following gist posted by github user "weakish":
```
curl -O https://gist.githubusercontent.com/weakish/927135/raw/40b870c8702c52a8d0ea6d3d689e45228055c7c3/busyroot.sh  && chmod +x busyroot.sh
./busyroot.sh init
chroot /chroot/ bin/sh
busybox install
```

The above gist creates a chroot environment, bootstraps it with need devices and files, nothing too fancy.

After executing the commands, we have a fully functioning isolated process with network access and own processes tree.
If you run:
```
echo $$
```
You should see 1, it means that our process has process id 1 on the system.

# Conclusion
Hopefully, this post has shed some light on the mystery of containers.
We learned to unshare namespaces, create network configuration that allows access to the internet and create a secure environment isolated from the global state of the host.
