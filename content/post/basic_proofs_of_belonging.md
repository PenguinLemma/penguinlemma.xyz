---
title: "Basic proofs of belonging"
date: 2020-06-13
draft: true
categories: [maths]
tags: [numbers, proofs, construction]
---
A real number is said to be *rational* if it can be expressed as a *ratio* between to integers. For example, $ 0.5 = 1/2 $, $ -2.8 = -14 / 5 $ and $ 2 = 2 / 1 $ are rational numbers.

We use the symbol $ \mathbb{Q} $ to represent the set of all rational numbers. These numbers behave very rationally with the basic operations. Like one would expect them to behave, to put it simply.

For example, if one adds or substracts two rational numbers, the result is again a rational number. Let\'s write this in a mathematical notation and see if we can prove it:

{{% math/statement Theorem %}}
If $a, b \in \mathbb{Q}$, then $ a+b, a-b \in \mathbb{Q}$.
{{% /math/statement %}}

If we want to *prove* that a certain number $r$ is rational, we have to write $r$ as a fraction, i.e.

{{% math/note %}}
To <b>show</b> that $r \in \mathbb{Q}$, we need to we have to <b>find</b> integers $p$ and $q$ such that $r = \frac{p}{q}$.
{{% /math/note %}}

Analogously, if we *know* that a number $r$ is rational, we can use the fact that $r$ can be expressed as a fraction in order to deduce other statements. In this case, we will say something along the lines of

{{% math/note %}}
Since $r$ <b>is</b> rational, we <b>know</b> that there are two integers, $p$ and $q$, such that $r = \frac{p}{q}$.
{{% /math/note %}}

{{% math/proof %}}
Since $a, b \in \mathbb{Q}$, there are $p, q, r, s \in \mathbb{Z}$ such that $a = p/q$ and $b = r/s$.
{{% /math/proof %}}
