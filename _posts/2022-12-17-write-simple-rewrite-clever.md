---
layout: post
title:  "Tip: Write simple; rewrite clever"
date:   2022-12-17 21:00:00 -0500
categories: programming tips
---

> When it is infeasible to unit test a large program, consider starting with a basic implementation. This implementation may be slow but should offer better confidence in correctness. Use this basic implementation's behavior to test iterations of the updated version.

I got a variant of this tip from Bruce Moreland's [Chess Programming Topics](https://web.archive.org/web/20071026090003/http://www.brucemo.com/compchess/programming/index.htm). I, unfortunately, cannot find the exact note.

In Chess Programming, it's easy to introduce hard-to-find bugs in the search algorithm especially as we add more search improvements. One way to mitigate this problem is to first implement a basic and slow but correct implementation. We can then make speed improvements on this implementation and compare both implementations' behaviors. 