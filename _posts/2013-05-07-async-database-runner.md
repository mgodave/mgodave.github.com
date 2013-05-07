---
layout: post
title: "Listenable Async Query Runner"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Apache [DBUtils][2] has an [AsyncQueryRunner][1] but it doesn't allow you to work with composable futures, specifically [Guava's][4] [ListenableFuture][3]. The following is a simple adapter on the AsyncQueryRunner using Guava's [ListeningExecutorService][5] to run queries and returning ListenableFutures which can be used to build more complicated chains of async operations. Specifically, I'm thinking of the [example][6] given in the Guava docs.

In the next post I'm going to talk a little more about coposable futures in Java.

<script src="https://gist.github.com/mgodave/3120153.js"> </script>

[1]: http://commons.apache.org/proper/commons-dbutils/apidocs/org/apache/commons/dbutils/AsyncQueryRunner.html
[2]: http://commons.apache.org/proper/commons-dbutils/
[3]: http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/ListenableFuture.html
[4]: https://code.google.com/p/guava-libraries/
[5]: http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/ListeningExecutorService.html
[6]: https://code.google.com/p/guava-libraries/wiki/ListenableFutureExplained#Application