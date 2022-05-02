---
layout: page
title: "Teach your bot how to aerial"
image: aerial_hit.jpg
description: For new Rocket League players, reliably hitting aerials is one of the more difficult mechanics to master, but for bots it's one of the simplest. These notes show how to control your bot to aerial to a specific location at a specific time.
---

<video autoplay loop muted width="800">
<source type="video/webm" src="/videos/aerials.webm">
</video>

One of the most useful and impressive maneuvers in Rocket League
is the aerial hit, where a car changes its orientation and feathers
the boost in such a way that it intercepts the ball. Although it takes
most people a while to develop intuition about how to aerial successfully,
we will see that it is a very simple mechanic to implement for a bot.
In particular, the following description will allow us to 
quantitatively determine:

* if the car can aerial to a certain point in a given amount of time
* how much boost it would take to reach that point
* how to control the car to carry out the aerial maneuver

## Problem Statement

Let $$\boldsymbol{c}(t)$$ denote the position of our car, and 
$$\boldsymbol{f}(t)$$ be the unit vector in the direction 
the car faces, both in world coordinates.

Additionally, we will take $$B(t), g$$ to denote the accelerations
due to boost and gravity (respectively), and $$\boldsymbol{P}$$
will be our target location.

For clarity, we will assume that the car has already left the ground, 
so that we can focus our attention on just the dynamics of aerials.

## Kinematics

The car's motion is described by an ordinary differential equation
that includes contributions due to boost and gravity, and initial conditions:

$$
\ddot{\boldsymbol{c}}(t) = B(t) \; \boldsymbol{f}(t) + g \; \boldsymbol{z}, \quad 
\dot{\boldsymbol{c}}(0) = \boldsymbol{v}_o, \quad
\boldsymbol{c}(0) = \boldsymbol{x}_o, \quad
\boldsymbol{f}(0) = \boldsymbol{f}_o
$$

where $$\boldsymbol{z}$$ is the unit vector in the z-direction. We would
like to find those functions $$B(t), \boldsymbol{f}(t)$$ that produce the
trajectory that intersects our target at the prescribed time, $$\Delta t$$:

$$
\boldsymbol{c}(\Delta t) = \boldsymbol{P}
$$

Because the original differential equation is separable, we can
directly integrate twice to get

$$
\boldsymbol{c}(t) = \boldsymbol{x}_o + \boldsymbol{v}_o \; t + \frac{1}{2} \; g \; t^2 \; \boldsymbol{z} + 
\int_0^t \int_0^\tau B(\sigma) \; \boldsymbol{f}(\sigma) \; d\sigma d\tau
$$

Now, we demand that the intersection condition is satisfied:

$$
\boldsymbol{c}(\Delta t) = \boldsymbol{P} = \boldsymbol{x}_o + \boldsymbol{v}_o \; \Delta t + \frac{1}{2} \; g \; \Delta t^2 \; \boldsymbol{z} + 
\int_0^{\Delta t} \int_0^\tau B(\sigma) \; \boldsymbol{f}(\sigma) \; d\sigma d\tau
$$

Rearrange terms to group all the things we can control on the left side, and everything
else on the right:

$$
\int_0^{\Delta t} \int_0^\tau B(\sigma) \; \boldsymbol{f}(\sigma) \; d\sigma d\tau = 
\boldsymbol{P} - \boldsymbol{x}_o - \boldsymbol{v}_o \; \Delta t - \frac{1}{2} \; g \; \Delta t^2 \; \boldsymbol{z}
$$

Although this expression is very general, it isn't immediately obvious how to use it to determine
our controls. What if we assumed that the car did the following: 

1. take some amount of time, $$0 < t < T$$, to turn the car so it faces a certain direction $$\boldsymbol{f}^*$$, without boosting
2. keep facing that direction, and boost with some constant acceleration for $$ T < t < \Delta t $$

More rigorously: $$ B(t) = \begin{cases}
  0 & 0 < t < T \\
  B_0 & T < t < \Delta t
\end{cases}, \qquad \boldsymbol{f}(t) = \begin{cases}
  q(t) & 0 < t < T \\
  \boldsymbol{f}^* & T < t < \Delta t
