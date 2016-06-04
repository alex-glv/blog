+++
date = "2016-05-08"
draft = false
title = "Refactoring business-critical PHP application with RabbitMQ"
description = "In this post I'll walk through the steps I took to improve critical partrs of PHP application with the help of RabbitMQ"

+++

# The Problem

While doing contract work optimizing and improving stability of an old PHP-based billing application I was constantly facing issues with long running processes that failed and couldn't easily restore their state.

The actual process was performing daily billing services based on subscription rules.

The process would start, extract a whole bunch of information at once and start processing it one by one, withou keeping an intermediate progress results.

It would take input and produce side effects, churning along as it goes through.

Imagine a manufacturing process. You probably see a conveyer belt and items at various stages being combined with other items through molding and fusing and cooling and whatnot.

What you don't imagine is a room with closed doors, someone puts all the required ingredients at once through a narrow window. They're being ingested and rumbling and grumbling starts. In 3 hours your side-effect (a manufactured item) is spit out.

This is fine until it isn't and something breaks.
If something breaks, you want to know exactly where it happened by looking at the intermediate state.

# Solution: queues to the rescue

I decided to break down the process by introducing queues. Simply because I wanted to learn about queues and thought it would be a great way to break the complexity of the system.

> If you are not using queues extensively, you should start right away
> - Rich Hickey

I was vaguely familiar with RabbitMQ and after reading some documentation decided it was a good fit.

# AMQP - the messaging protocol

AMQP is a flexible messaging protocol. It defines basic roles and entities involved in the message lifecycle.
This is the standart that RabbitMQ conforms to. There's a great introduction article on RabbitMQ[1] website that will help you with the basics. It's a prerequisite to using RabbitMQ. 

You don't want to lose all your messages in the queue if all of your subscribers go down, so you have to specify appropriate options.

# Define roles - workers, producers and messages

You have to break down the service into multiple roles.
Recurring billing service might be broken down in multiple stages like:
- Find subscriptions due to be billed
- Eliminate expired credit card records
- Eliminate records that have cancelled their billing preferences and opted out
- Schedule all the rest

The last item will be our "messages".
We'll send the user data over the queue system to the other side, where it will be picked up by our workers.

Workers will be the following:
- Biller 
- Deactivator

Biller will try to perform the charge and Deactivator will manage the account suspension.

This approach solves a couple of issue:
- Scalability: we are able to attach multiple workers for each role and easily scale our throughput
- Separation of concerns: our small services will each reside in their own space and will be neatly separated 
- Refactoring space: it becomes easier to reason about the component potentially allows better deployment strategies. In addition, testing becames much simpler with having less mocks, because we'll offsource a lot of work on our messaging platform

# Considerations

Having workers sitting and waiting for work to do means that the process will be long lived with all the implications.
You have to support the lifecycle, make sure the processes are started if they go down and try to eliminate all possible memory leaks.

I have been using supervisor[2] to manage the processes.

# Queue listeners

Here's an example of a long lived queue listener:

```
    protected function listen($queueName) 
    {
        /** @var \BunnyAcme\Queue\Driver\Driver $driver */
        $workers = $this->queueManager->getWorkers($queueName);
        $driver = $this->queueManager->getDriver(); // this will be AMQP driver

        $driver->listen($queueName, $workers); // listen will block
    }
```

Here, getWorkers() method will return all the instances and of workers responding to our queue name.
The underlying structure is an associative array where the index is the queue name and the value is the implementation class that conforms to the Worker interface.
Simple worker interface:

```
<?php

namespace BunnyAcme\Queue\Workers;

interface Worker {
    public function __construct($container);
    public function handleJob($payload);
}
```
