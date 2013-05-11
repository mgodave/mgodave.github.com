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

There is nothing specifically wrong with this piece of code except that it's a bit unwieldly and verbose when trying to chain more than one or two operations. For instance, suppose we want to lookup a value from the database, query a list of files based on that value from another datastore or service, download those files and finally merge them into one file. There are four distinct steps here (query, lookup, download, merge) and at least three different services (database, filestore, disk). In order to do this we'd have to build and store of each intermediate result. Even if we remove the anonymouse inner classes we are still left with a rat's nest of things we don't need.

<script src="https://gist.github.com/mgodave/5538189.js"> </script>

To be clear, this is not the end of the world, but, to me, it seems a little cluttered and hard to read and understand, via inspection, what's going on. In addition, I don't really have any use for the intermediate results. It should be easier to chain multiple transforms together. This got me thinking: was there  a more pleasent way to accomplish the same thing? Instead of reinventing the wheel I went looking in the Guava libraries for utilities that might help. I first looked at the collection of static methods in the [Functions][3] class and specifically [Functions::compose][2], but, it seemed unweildly to build a single composition of functions and try to lay async semantics on top. Each function would be a wrapped function/executor combo, this was still blatantly ugly. I then went and looked back at the static methods on the [Futures][4] class and settled on trying to find a better way to build a composition with transform. I settled on using a [Builder][5] pattern to chain [Futures::transform][3] operations while having the builder keep track of the intermediate results. This ended up working quite nicely.

The following is an example using this pattern to accomplish the file merge operation.

<script src="https://gist.github.com/mgodave/5535727.js"> </script>

What's happening here? At each stage of the builder we're returning a new builder parameterized with the input to the first function, the output of this stage's function (which is the input to the next stage), and the output of the final product. In addition, the builder takes a default Executor (line 13) on which each function is executed, by default this uses Guava's MoreExecutors.sameThreadExecutor(). On top of that, each stage can override the default executor for a more appropriate one (as we do on line 17).

You can find this implementation and some other thing's I've been playing with [here](https://github.com/mgodave/AsyncUtils)

[1]: http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction)
[2]: http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Functions.html#compose(com.google.common.base.Function, com.google.common.base.Function)