---
layout: single
title:  "Predicting Control Surface Hinge Moments using Computational Fluid Dynamics"
date:   2025-04-01 17:37:00 -0600
categories: projects
use_math: true
header:
  overlay_image: /assets/images/barycentric/barycentric-colors-blackfill.png
  og_image: /assets/images/barycentric/barycentric-colors-blackfill.png
  teaser: /assets/images/barycentric/barycentric-colors-blackfill.png
  actions:
    - label: "Download the Thesis!"
      url: /assets/docs/Simpson-2016-hinge-moment-cfd.pdf
tags:
  - CFD
  - Master's Thesis
---

Designing a control surface is a tricky balance between giving adequate control authority and keeping a low stick force (or actuator force).
Control surfaces and their hinge geometry are traditionally designed based on heritage design, experience, and wind tunnel testing.
Wind tunnel testing in particular is costly, and design changes at that stage of the engineering process are especially costly.
With modern computing resources being what they are, what if we could calculate hinge moments using CFD?
That is the question I sought to answer with my Master's research.
This article is meant to be a pretty high-level summary.
The full thesis is available [here](/assets/docs//Simpson-2016-hinge-moment-cfd.pdf) if you are interested in the complete details.

# Introduction

With this research, I wanted to explore the full range of computational methods.
This means 2D and 3D, and everything from a regression on historical data to the cutting edge CFD techniques.
I had the privilege of interning at NASA's Computational Aerosciences Branch (CASB) --- specifically the group that makes the Fun3D CFD solver --- so it's fair to say I was working with the cutting edge at the time.

# 2D Case
For the 2D case, I selected a NASA GA(W)-1 airfoil (a.k.a. LS(1)-0417) with a 20% plain flap as my subject.
Note that is not a typo: this is a NASA airfoil, not an NACA airfoil.
This is the airfoil used by the Piper PA-38 Tomahawk wing, among many others.
The 20% plain flap means the flap is hinged through the geometry, and has a chord of 20% of the total airfoil chord.

{% include figure popup=true image_path="/assets/images/thesis/gaw1-geometry.png" alt="The GA(W)-1 airfoil." caption="The GA(W)-1 airfoil (from Ref. \[[1](#references)\])." %}

I chose this geometry because of the extensive wind tunnel test data that was collected in a wide range of conditions by Wentz \[[1](#references)\] at Wichita State University.
Below is a table of some key values for my selected case.

Quantity | Symbol | Value
---|---|---
Reynolds Number | $Re$ | 2.2 $\times$ 10<sup>6</sup>
Mach Number | $M$ | 0.13
Hinge axis x-coordinate | $x_h/c$ | 0.8
Hinge axis y-coordinate | $z_h/c$ | 0.03443

This test case is similar to an aircraft with a 2.4 ft chord flying at 86 knots at sea level
This condition is representative for an aircraft shortly after takeoff or on approach for landing --- the conditions in which frequent, large-angle deflections are required.

**Note:** The published GA(W)-1 airfoil coordinates are truncated or rounded. I used a custom program that I wrote to fit a spline to the coordinates and then move them within the rounding error to maximize the smoothness of the geometry. If you do not do this, the airfoil will be lumpy.
{: .notice--primary}


## USAF Stability and Control Datcom

The simplest among the solutions examined is the _USAF Stability and Control Datcom_ \[[2](#references)\].
The Datcom provides an emperical approach to estimating hinge moments.
It assumes low-speed, attached flow with linear hinge moment behavior (which we will show later is pretty far from reality).
This method relies on the airfoil geometry, specifically the airfoil thickness-to-chord ratio, the ratio of flap chord to airfoil chord, and three trailing edge angles.
My thesis includes all of the values used for this geometry, as well as an appendix with example calculations.

The Datcom provides estimates for the hinge moment derivatives with respect to angle of attack ($\alpha$) and deflection angle ($\delta$), denoted by $C_{h_\alpha}$ and $C_{h_\delta}$, respectively.
Using the Datcom calculations gives a hinge moment per degree angle of attack, $C_{h_\alpha}$ within 28% of the experimental values around $\alpha=0$, and a hinge moment per degree deflection, $C_{h_\delta}$ within 18% for a 5 degree deflection.
I took these values and estimated hinge moments using the linear equation below, where $C_{h_{00}}$ is the experimental hinge moment coefficient at $\delta=\alpha=0$.

$$
C_h = C_{h_{00}} + C_{h_\alpha}\alpha + C_{h_\delta}\delta
$$

This incorrectly assumes that hinge moment dependency on $\alpha$ and $\delta$ are independent, but it is our only option since that is what the Datcom assumes.
In reality, $C_{h_\delta}$ generally increases with $\alpha$ for this geometry.
Additionally, we must know (or estimate) at least one data point prior to extract hinge moments rather than hinge moment derivatives.
The result is the difficult-to-read plot below.

{% include figure popup=true image_path="/assets/images/thesis/gaw1-datcom-ch.png" alt="Hinge moment coefficients calculated using the USAF Stability and Control Datcom." caption="Hinge moment coefficients calculated using the USAF Stability and Control Datcom." %}

At small angles of attack (0-8 degrees) and small deflection angles (within 5-10 degrees), the results are decent.
Outside of those narrow conditions, I would say the results are unusable.
This should not be surprising based on the assumptions made by the Datcom.

**Bottom Line:** The Datcom does not give accurate results, especially for large angles of attack and deflection angles, and requires some *a priori* knowledge of at least one hinge moment value. 
{: .notice--primary}

## 2D Panel Method (XFOIL)

Panel method solvers are useful for fast, inexpensive solutions to fluid problems.
I used XFOIL, a panel method solver written by Drela \[[3](#references)\].
Without going into detail here (see my thesis for that), panel methods discretize the geometry into individual sections (i.e. panels) and calculate a solution based on potential flow theory, with each panel modeled as a vortex and a source.
XFOIL also includes an integral boundary layer calculation for the viscous terms.
For this work, I used XFOIL's built-in geometry manipulation tools to increase the number of panels on the geometry and simulate a control surface on the airfoil.
The upper and lower surfaces are deflected about the specified hinge location. 
This flap deflection is limited to simple flaps with no overhanging balance (like the geometry I am usin).
Note that multi-element sections are not possible in XFOIL, so hinge gaps cannot be modeled.
An example of the resulting geometry is shown below.

{% include figure popup=true image_path="/assets/images/thesis/gaw1-xfoil-deflection.png" alt="GA(W)-1 airfoil with XFOIL flap deflection." caption="GA(W)-1 airfoil with XFOIL flap deflection." %}

I ran XFOIL simulations for all angles of attack used in the wind tunnel reference, and for deflection angles ranging from -40 to 40 degrees in increments of 5 degrees.
I did this with a single script that opened XFOIL and entered all the necessary commands via XFOIL's command interface.
It was a kind of hacky way to go about it, but it gave results.
A complete run with 17 values of $\delta$ and six $\alpha$ sweeps took about 6 minutes to complete on hardware from 2013.
The results are shown below.

{% include figure popup=true image_path="/assets/images/thesis/gaw1-xfoil-ch.png" alt="GA(W)-1 hinge moments computed by XFOIL." caption="GA(W)-1 hinge moments computed by XFOIL." %}

The results from XFOIL were better than I expected.
At low angles of attack and low to moderate deflection angles, the hinge moment predictino is suitable for a conceptual design analysis.
It is not as good at larger deflection angles or moderate to high angles of attack.
This is due to flow separation, for which XFOIL is not well suited.

In addition to hinge moments, XFOIL also gives us lift, drag, and moment coefficients.
Unfortunately, these proved to be inaccurate for this geometry.

**Bottom Line:** XFOIL can be a useful tool for a conceptual design. It can show some hinge moment trends, but its inability to accurately resolve the boundary layer in the presence of flow separation limit the accuracy.
{: .notice--primary}


## 2D Navier-Stokes Solutions (Fun3D)

Fun3D is a finite volume CFD code developed by NASA.
I do not want to go into how CFD codes work (for that see my [thesis](/assets/docs/Simpson-2016-hinge-moment-cfd.pdf)).
Suffice to say that it can give very accurate numerical solutions to the Navier-Stokes equations.


### 2D Steady-state with Adjoint-Based Mesh Adaptation
Fun3D has a lot of features that most users will never need, but it is the only option for the few who do.
One such feature that was relatively new at the time was adjoint-based mesh adaptation.


#### A *Brief* Explanation

It would be easy to write way too much here, so I will explain it in very basic terms.
Essentially, the solver will pause before the solution converges, wiggle the nodes in the grid, and figure out which areas affect a provided cost function the most.
It then adds additional elements in those areas.
It sounds easy to use, but there are many parameters to change and it takes a great deal of experience to get it right.

One example of using adjoint-based mesh adaptation that I can give was for a supersonic diamond airfoil.
I provided a coarse initial mesh designed for inviscid flows and had the solver optimize the mesh to calculate the lift-to-drag ratio.
The result is the mesh in the figure below (colored by pressure).

{% include figure popup=true image_path="/assets/images/delaunay/diamond-airfoil.png" alt="An example of adjoint-based mesh adaptation for a 2D diamond airfoil." caption="An example of adjoint-based mesh adaptation for a 2D diamond airfoil." %}

Note how the grid is very coarse upstream (left) of the airfoil and very dense where shocks and expansions occur.


#### The GA(W)-1 Case

explain my work


# 3D Case
With an extra dimension, the problem becomes much more complex (and costly).
I chose a 45-degree swept horizontal stabilizer with an NACA 0012 airfoil and a 25% full-span elevator as the geometry.

# Conclusions
A number of meaningful conclusions can be drawn from the work presented in this thesis.
1. Empirical relations can be used to predict $C_{h_\delta}$ for a narrow range of conditions. When combined with some known value of $C_{h_0}$ , a crude estimate of hinge moment coefficient can be made.
2. XFOIL can accurately predict control surface hinge moments only for 2D configurations with small deflection angles and for small angles of attack. The speed of XFOIL makes it a useful tool during conceptual design for testing many different configurations in a short amount of time.
3. A steady-state Navier-Stokes solution is capable of accurately predicting control surface hinge moments in the presence of small, non-shedding regions of flow separations.
4. A time-accurate Navier-Stokes simulation is required for accurate prediction of control surface hinge moments when separated flow with vortex shedding is present. A time-accurate solution also provides a time history of the hinge moments rather than a single average value. This is important when considering the structure and mechanical devices required to support and actuate control surfaces.
5. The spatial resolution required for accurate hinge moment prediction is much finer than expected. I point out an example in my thesis that provides a node count of 18-30 million grid points for a full aircraft solution, which is comparable to the 3D horizontal stabilizer in my work. Fewer grid points are insufficient to accurately capture the control surface hinge moments.
6. The time step required to capture the unsteady behavior is very small. The unsteady behavior tends to decay to a steady-state solution if the time step is too large.
7. The initial condition of a time-accurate solution dramatically affects the number of iterations required for the long-term periodic behavior to develop. Restarting from a steady-state solution requires many more iterations for the periodic behavior to emerge. The most reliable method found to excite the unsteady behavior is to initialize the flow with the freestream velocity.
8. The final periodic behavior of a time-accurate solution depends slightly on the initial condition. The effect is small, but it does affect the time-averaged values. The exact relationship between initial condition and the final periodic behavior in this study is unknown.
9. Adjoint-based mesh adaptation can be used to automatically refine a CFD mesh to solve control surface hinge moments. The resulting hinge moment prediction is com-parable to the steady-state solution with a fine mesh.
10. The cost of an adaptive, steady-state, subsonic 2D case is not justified in terms of solution accuracy. A time-accurate solution can be computed using a globally-refined mesh with much greater accuracy for approximately the same cost as an adaptive solution. For transonic and supersonic cases, the adaptive solution will have the additional benefit of refining the mesh near shocks, which is generally difficult to accomplish through manual grid refinement.
11. CFD is a cost-effective method of predicting control surface hinge moments. Wind tunnel tests are more expensive than a CFD solution for the same conditions, and CFD generally offers a shorter development cycle because wind-tunnel availability is scarce. In a program that spends $1-3 million per day, the cost savings by shortening the development cycle is much more significant than a reduction in a single expenditure.

# References
1. W. H. Wentz, Jr., H. C. Seetharam, and K. A. Fiscko. <cite>Force and Pressure Tests of the GA(W)-1 Airfoil with a 20% Aileron and Pressure Tests with a 30% Fowler Flap.</cite> Contractor Report CR-2833, NASA, June 1977.
2. R. D. Finck et al. <cite>USAF Stability and Control Datcom</cite>. Report AFWAL-TR-83-3048, Flight Control Division, Air Force Flight Dynamics Laboratory, Wright-Patterson Air Force Base, Dayton, Ohio, April 1978.
3. Mark Drela. <cite>XFOIL: An Analysis and Design for Low Reynolds Number Airfoils.</cite> In Low
Reynolds Number Aerodynamics, volume 54 of Lecture Notes in Engineering. Springer-
Verlag, 1989.