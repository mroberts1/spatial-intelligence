---
title: Marble — A Multimodal World Model
source_type: article
url: https://www.worldlabs.ai/blog/marble-world-model
authors: World Labs
date: 2025
tags: #world-models #multimodal #generative-3d #spatial-intelligence
tier: primary
last_verified: 2026-05-09
superseded_by:
---

## Summary

Product article introducing Marble, World Labs' first-in-class generative multimodal world model. Marble instantiates the three pillars of world models: it is generative (creates 3D worlds), multimodal (accepts diverse inputs), and interactive (allows editing and expansion). The article showcases concrete capabilities: text-to-world, image-to-world, multi-image prompting, AI-native editing, Chisel (structure-style separation), world expansion, composition, and multi-format export.

## Core Capabilities

### Multimodal Inputs (Generative + Multimodal Pillars)

Marble accepts diverse input signals and lifts them into full 3D worlds:

**Text-to-world**: Single text prompt → explorable 3D world
- Example: "A detailed, lived-in hobbit kitchen filled with woven baskets and copper kettles, awash in calm pale-blue daylight"
- Marble invents all details not present in text

**Image-to-world**: Single image → 3D world
- Integrate with other AI image-generation tools: generate images, lift to 3D
- Works with photographs, AI-generated images

**Multi-image prompting**: Multiple views → unified 3D world
- Input: front, side, perspective views (from image generation or real photos)
- Marble reconstructs 3D that reconciles all views, seamlessly stitches transitions
- More precise control than single-image; less than 3D specification

**Video-to-world**: Real-world location from video/photo sequence → stylized 3D world
- Combine photogrammetry data with generative completion
- Example: Mount Everest photos → interpretive 3D world

### Interactive Editing (Interactive Pillar)

**AI-native editing tools**: Local and structural modifications
- Remove/change objects in specific regions
- Swap objects (door → fireplace)
- Change visual style (recolor, retexture)
- Restructure large parts (remove wall, extend room)

**Iterative refinement**: Feedback loop for creative process
- Generate → see result → edit → refine

### Chisel: Structure-Style Decoupling

**Key innovation**: Separate spatial structure from visual appearance

**Workflow**:
1. Create coarse 3D layout: boxes, planes, imported assets
2. Add text prompt describing style + details
3. Marble synthesizes: coarse geometry + style guidance → detailed 3D world

**Benefits**:
- Fine-grained control over scene structure
- Text prompt handles visual appearance
- Same structure, different styles (Scandinavian bedroom vs. Japanese vs. Mediterranean)
- Import existing 3D assets, have Marble restyle them

**Example**: Bedroom coarse layout (dimensions, furniture placement) + 6 different text prompts = 6 completely different visual styles, same spatial structure

### Scaling: Expansion and Composition

**World expansion**:
- Select region of generated world
- Marble creates more content, enlarges traversable area
- Improves artifact-prone regions (table backs, far corners) through re-generation at higher detail

**World composition**:
- Combine multiple generated worlds
- Manual control over layout and positioning
- Create very large spaces (train carriages, spacecraft, houses, haunted mansions)

## Export Options (Downstream Integration)

Marble worlds are persistent, editable 3D objects. Multiple export paths:

### Gaussian Splats
- Highest fidelity representation
- Render via [[Spark.js]] (browser, THREE.js, WebGL2)
- Interactive, real-time, mobile-compatible

### Triangle Meshes
- **Collider meshes**: Low-fidelity for physics simulation
- **High-quality meshes**: Visual fidelity matching splats
- Interoperate with industry tools (Blender, game engines, CAD)
- Support downstream simulation and rendering

### Video
- **Pixel-accurate rendering**: Full camera control
- **Video enhancement**: Detail addition, artifact removal, dynamic elements
- Examples in article generated directly from Marble

## Architectural Insights

### Multimodal Integration

**How Marble handles diverse modalities**:
1. Encode all inputs (text, images, video, 3D geometry) into shared spatial representation
2. Generate consistent 3D world respecting all constraints
3. Trade-off: single prompt maximizes creativity; multi-image maximizes control

**Implicit multimodal grounding**: Models learn to honor:
- Semantic content (text concepts)
- Visual appearance (image colors, textures)
- Geometric constraints (viewer position in multi-image)
- Spatial layout (coarse 3D in Chisel)

### The Generative Loop

Generation → Perception → Editing → Refinement

Unlike static 3D asset libraries, Marble enables:
- Infinite world variety (never see same space twice)
- User-guided creative process (iteration on edits)
- Rapid prototyping (minutes vs. weeks for traditional 3D design)

## Applications (Beyond Creative Tools)

**Mentioned in article**:
- Cinematic filmmaking (control camera, lighting, composition)
- Game design (rapid environment generation)
- Interactive worlds (immersive experiences)
- Robotics simulations (training environments, digital twins)
- Therapeutic environments

**Implied but not developed**:
- Product/industrial design (iterative 3D prototyping)
- Architecture (visualization before construction)
- Scientific visualization (computational results)

## Limitations (Implicit)

**What's not yet there**:

1. **Physics integration**: Exported meshes support simulation, but generation doesn't explicitly model physics
2. **Interactivity**: Worlds are static; future versions will support agent interaction
3. **Multi-agent scenarios**: No mention of collaborative editing or shared worlds
4. **Long-horizon consistency**: Worlds degrade with expansion; stitching challenges with composition

## Connection to Spatial Stack

Marble realizes two of three components:

- **3D Representations (code)** ✓ — Outputs explicit 3D (splats, meshes)
- **Neural Graphics (language)** ✓ — Generates from diverse inputs; multimodal synthesis
- **Simulation Engines (chips)** ✗ — Exported meshes can integrate with physics, but generation doesn't enforce physics consistency

## Marble Labs Ecosystem

**Community platform**:
- Case studies (cinema, robotics, therapy)
- Tutorials and documentation
- Showcase of real-world workflows
- Lower barrier to entry (LLMs generating THREE.js code + Marble/Spark)

## Future Directions (From Article)

**Explicit statement**: "Marble is just a step on our journey toward spatial intelligence."

**Next frontier**: **Interactivity**
- Humans and agents interact with generated worlds
- Unlock: simulation, robotics, embodied AI
- Implies world models that support persistent, mutable state

## Connected Concepts

- [[World Models]] — Marble exemplifies three-pillar architecture
- [[3D Representations]] — Outputs explicit, editable 3D
- [[3D Gaussian Splatting]] — Preferred export format
- [[Neural Graphics]] — Generative synthesis of 3D
- [[Embodied AI]] — Future application domain

## Quotes

> "Marble is the first of its kind - a next-generation world model making strides toward this vision."

> "Chisel decouples structure from style."

> "Marble is just a step on our journey toward spatial intelligence. Going forward, a key opportunity is **interactivity**."
