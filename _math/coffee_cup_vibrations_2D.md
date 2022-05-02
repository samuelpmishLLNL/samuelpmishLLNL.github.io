---
layout: page
title: "FEA from Scratch: Coffee Mug Vibrations"
image: mug_thumbnail.gif
description: blah blah

---

There is a great video on numberphile's youtube channel, where Tadashi Tokieda observes an interesting phenomenon: if you tap the rim of a coffee mug with a spoon, you get slightly different pitches, depending on where you tap. He gives a beautiful explanation on why this occurs, and how to understand it intuitively. If you haven't seen this video already, please go watch it!

<iframe width="560" height="315" src="https://www.youtube.com/embed/MfzNJE4CK_s" title="YouTube video player" margin="auto" display="block" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

This problem seems simple, but it actually touches on a lot of math and physics topics that are close to my heart (theory of elasticity, partial differential equations, eigenvalue problems). It makes for a nice example of how the finite element method can be used simulate and analyze physical systems. In these notes, we'll write a small 2D finite element code from scratch in Mathematica to perform the analysis.



## Finite Element Analysis

### Meshing: Discretize the Domain

The starting point for any finite element analysis is a mesh that represents the geometry we care about. For this analysis, we'll approximate the mug geometry in 2D with a coarse mesh made up of a ring of quadrilateral elements, with an extra square hanging off one side to represent the handle:

![test](../images/test.svg)



The next step is to define "shape functions" for the elements that appear in the mesh, so that we can parametrize the solution over the entire domain, by just specifying values at the nodes. For 4-node quadrilateral elements, the shape functions are:
$$
\phi(\xi, \eta) \coloneqq \frac{1}{4}\begin{bmatrix}
(1 - \xi) (1 - \eta) \\
(1 + \xi) (1 - \eta) \\
(1 - \xi) (1 + \eta) \\
(1 + \xi) (1 + \eta)
\end{bmatrix}
$$
or, in Mathematica (the derivatives w.r.t. $\xi, \eta$  will be useful in just a moment)

![zero_energy_modes](../images/quad_shape_functions.svg)



### Physics: Mechanics of Vibrations

Now that the domain has been discretized, we can look at the physics. The classical-mechanics way to describe systems like this is to start off by writing down expressions for the kinetic energy
$$
T \;\;=\;\; \frac{1}{2} \int_\Omega \rho(\bold{x}) \; \dot{u}_i(\bold{x}) \; \dot{u}_i(\bold{x}) \; \text{d}\bold{x}
\;\;= \;\;\frac{1}{2} \bold{\dot{u}}^\top \bold{M} \; \bold{\dot{u}}
$$
and potential energy [^1] 
$$
V \;\;=\;\; 
\frac{1}{2} \int_\Omega \frac{\partial u_i}{\partial x_j}\bigg|_{\bold{x}} \;\; C_{ijkl} \;\; \frac{\partial u_k}{\partial x_l}\bigg|_{\bold{x}} \; \text{d}\bold{x} \;\;= \;\;\frac{1}{2} \bold{u}^\top \bold{K} \; \bold{u},
$$
where $\bold{u}$ is our vector of nodal displacements. From here, the behavior of the system is described by the Euler-Lagrange equations, where $L = T - V$ is the "Lagrangian", 
$$
\begin{align}
\bold{0} &= \frac{d}{dt} \frac{\partial L}{\partial \dot{\bold{u}}} - \frac{\partial L}{\partial \bold{u}} \\ \\
&= \frac{d}{dt} \frac{\partial T}{\partial \dot{\bold{u}}} + \frac{\partial V}{\partial \bold{u}} \\ \\
&= \frac{d}{dt} (\bold{M} \; \bold{\dot{u}}) + \bold{K} \; \bold{u} \\ \\
\Aboxed{
\bold{0} &= \bold{M} \; \bold{\ddot{u}} + \bold{K} \; \bold{u}
}
\end{align}
$$
So, after turning the algebraic crank, we get a vector-valued version of the equation for a simple harmonic oscillator, which makes perfect sense. Now, the "mass" and "stiffness" terms in the equation are actually matrices rather than scalars, but much of the intuition from the 1D problem (sinusoidal solutions, natural frequencies, etc.) still applies. 

Here's some Mathematica code to calculate these mass and stiffness matrices, $\bold{M}, \bold{K}$:

![mug_2D_mass_and_stiffness_calculation](../images/mug_2D_mass_and_stiffness_calculation.svg)

For brevity, I've omitted some steps on how to derive the code that is used to calculate $\bold{M}, \bold{K}$, but a more detailed explanations will be given in a future set of notes.



### Mode Shapes and their Natural Frequencies



[^1]: the tensor $C_{ijkl}$​ is analogous to the spring constant in the expression for the potential energy of a spring $U_\text{spring} \coloneqq \frac{1}{2} \Delta L \; k \; \Delta L$​.



![zero_energy_modes](../images/zero_energy_modes.gif)

```mathematica
frames = Table[
   deformed = nodes + Sin[t] Partition[Q[[1]], 2];
   Graphics[{
     {Opacity[0.1], Line[nodes[[#[[{1, 2, 4, 3, 1}]]]]] & /@ elements},
     Red, Line[deformed[[#[[{1, 2, 4, 3, 1}]]]]] & /@ elements
     }, PlotRange -> 1.75],
   {t, 0, 1.99 \[Pi], (2 \[Pi])/60}
];
ListAnimate[frames]
```

![lowest_two_energy_modes](../images/lowest_two_energy_modes.gif)

![mug_thumbnail](../images/mug_thumbnail.gif)

