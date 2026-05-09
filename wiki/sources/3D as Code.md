---
title: 3D as Code
source_type: article
url: https://www.worldlabs.ai/blog/3d-as-code
authors: World Labs
date: 2024
tags: #spatial-intelligence #world-models #3d-representations #neural-graphics
tier: primary
last_verified: 2026-05-09
superseded_by:
---



## Summary

This essay from World Labs establishes a foundational analogy: 3D representations are to spatial systems what code is to software systems. Just as code provides a durable, inspectable, composable interface between humans and machines, 3D representations enable humans and AI to collaborate on generating, editing, and simulating worlds.

## Key Concepts

### 3D as Spatial Code
- 3D representations serve as the interface between humans and machines for spatial reasoning
- Enables composite workflows where designers and world models iterate together
- Allows machine-to-machine spatial communication (rendering engines, simulation systems, robotics stacks)

### Neural Graphics as Programming Language
- NeRF and Gaussian splatting enable generation and rendering of world-scale representations
- GPUs enable both neural networks and graphics workloads simultaneously
- Generative pipelines can now create high-fidelity digital environments instead of hand-authored assets

### Simulation Engines as Spatial Chips
- Simulation engines (like physics, collision, lighting systems) are the "hardware" executing world models
- Critical distinction: structured 3D outputs with simulation (vs. black-box pixel-based generation)
- Enables persistence, determinism, replayability—guarantees that raw neural inference alone cannot provide

### The Importance of Externalization
- Structured outputs can be inspected, validated, versioned, tested, and reused
- Composable pipelines emerge when intermediate representations are explicit
- Contrast: pixel-generation models that entangle all of state, dynamics, and rendering cannot be extended

## Connected Concepts

- [[World Models]]
- [[Embodied AI]]
- [[3D Graphics]]
- [[Simulation Systems]]

## Quotes

> "If 3D is the spatial analogue of code, then the trajectory is clear: worlds become 'programmable', a medium that can be generated, edited, composed, and shared by people and machines alike."

> "In a factorized system, 3D becomes a powerful interface for people and machines, exposing inputs and outputs that are controllable, repeatable, and interoperable."
