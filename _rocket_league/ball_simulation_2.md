---
layout: page
title: "Ball Simulation 2: Rocket League's Collision Meshes"
image: pitch_mesh.png
description: This post describes how we can use meshes taken from Rocket League to detect collisions, allowing us to predict precisely where the ball will be for the next few seconds.
---

In a [previous post](/rocket_league/ball_simulation_1/), 
we looked at a physical model for how the ball behaves in Rocket League.
This model accurately described what happens when the ball hits an
obstacle with a given surface normal, but without a way to find
these collisions (and their surface normals) it was not very useful.

This post describes how we can use meshes taken from Rocket League
to detect collisions, and calculate the surface normal required by 
our bouncing model from earlier.

## Mesh Files

![](pitch_mesh.png)

The RLBot community has collision meshes for the map corners, goals, and 
cylindrical ramps on the sides of the field. I've put these together
into the mesh above, and it can be downloaded 
[here](/notes/RocketLeague/pitch.obj).

## Naive Collision Detection

With the mesh above, we can read the 8028 triangles individually,
and check sphere-triangle intersection tests on each of them in
order to find all possible faces that might be in contact with the
ball. 

A basic implementation of the necessary triangle-sphere intersection
tests needed to accomplish this is given below:

~~~cpp
struct sphere{
  vec3 center;
  float radius;
};

// CCW winding
struct tri{
  vec3 p[3];
};

float distance_between(const vec3 & start, const vec3 & dir, const vec3 & p) {

  float u = clamp(dot(p - start, dir) / dot(dir, dir), 0.0f, 1.0f);
  return norm(start + u * dir - p);

}

bool intersect(const tri & a, const sphere & b) {

  float dist;
    
  vec3 e1 = a.p[1] - a.p[0];
  vec3 e2 = a.p[2] - a.p[1];
  vec3 e3 = a.p[0] - a.p[2];
  vec3 n  = normalize(cross(e3, e1));

  mat3 A = {
    {e1[0], -e3[0], n[0]},
    {e1[1], -e3[1], n[1]},
    {e1[2], -e3[2], n[2]}
  };

  vec3 x = dot(inv(A), b.center - a.p[0]);

  float u = x[0];
  float v = x[1];
  float w = 1.0f - u - v;
  float z = x[2];

  // if the projection of sphere's center 
  // along the triangle normal puts it inside
  // the triangle, then we can just check
  // the out-of-plane distance
  if (0.0f <= u && u <= 1.0f &&
      0.0f <= v && v <= 1.0f &&
      0.0f <= w && w <= 1.0f) {

    dist = fabs(z);

  // otherwise, check the distances to
  // the closest edge of the triangle
  } else {

    dist = b.radius + 1.0f;
    dist = fmin(dist, distance_between(a.p[0], e1, b.center));
    dist = fmin(dist, distance_between(a.p[1], e2, b.center));
    dist = fmin(dist, distance_between(a.p[2], e3, b.center));
    
  }

  return dist <= b.radius;

}
~~~

Let's try this naive approach on a benchmark problem that
randomly generates spheres and checks each triangle for
intersection:

~~~
$ bin/Release/queries_test.exe -naive
successfully read 8028 triangles
completed 1000 queries in 0.995355 seconds
~~~

Although this can improve the fidelity of our ball predictions,
spending an entire millisecond to see if the ball hits the
mesh is way too expensive. We might want to predict ahead 
a few hundred time steps to plan a bot's actions, but 
that really isn't possible with this naive approach.

## Bounding Volume Hierarchies

One way to improve the performance of these geometric queries
is to use a spatial acceleration structure. The basic idea behind
this is to analyze the geometry of our mesh, and systematically
group sections that are close to each other. Then, when we perform
a query (i.e. find which parts of the mesh intersect an given object),
we can cull entire sections of the mesh at a time, quickly zeroing in
on only nearby geometry.

I highly recommend reading through the 3 part series on BVH generation
on NVIDIA's blog:

-[part 1](https://devblogs.nvidia.com/thinking-parallel-part-i-collision-detection-gpu/)

-[part 2](https://devblogs.nvidia.com/thinking-parallel-part-ii-tree-traversal-gpu/)

-[part 3](https://devblogs.nvidia.com/thinking-parallel-part-iii-tree-construction-gpu/)

By first constructing a BVH for our mesh, we can considerably reduce the
time required to perform these queries:

~~~
$ bin/Release/queries_test.exe -bvh
successfully read 8028 triangles
constructing BVH...
completed 1000 queries in 0.000456973 seconds
~~~

So, this has given us a 2000x speedup over the naive implementation (this
is more a statement of how inefficient the naive version is)! This means
we can get the accuracy benefits of using the actual collision mesh geometry,
without sacrificing performance.

I hope to release my sources for the BVH construction and other
Rocket League utilities on github soon.

## Prediction Benefits

These plots compare actual ball trajectories recorded in Rocket
League (solid lines) to the predicted trajectories computed with
this model of ball bounce physics, given only the initial conditions
(dashed lines). These plots are generated from data file 
"episode\_000121.csv" in the dataset provided 
[here](ball_bounce_data.zip).

Ball locations, as a function of time step:
![](bounce_roll_positions.png)

Ball velocities, as a function of time step:
![](bounce_roll_velocities.png)

Ball angular velocities, as a function of time step:
![](bounce_roll_angular_velocities.png)

Here, we see that by using actual geometry, we are able to
resolve not only the simple bounces with the ground (timesteps 0 to 400),
but we can also predict how the ball will roll up one of the cylindrical
sides of the wall (timesteps 400-600). The predicted motion closely 
follows the observed results, even for this long 10-second simulation.
