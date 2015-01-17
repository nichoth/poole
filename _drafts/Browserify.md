---
layout: post
title: Browserify
published: false
---

This post will have stuff about using browserify.

* http://ampersandjs.com/learn/npm-browserify-and-modules

## Build Process ##
* http://substack.net/task_automation_with_npm_run

**Problem**: Beefy doesn't refresh right.

## Shimming ##
**Problem**: We need to use Visal Search, a standard JS utility, in a project using CommonJS modules and Browserify.

* https://github.com/substack/browserify-handbook#shimming
* https://github.com/thlorenz/browserify-shim

## ? ##
What is a solution to the `../../../../` problem? We need some global config variables for browserify / node.


## NPM Modules ##
*Problems with small modules?*

* harder to piece together because modules are so generic? See Yehuda Katz [video](https://www.youtube.com/watch?v=NM4EyOmaPbE#t=1468):
>It's better if the framework is trying to figure out how this all fits together, because trying to piece together a bunch of solutions that work for each one of these individual boxes doesn't necessarily produce a coherent whole story.

* Code bloat because different NPM modules have different dependencies that do the same thing. (One module depends on Bluebird, another depends on Q, for example).

