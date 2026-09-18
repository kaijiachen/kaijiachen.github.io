---
layout: archive
title: "Research"
permalink: /projects/
author_profile: true
---

***

# Teaching a Car to Drift

**The goal is simple to state and hard to do: make an autonomous car lose traction on purpose, and keep it exactly where I want it.**

Most autonomous vehicles are built to never get near the edge. Stay below the friction limit, keep the tires gripping, keep the math linear — and the car is easy to control. That works right up until the moment it doesn't: a child steps out from behind a parked van, a patch of black ice appears at 60 mph, a truck jackknifes across two lanes. In those moments the safest trajectory is often *not* the one that keeps the tires gripping. It is the one that a rally driver would take — brake late, throw the car sideways, and steer with the rear.

Human experts do this. They do it with no model, no solver, and about 200 ms of reaction time. My work is about giving a machine the same ability, but with math behind it instead of intuition.

## Why is this hard?

Drifting is what happens when you drive a car off the end of every assumption that makes vehicle control tractable.

- **The tires are saturated.** Below the limit, lateral tire force is roughly proportional to slip angle — a straight line, and a controls engineer's best friend. At the limit, that line bends over and flattens out. Ask for more force and the tire gives you *less*. All the standard linear machinery quietly stops being valid exactly when you need it most.
- **The equilibrium you want is unstable.** A steady-state drift is a real equilibrium of the vehicle dynamics — but it is a **saddle point**. Left alone, the car does not stay in it; it spins or straightens out within a fraction of a second. Balancing a drift is closer to balancing an inverted pendulum than to following a lane.
- **The model is wrong, and it's wrong differently every day.** Tire-road friction $\mu$ changes with temperature, surface, rubber wear, and whether it rained an hour ago. A controller that needs an exact friction coefficient is a controller that works in simulation.
- **The clock is brutal.** Recovery decisions live on a ~10–50 ms timescale. Whatever the controller does, it has to do it *now*, at the limits of actuation, with steering and throttle already pinned near their bounds.

## Why is it worth it?

Because everything past the friction limit is where crashes happen.

- **Emergency maneuvers.** Obstacle avoidance at highway speed can demand more lateral acceleration than a grip-limited controller is willing to ask for. A vehicle that can operate *through* saturation has strictly more options than one that can't.
- **Ice, snow, gravel, rain.** On low-friction surfaces, ordinary driving already lives near the limit. "Never lose grip" is not a strategy there — it's an assumption that has already failed.
- **Recovery, not just avoidance.** Once a car is sliding — from ice, a gust, a blown tire — it is *already* in the unstable regime. A controller that only knows how to drive in the linear region has nothing useful to say. One that can stabilize a drift can bring the car back.
- **It generalizes.** The real research question isn't cars. It's: *how do you give a robot performance guarantees in the regime where its model is least trustworthy?* That question shows up again in legged robots, aerial vehicles, and manipulation.

## How: model predictive control around an unstable equilibrium

The vehicle is modeled as a single-track ("bicycle") model, where the interesting physics lives entirely in the tire force curves:

$$
\begin{aligned}
m V (\dot{\beta} + r) &= F_{yf}\cos\delta + F_{yr} \\
I_z \dot{r} &= a\, F_{yf}\cos\delta - b\, F_{yr}
\end{aligned}
$$

with sideslip $\beta$, yaw rate $r$, steering $\delta$, and lateral tire forces given by a saturating (Pacejka-style) law

$$
F_{y} = -\,\mu F_z \sin\!\big(C \arctan(B\,\alpha)\big),
$$

Here $\alpha$ is the slip angle. The $\arctan$ is the whole story: it is linear near zero, then flattens. A **drift equilibrium** $x^\star = (V^\star, \beta^\star, r^\star)$ is a fixed point of these dynamics with the rear tire *fully saturated*, $|F_{yr}| = \mu F_{zr}$ — and the Jacobian there has an eigenvalue in the right half-plane. That single fact is why this is a control problem and not a trajectory-generation problem.

The controller solves, at every timestep, a finite-horizon optimal control problem and applies only its first move:

