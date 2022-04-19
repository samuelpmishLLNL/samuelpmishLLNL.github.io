---
layout: page
title: "Aerial Turn Controller"
tags: wip
---

<video autoplay loop muted width="640">
<source type="video/webm" src="aerial_recovery.webm">
</video>

Although most RLBot developers have implemented robust controllers for
steering on the ground, many bots still struggle to precisely control
their cars in the air. In this post, I will describe, in detail, an aerial
controller that can make a car quickly reorient itself to an arbitrary 
direction.

The two main applications for this are aerial recovery (detecting
the surface a car will hit, and turning to land upright) and setting
up aerial hits, as discussed in this post.

If you aren't interested in derivations or how it works, check out
the example code and summary.

## Problem Statement

Let $$\boldsymbol{\Theta}(t), \boldsymbol{\omega}(t)$$ 
denote the orientation matrix and angular velocity of our car,
respectively, at time t. We will use $$\boldsymbol{\Phi}$$ 
to represent the target orientation we want to reach.

Our goal is to find the angular accelerations $$\boldsymbol{a}(t)$$
that bring the car's orientation to the desired state, and keep it
there. After that, it is a simple matter to find the actual 
roll, pitch, and yaw control values that produce those 
angular accelerations.

## 1D Example

Imagine a scenario where a car has just hit an aerial, and is now
falling toward the ground. We would like to turn the car so that it
lands cleanly.

IMAGE

In order to reorient as fast as possible, we want to turn with
the greatest acceleration we can. However, if we keep 
accelerating at maximum until reaching the right angle, the car
will spin right past the desired orientation:

IMAGE

Instead, by changing the angular acceleration at the right time,
we can ensure that the car stops spinning at the same time it reaches
the desired orientation:

IMAGE

The question is: how do we know when to make this change?



## Summary
