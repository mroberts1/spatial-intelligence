---
title: Streaming 3DGS Worlds on the Web
source_type: article
url: https://www.worldlabs.ai/blog/streaming-3dgs-worlds-on-the-web
authors: World Labs (Spark team)
date: 2024
tags: #neural-graphics #3d-gaussian-splatting #rendering #web-graphics
tier: primary
last_verified: 2026-05-09
superseded_by:
---

## Summary

Technical deep dive into Spark 2.0, World Labs' open-source 3D Gaussian Splatting (3DGS) renderer for the web. The article describes how Spark enables rendering and streaming massive 3D scenes (40M+ splats) on consumer devices including mobile phones, VR headsets, and web browsers.

## Technical Overview

### 3D Gaussian Splatting (3DGS)

**Representation**: Millions of semi-transparent, colored ellipsoids (splats) that blend together to form surfaces and details.

**Advantages over meshes**:
- Natural representation for neural graphics outputs (NeRF, diffusion models)
- Efficient rendering via splatting (2D projection + Gaussian opacity computation)
- Supports dynamic animation and real-time editing

**Rendering approach**: Painter's algorithm
1. Sort splats back-to-front from camera viewpoint
2. Project each splat to 2D ellipse on screen
3. Compute opacity from Gaussian profile
4. Blend colors using alpha compositing

### Spark System Design

**Three-step rendering pipeline**:

1. **Generate global splat list**: Collect splats from all 3DGS objects, transform to common coordinate system, apply user-programmable GPU pipeline (recoloring, opacity adjustment, animation, etc.)

2. **Sort splats**: Compute distances on GPU, read back to CPU, sort via radix sort in Web Worker thread (WebGL2 limitations prevent GPU sorting)

3. **Render**: Single instanced draw call, per-pixel Gaussian opacity computation, alpha blending

### Core Innovation: Level-of-Detail (LoD) System

**Problem**: 
- Large 3D scans contain tens of millions of splats
- Consumer devices can only render 1-5M splats at interactive frame rates
- Streaming 100MB-1GB of data over internet is slow

**Solution: LoD Splat Tree**

A continuous (not discrete) LoD system where splats exist in a hierarchy:
- Each interior node is a lower-resolution version created by merging child splats
- Tree root is single splat representing entire scene
- Render a "tree cut" of splats optimized for current viewpoint

**LoD Tree Traversal Algorithm**:

Uses priority queue to select best splats within a budget (500K-2.5M splats depending on device):

1. Start from root splat, compute screen dimension, insert into priority queue
2. Pop largest splat from queue
3. If smaller than 1 pixel or leaf node → add to output set
4. If expanding to children would exceed budget → move remaining queue splats to output and stop
5. Otherwise expand children, compute screen dimensions, insert into queue

This runs in O(B) time where B is splat budget, independent of total scene size.

**Foveated Rendering**: Bias splat budget toward viewer's gaze direction using foveation scale parameters.

### Tree Generation Algorithms

**Tiny-LoD** (default, on-demand):
- Fast, low memory
- Grid-based: divides space into cubes of varying sizes, merges splats in same cube
- Uses sorting+chunking technique from computational genomics for memory efficiency
- Variable merge ratio (default 1.3) produces smoother detail transitions than power-of-2

**Bhatt-LoD** (offline):
- Higher quality
- Iterative pairing based on Bhattacharyya distance (statistical shape similarity) + color similarity
- Produces trees 30-40% larger than input splat count
- Best for critical applications

### Progressive Streaming: .RAD File Format

**Problem with existing formats**:
- **.PLY**: Row-ordered, uncompressed, 2.3GB for 10M splats, but naturally streamable
- **.SPZ**: Column-ordered, compressed, 200-250MB for 10M splats, but requires entire file before any splat is complete

**Solution: .RAD format** (RADiance fields):

Structure:
- Header: `RAD0` + metadata size + JSON metadata (chunk offsets)
- Multiple 64K splat chunks, each with header + compressed column-order data
- Gzip compression (good for column-oriented data)
- Random-access chunk fetching

**Spatial Chunking Strategy**:
- Partition LoD tree across 4 dimensions (3 spatial + LoD level)
- First chunk: coarsest 64K splats from root downward
- Subsequent chunks: subdivide space recursively into regions
- Coarse version visible immediately; details refine as chunks load
- Parallel Web Worker fetching prioritizes chunks for current viewpoint

### Virtual Memory for Splats

**Concept**: Adapt OS virtual memory to GPU splat management

**Implementation**:
- Fixed GPU pool: 16M splats
- GPU page table: maps 64K splat pages from .RAD file chunks
- Least-recently-used (LRU) eviction when pool full
- Supports multiple .RAD files sharing same page table

**Result**: Access to infinite splat worlds with fixed GPU memory allocation.

### Applications

**Spark 2.0 enables**:
- **Marble integration**: Generate explorable 3D worlds from text/images
- **Web streaming**: 40M+ splat scenes on mobile/VR/desktop
- **Multiplayer**: Shared explorable worlds (Starspeed example: 100M splat sci-fi environments)
- **Artist tools**: Dormant Memories (interactive scans), HoloLab demonstrations

## Technical Takeaways

1. **Continuous LoD better than discrete**: Avoids "popping" artifacts when switching detail levels

2. **Spatial partitioning enables coarse-to-fine loading**: First chunk loads immediately; subsequent chunks refine visible regions

3. **GPU-CPU hybrid**: GPU computes distances, CPU sorts (due to WebGL2 limitations); both can run in parallel

4. **Adapt OS concepts to graphics**: Virtual memory model applies naturally to splat streaming

5. **Scalability**: Algorithms scale with device capability (splat budget) not scene size

## Connected Concepts

- [[Neural Graphics]] — Spark implements NeRF/diffusion model outputs at scale
- [[3D Representations]] — Splats as explicit, editable spatial data
- [[World Models]] — Marble generates 3DGS worlds; Spark renders them
- [[Spatial Intelligence]] — High-fidelity 3D worlds enable embodied AI perception

## Quotes

> "3D Gaussian Splatting made it possible to render these scans interactively on consumer devices."

> "With Spark 2.0 we've added a Level-of-Detail system that can stream and render huge 3DGS worlds on any device."

> "Spark allows us to easily create huge composite worlds by simply adding 3DGS LoD objects anywhere in space."

## Sources

- Open source: https://github.com/sparkjsdev/spark/
- Spark documentation: https://sparkjs.dev/
- Interactive examples: https://sparkjs.dev/examples/
