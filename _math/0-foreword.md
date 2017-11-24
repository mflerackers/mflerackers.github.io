---
layout: default
title: Foreword
math: true
---

[Next - Points and Vectors](1-points-and-vectors.html)

# [Math for game development](../) : Foreword

## Foreword

I would have liked to say that this book in progress was written for the love of mathematics, or the goal of revisiting and understanding every detail of the mathematics we use as we research new methods in graphics, write commercial animation software or create games. However that would only be half the truth. An equal amount stems from the frustration of seeing many game developers either struggle with creating their vision, writing wrong or superfluous mathematical statements or even just copy pasting code without understanding what they are doing.

Of course education is certainly to blame in many cases. Mathematics is often taught in a purely theoretical way, mostly in the way of "learn these recipes to solve these problems". Even at university level there is hardly a hint to how to apply what you learn, no emphasis on what's most important and a lack of showing how useful it is beyond passing exams.

I hope that for people reading this as a book, or using it as a reference, the experience will be different. It will tell you what to memorize, and what can be derived from that knowledge. It will show numerous examples of where formulas or concepts are used in real situations. In some cases you'll get insights which are seemingly unknown to most people in the software industry judging from what I've seen.

Ultimately I hope that reading this instills the need to learn more in some of you. The urge to understand mathematics instead of using it as a black box in order to build new methods and experiences on top of that knowledge.

The code provided within the book as examples is written in lua. This is because lua is both very clear in syntax as well as an easy language to learn. The example code is written using functions rather than in an object oriented style in order to prioritize the mathematics. Moreover it is written in a functional style where output only depends on input and no internal or external state is being mutated. In some cases, where objects with overloaded operators make the code more clear, an object oriented style is adopted, however the style is kept functional, thus the objects are immutable in as far as is practical in lua.

This is still an early draft, and will be revised when I find mistakes, places which are unclear or where additional information is needed.