$$
\begin{aligned}
\min_{u_{0:N-1}} \quad & \sum_{k=0}^{N-1} \underbrace{\|x_k - x^\star\|_Q^2}_{\text{stay in the drift}} + \underbrace{\|u_k - u^\star\|_R^2}_{\text{don't thrash the actuators}} \;+\; \underbrace{\|x_N - x^\star\|_P^2}_{\text{terminal cost}} \\[4pt]
\text{s.t.}\quad & x_{k+1} = f(x_k, u_k), \qquad x_0 = x(t) \\
& u_k \in \mathcal{U}, \quad x_k \in \mathcal{X}, \quad x_N \in \mathcal{X}_f
\end{aligned}
$$

Then throw the rest away, re-measure, and solve again. That last part — **re-solving from the true state, every few milliseconds** — is what makes MPC work on an unstable equilibrium: the feedback loop closes faster than the instability can grow, and the constraint sets $\mathcal{U}, \mathcal{X}$ let me write down "the steering rack physically stops here" as a first-class part of the problem instead of a hack.

The open question is what MPC alone cannot give you: a *certificate*. Recursive feasibility and stability guarantees rest on the model, and the model is exactly what's uncertain at the limit.

## What's next: neural Lyapunov control with guarantees

The direction I'm most excited about is closing that gap — learning a controller **and** a proof of its stability at the same time.

The idea: parameterize both a control policy $\pi_\theta$ and a candidate Lyapunov function $V_\theta$ as neural networks, and train them jointly against the Lyapunov conditions themselves:

$$
V_\theta(x^\star) = 0,\qquad V_\theta(x) > 0,\qquad \dot{V}_\theta(x) = \nabla V_\theta(x)^\top f\big(x, \pi_\theta(x)\big) \;\le\; -c\,V_\theta(x)
$$

for all $x$ in a region of attraction $\mathcal{D}$. Violations of these become the training loss. The part that makes it more than curve fitting is **verification**: a falsifier (SMT-based, or via Lipschitz/bound-propagation arguments) searches $\mathcal{D}$ for counterexamples, feeds them back into training, and the loop repeats until none can be found. What comes out is not just a policy that worked on the test set — it's a policy with a certified region of attraction.

Pair that with a safety filter (control barrier functions) that can override the learned policy whenever it would leave the certified set, and the goal is a controller that is:

- **expressive** enough to exploit the nonlinear, saturated regime that makes drifting possible,
- **fast** enough to run in a real control loop (a network forward pass, not an online solve), and
- **certified**, so "it works" is a theorem rather than an empirical claim.

Getting neural Lyapunov methods to scale to real vehicle dynamics — with friction uncertainty, actuator limits, and hardware-in-the-loop validation — is where I'm headed next.

## The platform

Everything gets validated on hardware: a **Traxxas 1:10 Mustang** RC platform, because a car that only drifts in simulation is a screensaver.

Current work at [CARA LAB](https://cara-lab-rice.github.io/) focuses on automated vehicle control beyond stability limits — results coming soon.

[![Autonomous RC Car Drift Demo](https://drive.google.com/file/d/1UGKPXjGwz7lvS_XDcLGZ2pr7NxeT5Hwm/view?usp=drive_link)

*Autonomous drift control on the Traxxas 1:10 platform.*

***

# Undergraduate Research

My undergraduate research focused on the **design of mechatronic systems and the rapid prototyping** of robotic and medical devices, with an emphasis on translating engineering concepts into functional, experimentally validated hardware — work at the intersection of mechanical design, modeling, and applied engineering for biomedical and robotic applications.

**Representative projects:**

1. [Minimally invasive healing of bone implant–cement interfaces by aerogel cement and remote heating](https://doi.org/10.1016/j.device.2024.100680)

2. [Predicting biaxial failure strengths of aortic tissues using a dispersed fiber failure model](https://doi.org/10.1016/j.eml.2024.102287)

3. Senior Design — [An Implantable Finger Prosthetic](https://kaijiasresearch.godaddysites.com/finger-prosthetic)

More detail on these is available on my [undergraduate website](https://kaijiasresearch.godaddysites.com/).

***

# Class Projects

1. **Nonlinear System Analysis & Control** — [Control of a two-link robot manipulator](https://drive.google.com/file/d/1Pj5eOyawrSS5wAX1YZMAgVT9jrD7OW5Z/view?usp=sharing)

2. **Mechatronics** — [Omni-Wheel Robot](https://www.youtube.com/watch?v=egVPhuDyUTs)
