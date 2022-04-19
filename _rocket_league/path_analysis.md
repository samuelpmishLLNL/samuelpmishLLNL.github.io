---
layout: page
title: "How to Analyze Drivable Paths"
---

## Overview

An important skill in Rocket League is the ability to precisely estimate
how long it will take us to reach a certain destination. These time
estimates let us answer useful high level questions:
- will I be able to reach my destination in time?
- will I have enough boost?
- will I be able to arrive before my opponent(s)?
- is there enough time for me to pick up a boost pad along the way?
- if I can reach the destination in time, what speeds can I arrive at?

The answers to these questions determine the decisions we make, so it 
is essential that our time estimates are as accurate as possible. This
article considers how to analyze a particular drivable path in order
to answer the questions above. 

This analysis does not consider powersliding.

### Curves

Let's say I have a curve $$ \boldsymbol{c}(t) $$, where the parameter
$$t$$ is not necessarily time, but just a number ranging 
from 0 to 1 (0 being the beginning of the curve, and 1 being the end).

<video autoplay loop muted width="640">
<source type="video/webm" src="/curve_parametrization.webm">
Your browser does not support the video element.
</video>

In order to calculate the time to traverse a curve, it is helpful
to first figure out the length of any section of the curve. To get
started, let's zoom in on an arbitrary section of this curve 
corresponding to the parameter values $$(t - \frac{\Delta t}{2}, t + \frac{\Delta t}{2})$$. 

