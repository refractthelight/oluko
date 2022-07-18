---
layout: post
title:  "ggchess: Building a Chess Engine"
date:   2022-07-17 21:00:00 -0700
categories: chess algorithms go ggchess
---

Over the next several months, I will work through [chessengines.org][ce] to develop a chess engine, `ggchess`. The final product will be a simple UI and a [Go] implementation of the engine.

I have three objectives for this project:

1. **Explore game playing algorithms:** I'm looking forward to understanding and implement algorithms like [Alpha-beta pruning][ab] and [Monte Carlo tree search][mcts].
2. **Strengthen programming skills:** Implementing an engine is a non-trivial task and I expect this project to stretch my programming abilities. Additionally, I've enjoyed using Go at work and I hope to master advanced features like [concurrency]. 
3. **Document regularly:** I decided to seriously document my projects after being inspired by the Martin Molin's [Marble Machine X][mmx] project. I will share my progress and reflections in a series of blogposts. I hope these posts will be an opportunity to receive feedback and ideas from readers. 

The source code will be at [refractthelight/ggchess][ggchess].

## Inspiration

I'm heavily inspired by the [General Game Playing][ggp] project at Stanford University and DeepMind's [AlphaGo][ag]. I highly recommend the beautiful [AlphaGo Documentary][agd] and Lex Fridman's [interview] with Demis Hassabis, DeepMind CEO.

## Links

* [ggchess: Go Chess Engine][ggchess]
* [Chess Engines: A Zero to One][ce]
* [Go Programming Language][go]
* [Go Concurrency][concurrency]
* [Alpha-beta pruning][ab]
* [Monte Carlo tree search][mcts]
* [Building the Marble Machine X][mmx]
* [General Game Playing][ggp]
* [AlphaGo][ag]
* [AlphaGo - The Movie][agd]
* [Demis Hassabis: DeepMind - AI, Superintelligence & the Future of Humanity][interview]

[ab]: https://en.wikipedia.org/wiki/Alpha%E2%80%93beta_pruning
[ag]: https://www.deepmind.com/research/highlighted-research/alphago
[agd]: https://www.youtube.com/watch?v=WXuK6gekU1Y&ab_channel=DeepMind
[ce]: https://www.chessengines.org/
[concurrency]: https://go.dev/tour/concurrency/1
[ggchess]: https://github.com/refractthelight/ggchess
[ggp]: http://ggp.stanford.edu/stanford/index.php
[go]: https://go.dev/
[interview]: https://www.youtube.com/watch?v=Gfr50f6ZBvo&ab_channel=LexFridman
[mcts]: https://en.wikipedia.org/wiki/Monte_Carlo_tree_search
[mmx]: https://www.youtube.com/watch?v=Ld7zTApixXE&list=PLLLYkE3G1HED6rW-bkliHbMroHYFf4ukv&ab_channel=Wintergatan
