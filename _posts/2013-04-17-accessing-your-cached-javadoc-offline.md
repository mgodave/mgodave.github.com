---
layout: post
title: "Accessing your cached javadoc offline"
description: ""
category: 
tags: []
---
{% include JB/setup %}

A few weeks ago I was on a plane and really wanted a way to access the javadocs for some of the libraries I was currently using. I had everything I needed, it was all right there in my maven cache but I didn't have a convenient way to navigate through the documents. My quick fix at that point was to extract the javadoc jar and point my browser to the files. This quickly became a pain when I needed to start looking at other libraries' documents. So, like any lazy engineer, I wrote a simple program to handle all this for me.

What follows is a simple python script that uses the built-in python web server and the zip utilities to read javadocs directly out of the javadoc jar in your maven repository.

[https://gist.github.com/mgodave/5406999](https://gist.github.com/mgodave/5406999)

<script src="https://gist.github.com/mgodave/5406999.js"></script>

This script is simple (and ugly) so expect to edit the code if you need different configuration.

To use it run 

```
python javadoc.py
``` 

You can then hit an index of all cached artifacts with a javadoc jar at http://localhost:8080/m2/ (don't forget the trailing slash) or the specific artifact at http://localhost:8080/m2/[groupId]/[artifactId]/[version]/[index.html | path_to_file]

TODO:
* It's ugly, badly written python. fix it
* Support ivy/gradle caches
* Read the pom.xml/ivy.xml instead of path inference
* Use a web framework instead of manual path parsing