\end{cases}$$

If we put these expressions into the double integral above and simplify, we get:

$$
\frac{1}{2} B_0 \; (T - \Delta t)^2 \boldsymbol{f}^* = 
\boldsymbol{P} - \boldsymbol{x}_o - \boldsymbol{v}_o \; \Delta t - \frac{1}{2} \; g \; \Delta t^2 \; \boldsymbol{z}
$$

Dividing through by some constants gives us the working equation:

$$
B_0 \boldsymbol{f}^* = \frac{2}{(T - \Delta t)^2} \bigg(
\boldsymbol{P} - \boldsymbol{x}_o - \boldsymbol{v}_o \; \Delta t - \frac{1}{2} \; g \; \Delta t^2 \; \boldsymbol{z}
\bigg) = \bar{\boldsymbol{A}}
$$

## Interpretation

Upon closer inspection, we see that each side of the working equation above has the dimension of acceleration.
This means that we can interpret the right-hand side as the acceleration needed to arrive at target
$$\boldsymbol P$$ in time $$\Delta t$$.

This means that $$ B_0 = \|\bar{\boldsymbol{A}}\| $$ and 
$$ \displaystyle \boldsymbol{f}^* = \frac{\bar{\boldsymbol{A}}}{\|\bar{\boldsymbol{A}}\|} $$

> How can we tell if $$P$$ is reachable by this maneuver? 

Simply: check if $$B_0$$ is smaller than the car's actual boost acceleration (~1000 u/s^2).
If the maneuver requires an acceleration larger than what the car can achieve, we cannot
get there in time.

> How much boost will I need to complete this maneuver?

If we know the average acceleration, $$B_0$$, and how long we will be boosting $$(\Delta t - T),$$
then we know how long we will have to hold the boost button down, so we know how much boost it will take.

> How do I feather boost to produce the right average acceleration?

Keep a running counter for the boost, and do something like this:


~~~python
class Aerial:

  def __init__(self):
    self.boost_counter = 0.0
    self.B_max = 1000.0
    ...

  def get_output():

    ...

    # compute A, B_0
    A = ...
    B_0 = norm(A)
    
    use_boost = 0
    
    # set the boost in such a way that its duty cycle
    # approximates the desired average boost ratio
    use_boost -= round(self.boost_counter)
    self.boost_counter += B_0 / self.B_max
    use_boost += round(self.boost_counter)
    
    self.controls.boost = 1 if use_boost else 0

    ...

~~~

> How do I make my car turn to the correct direction, $$\boldsymbol{f}^*$$?

Stay tuned for an aerial turn controller, that also provides estimates for how
long it takes to perform the turn.

> How do I freestyle with my bot?

To make contact with the ball, you only need to face the front of the car in the correct
direction, meaning you can keep rolling as much as you want once the front direction is aligned.
Additionally, you can boost more than the average acceleration to make the necessary
course correction in less time. Once the necessary acceleration is close to zero, 
your car is already on an intersection trajectory, so you can stop boosting and just 
spin around in any direction you want.

## Summary

To hit an aerial, use the working equation below to find the acceleration magnitude and direction
required to make contact. 

$$
\bar{\boldsymbol{A}} = \frac{2}{(T - \Delta t)^2} \bigg(
\boldsymbol{P} - \boldsymbol{x}_o - \boldsymbol{v}_o \; \Delta t - \frac{1}{2} \; g \; \Delta t^2 \; \boldsymbol{z}
\bigg)
$$

$$ B_0 = \|\bar{\boldsymbol{A}}\| $$ and 
$$ \displaystyle \boldsymbol{f}^* = \frac{\bar{\boldsymbol{A}}}{\|\bar{\boldsymbol{A}}\|} $$

If the directions are aligned (i.e. $$\boldsymbol{f} \approx \boldsymbol{f}^*$$), feather boost to produce the right
acceleration, $$B_0$$, otherwise perform an aerial turn to align the car with the appropriate direction, 
$$\boldsymbol{f}^*$$.
