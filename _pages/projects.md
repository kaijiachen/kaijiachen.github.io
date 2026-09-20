---
layout: archive
title: "Research"
permalink: /projects/
author_profile: true
---

***

# Teaching a Car to Drift - 1:10 Scale Ford Mustang 

**The goal is simple to state and hard to do: make an autonomous car lose traction on purpose, and keep it exactly where I want it.**

Most autonomous vehicles are built to never get near the edge. Stay below the friction limit, keep the tires gripping, keep the math linear. That works right up until it doesn't: black ice, a child stepping out from behind a parked van, a truck jackknifing across two lanes. In those moments the safest move is often the one a rally driver would make, throwing the car sideways and steering with the rear. Human experts do this with no model and no solver. My work is about giving a machine the same ability, with math behind it instead of intuition.

## Why it's hard

Drifting runs off the end of every assumption that makes vehicle control easy. The tires are **saturated**, so asking for more force gives you less, and the linear machinery stops being valid exactly when you need it. The drift itself is a **real but unstable equilibrium**: left alone, the car spins or straightens out within a fraction of a second, so holding a drift is closer to balancing an inverted pendulum than to following a lane. The **model is never quite right** either, since tire-road friction changes with surface, temperature, and weather. All of this has to be sorted out in tens of milliseconds, with steering and throttle already pinned near their limits.

## Why it's worth studying

Everything past the friction limit is where crashes happen.

- **Emergency maneuvers.** Avoiding an obstacle at highway speed can demand more lateral acceleration than a grip-limited controller is willing to ask for.
- **Ice, snow, gravel, rain.** On low-friction surfaces, ordinary driving already lives at the limit, so "never lose grip" isn't a strategy.
- **Recovery, not just avoidance.** A car that's already sliding is *already* in the unstable regime. A controller that only knows the linear region has nothing useful to say.
- **It generalizes.** The real question isn't cars. It's how you guarantee performance in the regime where a robot's model is least trustworthy.

## The approach: model predictive control

I steer the car toward a **drift equilibrium**, a sideways state that the physics will sustain but won't hold on its own. At every timestep the controller looks a short horizon into the future, picks the best sequence of moves, and applies only the first one:

$$
\min_{u_{0:N-1}} \; \sum_{k=0}^{N-1} \|x_k - x^\star\|_Q^2 + \|u_k - u^\star\|_R^2
\quad \text{s.t.} \quad x_{k+1} = f(x_k, u_k), \;\; u_k \in \mathcal{U}
$$

Then it throws the rest away, re-measures, and solves again, thousands of times a minute. That constant re-planning is what makes it possible to sit on an unstable equilibrium: the feedback loop closes faster than the instability can grow, and the constraint set encodes real limits like "the steering rack stops here" as part of the problem rather than as an afterthought.

## The platform
Everything gets validated on hardware, a **Traxxas 1:10 Mustang** RC car, because a car that only drifts in simulation is a screensaver. 

<div style="display: grid; grid-template-columns: minmax(0, 4fr) minmax(0, 3fr); gap: 16px; width: 100%; max-width: 480px; margin: 20px auto;">
  <img
    src="https://github.com/user-attachments/assets/b29d004a-ea3e-4342-8878-5146632a1036"
    alt="MPC_Top_View"
    style="display: block; width: 100%; height: auto; margin: 0;"
  />
  <img
    src="https://github.com/user-attachments/assets/2e1b008f-2538-4a6f-9b64-3e5015f2854b"
    alt="MoCapDrift"
    style="display: block; width: 100%; height: auto; margin: 0;"
  />
</div>
***

# Undergraduate Research

My undergraduate work focused on the **design of mechatronic systems and rapid prototyping** of robotic and medical devices, translating engineering concepts into functional, experimentally validated hardware.

1. [Minimally invasive healing of bone implant-cement interfaces by aerogel cement and remote heating](https://doi.org/10.1016/j.device.2024.100680)

2. [Predicting biaxial failure strengths of aortic tissues using a dispersed fiber failure model](https://doi.org/10.1016/j.eml.2024.102287)

3. Senior Design: [An Implantable Finger Prosthetic](https://kaijiasresearch.godaddysites.com/finger-prosthetic)

More detail on my [undergraduate website](https://kaijiasresearch.godaddysites.com/).

***

# Class Projects

1. **Nonlinear System Analysis & Control**: [Control of a two-link robot manipulator](https://drive.google.com/file/d/1Pj5eOyawrSS5wAX1YZMAgVT9jrD7OW5Z/view?usp=sharing)

2. **Mechatronics**: [Omni-Wheel Robot](https://www.youtube.com/watch?v=egVPhuDyUTs)
