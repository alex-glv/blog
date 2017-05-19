+++
date = "2017-05-19"
draft = false
title = "Om.next and what's next?"
description = "My foray into development of a Single-Page-Apps with om.next"

+++

I enjoy writing Clojure, a lot. I've written Javascript, PHP, Python, C, and Go. Clojure is by far the most satisfying language to write code and reason in. I keep coming back to it.
Whether it's the kind of people it attracts, alien ideas it encourages implementing things in, or simply personal biased preference, I don't know.

That also extends to Clojurescript. Clojurescript and Clojure have been my first combo in writing front and back-end apps.

I wanted a small front-end project to sharpen my skills in full-stack Single Page Applications development and researched my options. This was in November 2016.

First presentations about om.next had been about a year old, yet om.next was still in alpha and was promised to get in Beta "really soon".

The concept of declarative queries for front-end components made a lot of sense and seemed like a breathe of fresh air to the alternatives in the field.

I didn't expect smooth sailing and totally anticipated a lot of unknows and approached the learning with an open mind.

# om.current? om.next?

First off the bat, it seemed very confusing that the "mature" version of Om was in the same repo. Even though om.next was not recommended for use in production, om.current was de-facto "dead". The reason is the fact that they're totally incompatible. This is very similar to Angular 1 and Angular 2 scenario.

The Wiki was pretty good to start with.
Om.next has a only few, albeit really powerful concepts.

## Reconciler

Reconciler is like a messaging hub that receives queries or mutation requests and decides when to fetch the data from the local store or make a remote query to fetch it, and orchestates rendering. It also decides how to merge data into the local tree and when to force component updates.

## Parser

Parser functions are essentially query "decomposers" where you define how to fetch data for your components.

I found parser to be a beautiful decomplecting primitive.
You define a multimethod and receive a key from reconciler that it would like to fetch, and you decide how to obtain that key from the store. 

The simplest case is to fetch the key value from the state map.
I was exhilirated about it's flexibility and application possibilities.
## Query

Query is similar to GraphQL queries where you request the data in whichever shape you prefer, om parses your query and request your parser methods for each key in that query.

Queries can be joins, unions and simple keys. Joins are not dissimilar to SQL joins where you link entities together and fetch associated data.

## Normalization

Another powerful concept in om.next is data normalization.
The driving motivator for normalization is the nature of a front-end application. It can show the same data (or an entity) in different contexts (a list of entities and a widget with "top 5 *something* things").
In order to not have to update all those things at once when needed, your data is normalized and you only have to update it in one location, and the rest are simply links to it. 

Think a live score board of 5 teams. The main area of an app displays the teams and detailed information about each. Also, it holds current playing teams and their score. Once the score updates, you can merge the new score for the entity and whichever other app components show entity related information, simply hold links to the central location, always staying up to date.

You could access or mutate component's bit of data by simply accessing it with ```get-in```:
```
(get-in store [:items :by-id 123])
```

If you mutate that same item, all components that use it in the query will be scheduled for re-render.

# Server side and Datomic's pull query

Om.next's story does not end in the browser, though.
For better or worse, om has a back-end component that also comes with a parser.

Every time your front-end hits the "remote" part (presumably your back-end server) om's handler gets the requests, parses the query and hits the server side parses that will get the data, or perform a mutation.

Hopefully, I am not making too big of an assumption, but you have to buy in to om's backend to be able to use om on the front.

Om integrates *really well* with Datomic.

So well, in fact, that you can pass Om.next queries right into Datomic transactions and return data to the front-end without any processing. And that, I belive was the choice behind the query language as it is.

This was probably the best integration experience so far. 
If you are 100% Clojure and Clojurescript shop, this is the best thing you can get.

Om.next for front-end, Transit for transport, Om.next and Datomic on the back-end. It finally feels like a complete journey, "There and back again" quite in a literal sense, send query there and get it back again in the shape you exactly requested.

# Writing parsers...

This proved the most difficult part.
If you go a tiny bit outside of the tutorial scenario, you are hit with all the complexity of parsing trees.
It's very easy to trip over your whole app by making the tiniest of mistakes. Everytime I have to make changes to the parser, I think "Oh boy, here we go..." 

Something like implementing routing took me a lot of time to figure out how to do. I don't think it should be as difficult as it is now. However, Antony's blog really helped to understand valid approach and stick to one of them.

Since there's a concept of "root" query, that is you tell om.next at which part of the query to stop descending and simply send request to the remote.

It sucks if part of your root query contains a part that is not meant to go remotely, you'll have to rip it apart and put it back together. And while it doesn't seem very difficult in theory, it becomes hairy very fast.

Unfortunately there's no good reference app that implements routing.

My first to go were to-do list implemetation by David Nolan [1] and Antony Monteiro [2].

Second was CicleCI's front-end repo, which is both open-source and uses om.next.

Third reference was Untangled [3] framework which has some amazing deep dive into the om.next concepts.

Om.next had some really steep learning curve for me.
You'll have to analyze the way parsing works and juggle the concepts in your head to make sure you succesfully orchestrate your remote, components queries, parsers
and denormalization.


# Reasons why it doesn't work for me

While it's been an amazing experience learning om.next I don't think I will be using it for my projects any time soon.

Om.next has been in Alpha for more than a year and was meant to be in beta around half a year ago.

It was really demotivating to not see any milestones, and seeing pull requests sitting and collecting dust without any comments, accepts or rejections [4].

One of the reasons in my opinion is both stewards of the project Antony and David are busy working on Clojurescript compiler, so it doesn't seem that there's much traction on om.next.

Another big blow for me is that I can't write om.next app without having it on the server. I'd really love more "unified" commuication approach with my back-end, like GraphQL, where I can have polyglot clients all being able to communicate with my back-end.

It's actually very hard to go and do something for months and then say to yourself, that it's not working out for you and it's better to move on to something, yet it's quite liberating.

I think it makes zero sense to use tool as powerful as om.next for small to medium size apps, it's yet to be seen how it works out for apps at scale. It seems to be working really well for CicleCI [4].


# Final thoughts

I am opening my om.next efforts for the public: "boards-io" [5].

I won't say what it's similar to.
Let's say it's a clone of a product that rhymes with "Hé-llo".

Boards-io has Google OAuth integration, it has boards, columns and tasks to play around with.

You can use it as a reference for how to integrate routing, back-end and database storage for your data.

The end.

[1] https://github.com/anmonteiro/om-next-fullstack

[2] https://github.com/swannodette/om-next-demo

[3] http://untangled-web.github.io

[4] https://github.com/circleci/frontend

[5] https://github.com/alex-glv/boards-io
