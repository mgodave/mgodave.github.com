---
layout: post
title: "Composing ListenableFutures"
description: ""
category: 
tags: []
---
{% include JB/setup %}

I'd like to talk a little about composing ListenableFutures, specifically, I'd like to talk about this [example][1] copied from Guava's Futures::transform documentation.

<script src="https://gist.github.com/mgodave/5535286.js"> </script>

There is nothing specifically wrong with this piece of code except that it's a bit unwieldly and verbose when trying to chain more than one or two operations. For instance, suppose we want to lookup a value from the database, query a list of files based on that value from another datastore or service, download those files and finally merge them into one file. There are four distinct steps here (query, lookup, download, merge) and at least three different services (database, filestore, disk). In order to do this we'd have to build and store of each intermediate results. 

<script src="https://gist.github.com/mgodave/5538189.js"> </script>

To be clear, this is not the end of the world, but, it got me thinking if there was a more pleasent way to do this and continuously chain transforming operations. As a side effect, my implementation returns a new AsyncFunction you can pass around and reuse (assuming you have created a pure function object without keeping or modifying internal state).

The following code is a sampling of what I've been working on to create a nicer flow for this type of operation.

<script src="https://gist.github.com/mgodave/5535727.js"> </script>

What's happening here? At each stage of the builder we're returning a new builder parameterized with the input to the first function, the output of this stage's function (which is the input to the next stage), and the output of the final product. In addition, the builder takes a default Executor (line 13) on which each function is executed, by default this uses Guava's MoreExecutors.sameThreadExecutor(). On top of that, each stage can override the default executor for a more appropriate one (as we do on line 17).

In adition, you can split the computation at any point by capturing the builder 

You can find this implementation and some other thing's I've been playing with [here](https://github.com/mgodave/AsyncUtils)

[1]: http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction)