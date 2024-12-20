---
title: Generators
weight: 8
---

{{< classic-dartpad >}}

# Generators

Here you'll learn about generators by implementing fizz buzz.

> Fizz buzz is a group word game for children to teach them about division.
> Players take turns to count incrementally, replacing any number divisible by
> three with the word "Fizz", and any number divisible by five with the word
> "Buzz", and any number divisible by both 3 and 5 with the word "Fizz Buzz".

- [Wikipedia](https://en.wikipedia.org/wiki/Fizz_buzz)

## Example

`1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Fizz, 13, 14, Fizz Buzz, 16, 17, Fizz, 19, Buzz, Fizz, 22, 23, Fizz, Buzz, 26, Fizz, 28, 29, Fizz Buzz`

# Code

Implement a fizz buzz generator using
[streams](https://dart.dev/articles/libraries/creating-streams).

```run-dartpad:theme-dark:mode-dart:width-100%:height-460px
{% include exercise path="codelab/lib/fizzbuzz/" %}
```
