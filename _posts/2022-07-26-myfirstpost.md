---
layout: post
title: My first post
category: Random
date: 2022-07-26 12:33:39 +0200
math: true
---

This is my first post on this website! It will just contain random elements, so pretend like you don't see it :)
{: .message }

As my first post, I might aswell just write something completely random, and what better random thing to write than math :)
For shortage of time, and lack of a better option, I will just completely randomly write the solution to an excersise from an 
abstract algebra book.

Let $$ G $$ be a finite group, and let $$ x, y $$ be two elements of $$ G $$ of order $$ 2 $$ such that the set $$ S=\{x, y \} $$ is a 
set of generators of $$ G $$. Prove that $$ G \cong D_{2n} $$ where  $$ n = |xy| $$.

Proof:
If $$ x, y $$ are generators of $$ G $$ then it follows that  $$ xy, x $$ also generate  $$ G $$. Remembering that one of the presentations for
the dihedral group is  $$ D_{2n} = \{ r, s ; r^n = s^2 = 1 \} $$ we observe that this presentation is valid if we choose $$ r = xy \text{ and } s = x $$.
This means that there is a unique homomorphism mapping elements from $$ G $$ to elements of $$ D_{2n} $$. Remembering that the set $$\{r, s\} $$ is a set of generators of $$ D_{2n} $$,  we note that the map is surjective, because all elements of $$ D_{2n} $$ can be obtained from the application of the map to elements of $$ G $$ . Note also that $$ |G| = |H| = 4 $$ and whenever the map is both surjective, and the groups are both of equal finite order, then we know that the map is bijective. Thus
$$ G \cong D_{2n} $$.

