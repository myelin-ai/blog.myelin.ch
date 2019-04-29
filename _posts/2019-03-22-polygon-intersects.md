---
layout: post
title:  "Geometry for coders: Implementing an intersection test for convex polygons"
author: Jan Nils Ferner
---

When implementing a geometric algorithm, it can be hard to translate mathematical concepts into code.
If you ever tried to implemnt your own collision checks, you might have experienced this when reading
about the Separating Axis Theorem (SAT).

## The SAT in a nutshell

The key concept at the heart of the theorem can be simplified as follows:

> If you can draw a line between two shapes, they are not touching each other.

Sounds easy, right? 

When working with convex shapes, this line can be straight.

![01](/assets/polygon-intersects/non-intersecting.svg)
