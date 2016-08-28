---
layout: post
title:  "Crossword Puzzles"
categories: "blog"
---

I'm terrible at crossword puzzles but maybe machine learning can save me. Every week my mother finishes the New York Times Magazine crossword and I've taken a few cracks at it myself. I love a good KenKen and nothing beats a Sudoku for airplane rides, but I can never seem to get the references that Will Shortz throws my way.

There are plenty of [Crossword Solvers](http://www.crosswordsolver.org/) which can do searches from a dictionary file for words with known length and some letters, but I'm picturing something that does it all. With a big training set and great NLP it should be easy enough to implement. Watson did it with Jeopardy and crossword answers are much more structured than is that game. I don't have the time now to build it myself but I'd love to see a program which can solve the Saturday NY Times crossword.

IBM and other big companies are pushing the limits of NLP very quickly and this may be the perfect time to use that research to get some garden-variety tools.

My program would have two components.

1. For each word in the puzzle create a list of restrictions (word length and letter placement) on a dictionary file

2. Use machine learning and word restrictions to generate possibilities. If above a threshold, accept the word

These would loop (or maybe coroutine) until I had a complete puzzle. With those at it's core, I'm fairly sure this design would do well.

If this already exists or if I oversimplified the process, shoot me an email! felday *at* brandeis *dot* edu.