![](//arclength.png)

In blue, we have the actual curve segment, whose length we would like
to know. In red, we use $$\boldsymbol{c}'(t)$$ to construct a line that locally approximates the 
curve. Observe that the actual length, $$\Delta s$$, is pretty close to
the length of the red line segment:

$$ \color{blue}{\Delta s} \; \color{black}{\approx} \; \color{red}{\big|\big|(\boldsymbol{c}(t) + \boldsymbol{c}'(t) \frac{\Delta t}{2}) - 
                      (\boldsymbol{c}(t) - \boldsymbol{c}'(t) \frac{\Delta t}{2})\big|\big|} $$

Simplifying the righthand side of the above gives us:

$$ \color{blue}{\Delta s} \; \color{black}{\approx} \; \color{red}{\big|\big|\boldsymbol{c}'(t) \, \Delta t \big|\big|} $$

Now, since this approximation gets better and better for smaller values of $$\Delta t$$, 
we consider the limit when $$\Delta t \rightarrow 0$$, in which case the $$\Delta$$'s become $$d$$'s,
and we turn the $$\approx$$ into an equality:

$$ \color{blue}{ds} \; \color{black}{=} \; \color{red}{\big|\big|\boldsymbol{c}'(t) \, dt \big|\big|} 
\; \color{black}{=} \; \color{red}{\big|\big|\boldsymbol{c}'(t) \big|\big| \, dt } $$

With this in mind, we can use calculus to add up all of these infinitesimally small lengths.
Let $$s(t)$$ represent the arc length from the start of the curve until it reaches the parameter 
value $$t$$:

$$ s(t) = \int ds =  \int_{0}^{t} || \boldsymbol{c}'(\tau) || \; d\tau $$

In general, this integral cannot be evaluated in closed form,
so we have to integrate numerically. This might sound complicated
at first, but it just amounts to sampling points along that curve
and calculating distances between adjacent points. Note: 
the total length of the curve is $$L = s(1)$$.

It is more convenient to reparameterize our original curve in terms of
arc length, since that directly tells us how much distance we have covered
already and how much we have left. We denote this arc-length-parameterizated curve 
by $$\bar{\boldsymbol{c}}$$, and observe that it relates to the original
in the following way:

$$\bar{\boldsymbol{c}}(s(t)) = \boldsymbol{c}(t)$$

Now that we have a grasp on the arc length of different sections of
the curve, we can calculate the time it would require to traverse
a small section of it:

$$ v(s) \approx \frac{\Delta s}{\Delta T} \qquad \Rightarrow \qquad \Delta T \approx \frac{\Delta s}{v(s)} $$

where $$v(s)$$ is the speed of the car after having travelled a distance $$s$$ along the curve. Like before,
when we make this small section of the curve infinitesimally small, 
we replace $$\Delta$$'s with $$d$$'s, and integrate both sides to 
add up all those small time contributions into a total time:

$$ T = \int dT = \int_0^{L} \frac{ds}{v(s)} $$

All that remains is to determine $$v(s)$$ such that it is as large
as possible, so that the total time will be minimized.

#### Aside: Curvature

The curvature, $$\kappa$$, at a point on a curve is related to the radius of the
osculating circle at that point (shown below in red).

<video autoplay loop muted width="640">
<source type="video/webm" src="/curvature.webm">
Your browser does not support the video element.
</video>

The relationship between curvature and radius is a reciprocal one: $$\kappa = \frac{1}{r}$$, 
where $$r$$ is the radius of the (red) osculating circle.
This definition is consistent with our intuition that straight lines should have zero curvature,
and large curvature implies a tight turn.

In 2D, it is also common to associate a sign with the curvature value, to indicate if the curve is
bending to the left (positive sign) or to the right (negative sign).

If we have an expression for $$\boldsymbol{c}(t)$$ directly (e.g. for the case of Hermite
curves, Bezier curves, Lagrange Interpolating Polynomails, etc), then 
the following formula can be used to calculate the exact curvature:

$$ \kappa(t) = \frac{\boldsymbol{c}'(t) \times \boldsymbol{c}''(t)}{||\boldsymbol{c}'(t)||^3} $$

If our curve is described by a sequence of points, then a finite difference stencil
can be used to approximate the expression above, but I recommend one of the two beautiful 
constructions from [this paper](/assets/curvature_approximations.pdf), 
with the relevant excerpt below (note, they use $$k$$ instead of $$\kappa$$ for curvature):

![](//curvature_approximation.png)

### Approximating $$v(s)$$

The shape of the curve imposes constraints on how fast the car
can be travelling instantenously. In another post, we showed that
the maximum curvature, $$\kappa_{max}(v)$$, is related to the speed of the car
in the following way:

![](//turning_curvature_with_fit.png)

With this in mind, a reasonable first guess for the optimal 
speed plan, $$v(s)$$, is one where we set the speed to the
maximum value allowed by the curvature of the path. Mathematically,
we can express this constraint as:

$$\kappa_{max}(v(s)) = \kappa(s)$$

Let's see what a simulation of this assumption looks like, when 
applied to the example path from earlier 
(using actual the actual curvature data from Rocket League):

<video autoplay loop muted width="684">
<source type="video/webm" src="/naive_speed_limit.webm">
Your browser does not support the video element.
</video>

From the plot of $$v(s)$$, we see that the car can travel at
maximum speed along the straightaways, and it must slow down to take
the sharp turns. Although this is a good first guess, it has some
problems. Most notably, look at how abruptly the speed changes when the
car reaches the sharp right turn. The car in Rocket League
can't actually follow this $$v(s)$$, because its requires accelerations
greater than the ones the car can achieve by boosting and braking.

So, let's go back and modify our model to try and include these
acceleration constraints from the dynamics of the car.

### A Better Approximation for $$v(s)$$

In [this post](/notes/RocketLeague/ground_control/), we showed 
how different controls affect the car's acceleration when driving.
For this analysis, we only care about braking and boosting, since
those are the optimal ways to change our speed the fastest.

For a straightaway, the plot below shows the relationship between
speed and distance traveled when boosting (yellow) and braking (blue):

![](//acceleration_curves.png)

We can use this information to improve our estimate of $$v(s)$$, 
by asserting that the slopes of $$v(s)$$ must be bounded by the values
from these curves. This will guarantee that the updated estimate will
respect the acceleration limits of the car. Below is a comparison of the
original $$v(s)$$ in blue, and the slope-limited form (in yellow):

![](//v_s.png)

By reading this plot, we can see that the optimal time of traversal involves boosting for the
first ~750 units, braking, and then accelerating again once the car is past the turn.
So, performing this analysis has found the exact point along the curve where we need
to start braking, so that we can still meet the speed requirements of the curve.
Additionally, it tells us the maximum possible value for $$v(L)$$ (the speed of
the car when it has reached its destination). 

## Summary

In this post we showed how to understand and calculate the arc length, curvature,
and maximum speeds attainable by a car with speed-dependent turning radius and finite acceleration.
The entire procedure is applicable to almost any curve in space, and we can use these
quantities to numerically evaluate the integral that estimates the least time
to traverse that path.

The planning problem (which path to take) is still an open question, 
but the ability to analyze a particular path is an essential step
toward being able to determine what the optimal path should be.

### Additional Considerations

Other accelerations experienced by the car can also be considered with this approach,
by generalizing the slope-limiting process for $$v(s)$$. If we also consider the 
instantaneous direction the car is facing and the curvature of the path, 
we can model the effects of gravity and turning friction to update our 
range of possible accelerations the car can experience!
