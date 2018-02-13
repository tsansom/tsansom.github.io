---
layout: post
comments: true
title: "Model-Level Hydrodynamic Particle Tracking"
subtitle: "Supporting oyster reef restoration projects in the Trinity-San Jacinto and Mission-Aransas Estuaries"
data: 2017-12-15 09:48:48 -0600
categories: particle-tracking TxBLEND
---

## Introduction
To support oyster reef restoration efforts and other potential projects (like oil spill response), I developed model-level particle tracking functionality for TxBLEND, a two-dimensional hydrodynamic and salinity transport model used by the Texas Water Development Board (TWDB) to simulate water circulation and salinity patterns in Texas estuaries. TxBLEND is a vertically-averaged, finite-element model which employs and unstructured, triangular element grid with linear basis functions. The model is designed specifically for Texas estuaries, which are generally very shallow and wide bodies of water.

Particle tracking studies are often conducted using velocity from model outputs, which typically have time scales of 30 minutes to one hour. The novelty of implementing particle tracking at the model-level is that the calculation of velocity, and subsequently position, happen on the model time-step. The typical time-step for TxBLEND model runs is two or three minutes. This allows me to capture smaller perturbations of the velocity field that would otherwise be lost to averaging when using hourly model output for example.

## Interpolating Velocity in a Triangular Element
Figure 1 shows the representation of a linear triangular element, $l$, as it would appear in the TxBLEND model gird. The points $i$, $j$, and $k$ represent the nodes of the element $l$. All variable calculations (*i.e.* velocity, salinity, water level, *etc.*) occur at the nodes, and as a result, variable interpolations inside the element are straight-forward.

<p align="center">
<img src="{{ site.url }}/images/element_schematic.png" width="500px">
</p>
<p align="center"><em><strong>Figure 1. </strong></em>Schematic of a TxBLEND Element $l$</p>

To interpolate the velocity at point $P$ inside of a linear triangular element (see Figure 1), basis functions must first be calculated for each of the three nodes, or vertices, of the triangular with respect to the point $P$. The basis functions described herein are used as weighting functions and must fulfill the requirement that they are unity, or one, at the node for which they are calculated and zero at all other nodes. Also, in the case of linear triangles, they must describe a plane over the triangular element. The general expression for the $i$th basis function takes the form

$$\phi_i = a_i + b_ix + c_iy$$

where $a_i$, $b_i$, and $c_i$ are constants identified with the $i$th basis function, while $x$ and $y$ are the horizontal coordinates (*i.e.* longitude and latitude) of the point $P$. With this information, the $i$th basis function, $\phi_i$, can be written as



$$
  \begin{tabular}{ccc}
  1 & 5 & 8 \\
  0 & 2 & 4 \\
  3 & 3 & -8
  \end{tabular}
$$
