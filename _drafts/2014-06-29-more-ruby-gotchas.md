---
layout: post
title:  "more ruby gotchas"
date:   2014-06-29 12:00:00
tags:
- coding
- ruby
- gotchas
---


###attribute=()
A common style guide practice is to have equals methods when setting values/attributes.

When you're inside an object you would need to call
`attribute=` with `self.attribute = 10`. Otherwise it would just create a local variable.


###error vs standarderror
