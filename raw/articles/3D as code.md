Text became the universal interface for software; 3D is becoming the universal interface for space. It’s the medium that allows humans and AI systems to generate, edit, simulate, and share worlds together.

![3D as code](https://www.worldlabs.ai/_next/image?url=%2Fimages%2F3d-as-code.jpg&w=2048&q=75)

There has never been a more exciting time for people and machines to work together. The interfaces connecting humans and machines are being redefined across the spectrum from abstract reasoning to physical tasks. Large language models spread quickly by building on text as a universal interface. Text is a shared medium, deeply embedded in our tools and habits; it can express ideas, give instructions, and communicate information between people and software. But what is the equivalent medium for describing and interacting with physical and virtual worlds?

**We believe that interface is 3D, enabling communication, collaboration, and creation for people and machines alike.** Three-dimensional representations allow people and machines to communicate about space, collaborate on designs, and create new environments. Over the past decades, a rich ecosystem of 3D tools and exchange formats has emerged to enable workflows that span ideation, modeling, simulation, presentation, and manufacturing.

This essay argues that there is an analogy between software code and 3D representations of the world. Building on the past decade of advances in neural graphics and simulation, a new generation of world models will accelerate existing spatial workflows and engender entirely new ones. Just as language models are revolutionizing software, world models will revolutionize how we shape and interact with virtual and physical worlds.

## 3D – the spatial “code”

We can understand the role of 3D representations in the spatial realm by comparing them to code. Code is a durable abstraction built to specify the underlying logic executed by processors. For decades, it has powered much of the modern world. Today, AI models are becoming highly proficient at reasoning about and generating code; this code is then executed on hardware that predates LLMs by a large margin. As interfaces, code and 3D share important structural similarities in why and how we use them.

**Between humans and machines:**

- Code serves as a powerful interface between humans and machines. When an AI system generates code, a human can inspect it, modify it, debug it and integrate it into larger systems. This enables composite workflows: programmers and AI coding agents can iteratively refine solutions together.
- 3D representations can play a similar role. When a world model generates a 3D scene, object, or environment, a human can open it in familiar tools, edit geometry, adjust constraints, rerun simulations, and correct errors. Here, too, composite workflows and pipelines can be built: designers and engineers can collaborate with generative world models.

**Between machines:**

- Code also functions as a machine-to-machine interface. AI-generated programs can plug into compilers, runtimes, APIs, and existing software infrastructure. Since code adheres to established abstractions, it interoperates with existing tools.
- Likewise, 3D outputs integrate with rendering engines, simulation systems, physics solvers, robotics stacks and CAD tools. When a world model produces structured 3D representations rather than pixels, it can participate in existing pipelines and interface with editing software and simulation engines.

In both cases, the key property is externalization of state into structured artifacts that other systems can consume.

Consider an alternative approach in the ‘code’ domain. Instead of having an LLM write a program, we could ask it to be the program. For example, we could prompt an LLM: “Sort the following list of one million numbers.” The model is capable of attempting to simulate this behavior entirely within its token stream, by ingesting the list and trying to re-emit it in sorted order.

But we rarely use LLMs this way, except as a party trick, and would not expect them to perfectly succeed at such a task. Why? Because code execution provides guarantees that raw inference does not, such as reproducible execution, human readability, and modular composability. Code can be stored, versioned, tested, and run independently of the models’ transient context window. It separates reasoning, representation, and execution: you think about the algorithm, you write down a program as text, and then you run it.

There is a direct parallel in spatial systems. The equivalent of asking an LLM to “be the program” would be discarding structured world representations and simulation engines, and instead relying purely on a black-box entanglement of state and measurement, such as models for action-conditioned pixel or state generation queried frame-by-frame. Such models may excel at their core task and be useful for a variety of applications, but they lack manipulable structure: their outputs cannot be inspected, edited, easily shared (e.g., shared experiences like multiplayer systems or shared intent and state between robots), or integrated into existing simulation and control systems.

## Neural graphics – a spatial “programming language”

If 3D is the spatial analogue of code, what plays the role of the programming language: precise, expressive and general enough to model the world? Over decades, a variety of 3D representations has emerged: meshes, voxels, point clouds, implicit fields, CAD formats, and so on. But creating rich, large-scale spaces, in particular for digital twins, has been difficult and hardware constrained. Traditional 3D engines were built around strict memory and compute limits, requiring simplified geometry and often hand-authored assets. In order to minimize memory use and bandwidth, pipelines were designed for asset reuse and compression. Data-driven approaches were too expensive and conflicted with fundamental assumptions driving design of these systems.

The explosive growth of hardware and software optimized for machine learning lifts these constraints. Modern GPUs, originally created for rendering triangles, proved to be extraordinarily useful for the large-scale matrix multiplication operations powering neural networks. New generations of GPUs are explicitly designed to also accommodate AI workloads, with large memory chips to fit models and datasets. At the same time, these GPUs still render graphics and run simulations exceptionally well.

In particular, this hardware trend enables new memory- and compute-hungry techniques like NeRF and Gaussian splatting to shine. We can now generate, store, and render world-scale representations that fit in memory, and recompute them dynamically if needed. Pipelines that once relied on static assets can become (partially or fully) generative. This enables higher-fidelity environments, greater diversity, and new application areas. Digital twins, for example, can move from being simplified and manually updated mockups to continuously updated high-precision mirrors of their physical counterparts, supporting monitoring, control and safety-critical workflows.

In this new stack, neural graphics functions like a programming language. It provides the expressive medium for describing and generating spatial structure, just as high-level languages describe computational structure.

## Simulation engines – the spatial “chips”

A world model becomes truly useful when it runs over time to enable interaction, persistence, and dynamics. If 3D is code, simulation engines are the chips running it.

Interactivity is not one single feature. It is a bundle of systems problems that simulation engines have addressed for decades: state management, physics, collision detection, lighting, synchronization, determinism, and replay.

At a minimum, long-horizon interactive experiences require persistence. The world must have an identity that survives beyond a single rendering pass. Actions leave traces, objects maintain state and a session can be resumed. This implies three core components:

- state management (what exists),
- update rules (how actions and physics/rules change it)
- and observations (how the current state is rendered into pixels or sensor outputs).

In principle, large diffusion or generative models could collapse all of this into an end-to-end mapping: (history + action) → next frame. Here, “state” lives only inside transient neural activations. This is a compelling research direction, with multiple models and projects investigating how far a “pixels all the way” approach can go.

But collapsing that stack introduces a fundamental tradeoff. When memory, dynamics, and rendering are entangled inside a single network, creation and consumption blur together. A physical interaction at run-time (kicking a ball) and a non-physical edit (deleting a wall) become the same type of input. Using our analogy above, editing the code becomes indistinguishable from executing the code. While convenient as an objective for training large-scale models, this conflation weakens guarantees around physical consistency, replayability and determinism.

The alternative is a factorized or hybrid runtime: learned world models generate and interpret structure, but make targeted use of external tooling resembling pieces of existing engines, as mediated by 3D interfaces and representations. Given the trajectory of LLM-based coding, it is likely these models will be able to build custom logic better suited to their use cases than off-the-shelf libraries and engines available today. But we predict that a distinction remains, between components for perception, generation, and reasoning, and pieces where “rules matter”.

**In a factorized system, 3D becomes a powerful interface for people and machines, exposing inputs and outputs that are controllable, repeatable, and interoperable.**

## 3D is an interface for people and machines

Given our analogy between 3D and code, let’s examine why 3D is a powerful medium for interfacing between people and machines to describe and interact with physical and virtual worlds.

**For machines.** Many software systems already operate spatially: simulators, robotics stacks, game engines, CAD tools, and GIS systems all interact through geometry, transforms, materials, trajectories, and constraints. If a world model produces outputs in the same structured language, it can plug directly into existing pipelines.

Just as importantly, machines increasingly need to communicate spatial intent to each other. A planning agent may mark a goal region, a safety monitor may mark forbidden areas, a perception module may annotate uncertain geometry, and a rendering module may request a new viewpoint: these are all spatial concepts.

If all spatial reasoning is entangled inside a single monolithic model, one way to implement this could be by sharing latent vectors. But that is a strong assumption that requires either a shared model or at least a shared latent space. In heterogeneous, modular environments that assumption breaks down. Even language is an inefficient interchange format for geometry and constraints; structured 3D is a more natural lingua franca.

Export capabilities matter just as much. When a world model can externalize its ‘thoughts’ to concrete representations (splats, meshes, videos), they become artifacts that can be inspected, validated, versioned, tested and reused – composable pipelines emerge.

**For people.** 3D interaction is also natural for humans. We spend all our waking lives navigating space: reaching, walking, manipulating, aligning, … Our mental models are structured around persistent objects and relationships: “the chair is under the table”, “the doorway connects these rooms”. When systems expose such explicit structures, they align with how we already think.

The contrast with purely image-based workflows is striking. In 2D animation, each frame must be redrawn, effectively reconstructing the world dozens of times per second. In 3D the world is built once, then moving the camera, changing lighting, animating objects. A single spatial edit propagates automatically across every rendered frame.

This separation between spatial 3D representation and rendering mirrors the separation between code and execution. You modify the source once and rerun it instead of rewriting every output from scratch.

## Toward the future

If 3D plays a role analogous to code as an interface between people and machines, then the trajectory is clear: worlds become “programmable”, a medium that can be generated, edited, composed, and shared by people and machines alike.

This is the direction we are building toward at World Labs:

- [Marble](https://www.worldlabs.ai/blog/marble-world-model) is a multimodal world model designed to reconstruct, generate and simulate 3D worlds. It can create persistent, navigable worlds from text, images, video or coarse 3D layouts. These worlds can be edited, expanded, exported (as Gaussian splats, meshes, or video) and integrated into downstream tools.
- The 3D conditioning interface of [Marble](https://marble.worldlabs.ai/), an experimental feature called Chisel, advances the idea of 3D as a coarse control layer. It allows creators to block out structure using walls, planes, volumes, and imported assets, then provides these as input to our model to generate rich and detailed visuals on top. Separating layout and style gives users explicit control over composition and appearance.
- [RTFM](https://www.worldlabs.ai/blog/rtfm) and [Spark](https://sparkjs.dev/) explore the rendering layer. RTFM experiments with ‘learned rendering’, producing complex visual effects such as reflections and shadows from simple structural inputs. Spark is a high-performance Gaussian splatting renderer that integrates with WebGL, bringing neural graphics into real-time web environments.

This landscape is evolving rapidly. World models will increasingly participate in hybrid stacks: generating structured worlds (the “code”), expressed through neural graphics (the “language”), executed inside simulation engines (the “chips”). This is a shift toward programmable, data-driven spatial systems capable of supporting realistic environments, digital twins, robotics, training, design, and entirely new categories of applications. The core premise remains stable: reliable communication and collaboration between humans, agents and software, requires an interface that is precise, compact, inspectable, and manipulable.

That interface is 3D.