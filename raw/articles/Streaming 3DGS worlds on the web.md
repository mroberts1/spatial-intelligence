[Spark](https://sparkjs.dev/) is a dynamic 3D Gaussian Splatting (3DGS) renderer built for the web. It integrates with the web's most popular 3D framework [THREE.js](https://threejs.org/) and uses [WebGL2](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API) to run on any device with a web browser, including desktop, iOS, Android, and [VR](https://immersiveweb.dev/).

Launched last year, Spark brought with it many features that no other renderer had: the ability to render multiple 3DGS objects in the same scene, real-time editing and relighting, and a shader graph system that allows users to create fully dynamic splat-based effects and animations.

With Spark 2.0 we've added a Level-of-Detail (LoD) system that can stream and render huge 3DGS worlds on any device. As you move around the world, Spark automatically optimizes the 3DGS detail level for the viewpoint and streams the necessary data over the Internet.

In this post we'll do a deep dive into the technical details that make this possible, and learn some computer graphics and systems engineering along the way. Already familiar with 3DGS? [Skip introduction and jump to Spark 2.0 LoD method.](#lod-splat-tree)

## 3D Gaussian Splatting

Scanning the world and turning it into 3D data on a computer, known as [photogrammetry](https://en.wikipedia.org/wiki/Photogrammetry), has been improving quickly with the advent of [Machine Learning techniques](https://www.matthewtancik.com/nerf). The introduction of [3D Gaussian Splatting](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/) made it possible to render these scans interactively on consumer devices.

In traditional 3D graphics we represent the surfaces of objects using texture-mapped triangles. But with 3DGS we instead use millions of semi-transparent, colored ellipsoids known as **splats** that blend together to form all the surfaces and tiny details.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/tri-vs-splat.mp4" aria-label="Comparison of triangle meshes and Gaussian splats" controls=""></video>

Click to view in 3D: Rendering the same object using texture-mapped triangle meshes (left) and Gaussian splats (right).

Each splat has the following properties: its center location in space , the radii of the ellipsoid's XYZ axes , a 3D orientation , RGB color , and its opacity . If we ignore the orientation and our splat is at the origin, we can describe its 3D "density" with the function , which has a [Gaussian profile](https://en.wikipedia.org/wiki/Gaussian_function).

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/3dgs-props.mp4" aria-label="Demonstration of 3D Gaussian Splat properties" controls=""></video>

Click to interact in 3D: Adjust the properties that control a 3D Gaussian Splat's appearance: its center, XYZ scales, rotation, opacity, and color.

The most common way to render these splats to the screen is the [painter's algorithm](https://en.wikipedia.org/wiki/Painter%27s_algorithm) where we sort the splats back-to-front, then draw them on top of each other, blending the semi-transparent areas together using the ["over" operator](https://en.wikipedia.org/wiki/Alpha_compositing). Each 3D ellipsoid is projected to the image plane as an approximate 2D ellipse, and for each pixel in the ellipse we compute the opacity from its Gaussian, and finally blend its color with the previously drawn splats.

## Spark's origins

Spark started out as an internal 3DGS renderer developed by World Labs because existing web renderers had shortcomings that would limit us in the future. For example, they could only correctly render one 3DGS object at a time, they couldn't dynamically animate the splats (colloquially known as "4DGS"), they were based on less popular 3D frameworks, or used WebGPU which [isn't available on all devices](https://web3dsurvey.com/webgpu). This renderer was featured in our 2024 [Large World Model research preview](https://www.worldlabs.ai/blog/generating-worlds) and early world showcase [Lofi Worlds](https://lofiworlds.ai/marble/).

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/lofi-worlds.mp4" aria-label="Lofi Worlds - a relaxing journey through 3D worlds" controls=""></video>

Click to experience in 3D: Lofi Worlds - a relaxing journey through atmospheric 3D worlds created by Marble. Best experienced in VR on Quest 3 or Apple Vision Pro.

But we wanted **anyone** to be able to build interactive 3DGS web experiences, so we decided to take our learnings and build a general-purpose, open-source 3DGS renderer. We chose to build it on top of the web's most popular 3D framework [THREE.js](https://threejs.org/), capable of making complex 3D experiences in the browser, even with vibe coding. We also targeted [WebGL2](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API), the only 3D web API that is almost guaranteed to run on [every device today](https://web3dsurvey.com/webgl2).

Spark was developed alongside [Marble](https://marble.worldlabs.ai/), which lets you create 3D worlds in the web browser using our [multi-modal world model](https://www.worldlabs.ai/blog/marble-world-model). This served as a concrete use case that helped shape Spark into a flexible, user-programmable 3DGS processing engine, and also as a demonstration for what could be built with Spark.

## Spark system design

We wanted to be able to construct huge worlds by joining 3DGS objects together, and to do this correctly you have to sort all the splats across different objects in a unified back-to-front order. Without this, objects only sorted locally look like they are "pasted over" each other rather than coexisting in the same 3D space.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/bad-sorting.mp4" aria-label="Comparison of local vs. global splat sorting" controls=""></video>

Click to experience in 3D: Comparison of incorrect local-only splat sorting vs. globally-sorted splats

3DGS is also a [rapidly evolving field](https://mrnerf.github.io/awesome-3D-gaussian-splatting/), and we wanted users to be able to experiment with new techniques for relighting, animation, real-time editing, or other creative/interactive effects. We wanted Spark to be flexible enough to handle these use cases without needing to continually modify Spark's core systems.

Spark addresses these using a three-step algorithm:

1. Generate one global list of splats from across all 3DGS objects, transformed into the same space
2. Sort this global list back-to-front for the current viewpoint
3. Render the splats in that order

### 1\. Generating a global list of splats

Step 1 iterates over each 3DGS object, contributing its splats to a global list. Because each 3DGS object can be independently positioned and oriented in space, we must at a minimum transform the splats into the same coordinate system.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/render-steps.mp4" aria-label="Spark 3DGS rendering steps" controls=""></video>

Click to experience in 3D: Spark renders dynamic, correctly-sorted 3D Gaussian Splats with a 3-step process: 1) Generate a global list of animated splats for the current frame, 2) Sort the splats back-to-front, 3) Render the splats back-to-front.

Spark takes this opportunity to run a customizable data pipeline on each splat on the GPU. This opens the door to all sorts of possibilities: recoloring the splats, adjusting opacity, SDF-based clipping, animated transitions, interpolating between scanned 4DGS frames, and more. This pipeline is user-programmable, using either [GLSL](https://sparkjs.dev/examples/#glsl) or by connecting [nodes into a computation graph](https://sparkjs.dev/docs/dyno-overview/) (akin to shader graph systems in 3D engines). Each 3DGS object can run its own pipeline, mixing and matching effects within a scene.

### 2\. Sorting the splats

Although the sorting in step 2 is possible to do on a modern GPU, the programming model provided by WebGL2 makes it prohibitive. Instead, Spark calculates splat distances on the GPU and reads back this list to the CPU. These distances are then sorted using a two-pass radix sort in a background [Web Worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) thread. Spark generalizes this further, allowing you to do this with [multiple viewpoints](https://sparkjs.dev/examples/#multiple-viewpoints) simultaneously, each with its own sort order.

### 3\. Rendering the splats

Once we have a global list of splats and their ordering, in step 3 we do a single [instanced draw call](https://webgl2fundamentals.org/webgl/lessons/webgl-instanced-drawing.html) that draws all the splats in one go. Each splat is rendered as an oriented quad (technically two triangles) enclosing the splat's projected 2D ellipse. The four vertices are calculated using a custom [vertex shader](https://developer.mozilla.org/en-US/docs/Web/API/WebGLShader), then the [fragment shader](https://webgl2fundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html) computes the opacity for each pixel by evaluating the Gaussian profile, and finally the GPU hardware blends the color into the frame buffer.

## Scaling up to huge worlds

Since Spark's initial launch, 3DGS has been growing in popularity but also in scale. 3D scans today often exceed tens of millions of splats, and World Labs is generating [bigger and better worlds](https://www.worldlabs.ai/blog/bigger-better-worlds) that can be [expanded and composed](https://www.worldlabs.ai/blog/marble-world-model#expand) into [huge](https://wlt-ai-cdn.art/spark-2.0/260413/explore.html?url=https://wlt-ai-cdn.art/spark-2.0/rad/cave-lod.rad&startPos=28.5,0.2,36.5&startQuat=0.08,0.45,-0.04,0.88&moveSpeed=1.5&splatCount=73%20M&showHelp=true&showPaging=true&splatLimit=1500000%22) [labyrinths](https://wlt-ai-cdn.art/spark-2.0/260413/explore.html?url=https://wlt-ai-cdn.art/spark-2.0/rad/ruins-lod.rad&startPos=-55,20,-38.3&startQuat=-0.01,-0.7,-0.01,0.72&moveSpeed=5&splatCount=26%20M&showHelp=true&showPaging=true&splatLimit=1500000).

However, most consumer devices can only render 1-5M splats at interactive frame rates. Large scenes need to download 100+ MB or even 1+ GB 3DGS data before displaying it. Mobile device browsers have a cap on GPU memory utilization, limiting the size of worlds that can be shared reliably online.

### Spark 2.0

The new Spark 2.0 is a complete solution for **preparing, streaming, and [rendering huge 3DGS scenes](https://wlt-ai-cdn.art/spark-2.0/260413/explore.html?url=https://wlt-ai-cdn.art/spark-2.0/rad/coit-40m-sh1-lod.rad&orient=1,0,0,0&scale=10&startPos=-0.858,2.203,-1.128&startQuat=-0.043,-0.909,-0.097,0.402&moveSpeed=1&splatCount=40%20M&credit=Vincent%20Woo&creditLink=https%3A%2F%2Fvincentwoo.com%2F&showHelp=true&showPaging=true&backgroundColor=cafefe&lodSplatScale=1.5&highDpi=true) on the web on every device**.

It employs three graphics and systems techniques to address the scaling challenges:

- **Level-of-Detail:** Preparing lower-resolution versions of the splats and calculating which subset of splats to render for the camera viewpoint. By rendering fewer splats when they're too far away to make out the details, we can improve rendering performance.
- **Progressive Streaming:** Loading in 3DGS details in a coarse-to-fine manner as data is downloaded, prioritizing data that will best resolve details depending on where the camera is looking.
- **Virtual Memory:** Allocate a fixed GPU memory pool for a **splat page table** that automatically swaps in and out chunks of 3DGS data as needed depending on where we are in the scene, giving access to huge pools of splats across multiple 3DGS objects fetched over the Internet.

## Level-of-Detail

A common technique in computer graphics to handle large 3D scenes is to use a [Level-of-Detail (LoD)](https://en.wikipedia.org/wiki/Level_of_detail_\(computer_graphics\)) system that automatically adjusts the amount of detail rendered depending on the distance to the viewer. The detail can also be lowered to get a higher frame rate, or increased when the viewer stops moving.

A classic example of LoD is [Mip-mapping](https://en.wikipedia.org/wiki/Mipmap) where a texture is downsampled into a pyramid of smaller textures, each texture level half the size of the previous, with a single pixel at the top. This allows us to quickly sample a texture pixel roughly the same size as a screen pixel at any distance from the camera.

A well-known modern LoD system for triangle meshes is Unreal Engine's [Nanite](https://en.wikipedia.org/wiki/Unreal_Engine_5#Nanite) which creates a hierarchy of triangle clusters at varying levels of detail and [selects a subset of clusters](https://advances.realtimerendering.com/s2021/Karis_Nanite_SIGGRAPH_Advances_2021_final.pdf) to render at just the right level of detail depending on the viewpoint.

![Left: Texture LoD using Mip-mapping. Right: Mesh LoD with Nanite](https://www.worldlabs.ai/images/mipmap-nanite.png)

Left: Texture LoD using Mip-mapping with a pyramid of lower-resolution textures. Right: Mesh LoD with Nanite with a hierarchy of triangle clusters, allowing selection of meshes with appropriate detail.

Different LoD approaches can exist on a spectrum from [**discrete** to **continuous** LoD methods](https://en.wikipedia.org/wiki/Level_of_detail_\(computer_graphics\)#Well_known_approaches). Discrete LoD involves creating several different versions of the splats from small to large splat counts, then swapping between the versions depending on the distance between their approximate bounds and the camera.

Although Spark's original system design supports this model out of the box, this approach results in "popping" artifacts when moving around and suddenly switching between versions, and has visible boundaries when grouping splats into tiles.

### Level-of-Detail Gaussian splat tree

Spark's LoD design is a **continuous LoD** method, where all splats exist in a hierarchy, an **LoD splat tree**. Spark individually select splats along a boundary cut of the tree that optimizes the splat detail within the viewport.

Each internal tree node is a lower-resolution version of its children, formed by merging the splats into a new one that approximates the shape and color of the child splats. This continues all the way up to the root of the tree, a single large splat that has the aggregate shape and color of all the splats in the object.

![3DGS LoD tree](https://www.worldlabs.ai/images/splat-tree.png)

Left: LoD splat tree generated by merging input leaf splats into larger interior node splats up to a single root splat. Right: Spark computes a tree cut that selects splats to render depending on the current viewpoint.

Using this LoD splat tree, Spark computes "slices" through it that selects the best set of splats to render for the current viewport. By setting a maximum splat budget (500K - 2.5M splats depending on device type) we ensure that we only have to render a constant number of splats each frame, resulting in a steady, high frame rate. By adjusting up or down we can trade off between frame rate and splat detail.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/lod-worlds.mp4" aria-label="Spark adjusting 3DGS detail for the viewpoint" controls=""></video>

Click to experience in 3D: Spark adjusts the 3DGS detail level continuously depending on the camera view frustum, and streams chunks of data over the Internet as needed (shown with green flashes).

### LoD splat tree traversal

Spark computes the best subset of LoD splats to render for the viewpoint by traversing the splat tree in time, where is the rendered splat budget, independent of the total splat tree size. No matter how many splats there are globally in the scene, Spark updates the fine-grained splat LoD in roughly uniform time. The tree traversal algorithm makes use of a priority queue:

1. Starting from the LoD splat root , compute its screen dimension , and insert into the priority queue.
2. Pop the maximum sized LoD splat from the priority queue. If is smaller than 1 pixel or is a leaf node, add it to the output set, and repeat step 2.
3. If replacing the LoD splat by its children would exceed the splat budget , move all the remaining queue splats to the output set and stop the algorithm.
4. Otherwise, for each child of parent , compute the screen dimension and insert into the queue. Finally, repeat step 2.
<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/lod-traversal.mp4" aria-label="Traversing LoD splat tree to compute details" controls=""></video>

Click to interact in 3D: Spark traverses the LoD splat tree from coarse to fine levels to compute a tree cut that balances splat pixel sizes within a budget of N splats.

Spark implements this algorithm in [Rust](https://rust-lang.org/) compiled to [WebAssembly (Wasm)](https://developer.mozilla.org/en-US/docs/WebAssembly) to run efficiently in a background Web Worker so that LoD updates run asynchronously and don't impact the main render loop.

### Composite LoD splat scenes

Spark generalizes this algorithm further by traversing multiple instances of LoD splat trees simultaneously. Instead of starting with a single root, for each 3DGS object we insert its screen dimension and splat node into the initial priority queue. The rest of the algorithm proceeds as before, selecting what detail to refine across all 3DGS objects in the scene simultaneously.

This allows us to easily create huge composite worlds by simply adding 3DGS LoD objects anywhere in space, and Spark will compute the best global subset of all LoD splats to render each frame.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/multi-lod.mp4" aria-label="Traversing multiple LoD splat trees simultaneously" controls=""></video>

Click to experience in 3D: Composite worlds created from multiple 3DGS objects, each with an LoD splat tree. Spark traverses them simultaneously to produce a unified level-of-detail rendering.

### Foveated splats

Spark implements fixed [foveated rendering](https://en.wikipedia.org/wiki/Foveated_imaging) that biases the LoD splat budget toward the center of the view direction. This increases the level of detail in the viewing direction, allocating fewer, larger LoD splats to the sides and behind the viewer.

Since Spark computes the LoD splat screen dimension as part of the LoD tree traversal to refine the coarsest splats first, we can adjust to precisely control the level of detail throughout the world. If we double the splat size to , it will be inserted higher in the priority queue and resolve into 2x finer detail. Conversely, if we halve the splat size to it will be lower in the priority queue and resolve into 2x coarser splats.

We call this scaling factor the foveation scale that varies as a function of splat view direction , adjusting the calculation from to , resulting in a final relative splat size of .

Spark uses four LoD parameters to control the view-dependent level-of-detail:

- **coneFov0**: Angle of a cone around the view direction with and full resolution.
- **coneFov**: Angle of a larger cone that will have its detail reduced by **coneFoveate**.
- **coneFoveate**: Foveation scale at the edge of **coneFov**, smoothly interpolating as the angle goes from **coneFov** to **coneFov0**. Setting this to results in 10x larger splats.
- **behindFoveate**: Foveation scale behind the camera, varying smoothly over from **coneFov** to **180** degrees.
<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/foveation.mp4" aria-label="LoD foveation focuses splat detail for viewpoint" controls=""></video>

Click to interact in 3D: Spark's LoD system uses foveated rendering to focus splat detail where the camera is looking.

## Generating LoD trees

Using its command-line tool [`build-lod`](https://sparkjs.dev/docs/lod-getting-started/), Spark can convert any 3DGS file into an LoD splat tree encoded in a new streamable.RAD file. Alternatively, it can load any file in the web browser and create the LoD tree on-demand using a background Web Worker. Both run the same Rust code, which compiles to both Wasm and native environments.

Spark 2.0 includes two algorithms for creating LoD splat trees: 1) a quick and compact algorithm called **Tiny-LoD** used by default when run on the web, and 2) a higher-quality **Bhatt-LoD** used by default on the command-line.

Both methods are "training-free" and don't require any reference images or other inputs, instead operating directly on the 3DGS data. Other tree generation methods are also possible, for example [NanoGS](https://saliteta.github.io/NanoGS/) could be used to generate a LoD tree.

### Tiny-LoD algorithm

This method is meant to be used "on-demand" where the priority is producing a reasonable LoD splat tree quickly without using too much memory. It is based on an older voxel octree algorithm we called **Quick-LoD** but uses a memory optimization technique from computational genomics.

We divide space into a grid of cubes, and allow the step size of this grid to vary by , where is a signed integer corresponding the LoD tree level and is the ratio between successive grid sizes. When this corresponds to a regular octree, a common 3D data structure used in graphics. Each input splat is inserted into the grid at a where its size is just less than the cube grid size . We place each splat in the cube that contains its center.

We process one at a time, starting from the lowest containing the smallest splats. For every cube grid at the current that contains more than one splat, merge those splats into a new splat. Create a corresponding LoD tree node with the previous splats as children. Once all cubes have been processed, increment and repeat the process. This continues until all input splats have been merged into a single root splat.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/tiny-lod.mp4" aria-label="Interactive demonstration of Tiny-LoD algorithm" controls=""></video>

Click to experience in 3D: Tiny-LoD generates an LoD splat tree by merging splats that fall within grid cubes, iterating up a hierarchy with larger grids until all splats are merged into a single root.

In computational genomics it's common to have to find data that contains the same substring of letters. This can be done easily using a hash map, but when the key space is large and values are variable sized arrays it becomes memory and cache inefficient due to data structure overhead. Tiny-LoD adopts a trick from computational genomics to do this efficiently:

1. Instead of a hash map, create one contiguous array of for each splat we insert, where .
2. Sort this array by , after which all splats with the same will be adjacent in the array. Although sorting is , this is often faster than hashing because it has better cache locality.
3. Iterate over grid cells by chunking the array beginning-to-end into groups of splats with the same .
4. Merge these groups of splats into new LoD splats.
5. Repeat step 1 at the next larger .

Although may seem like a natural pick, where splats are recursively merged up a regular power-of-2 octree, in practice we've found produces more pleasant results. With a non-integer the irregular grid boundaries break up patterns as we move up levels and reduces large changes in splat detail. Smaller bases such as produce even smoother detail transitions but also results in a taller tree that is slower to traverse and update. By default Tiny-LoD uses to balance these factors.

### Bhatt-LoD algorithm

This method is named after [Bhattacharyya](https://en.wikipedia.org/wiki/Anil_Kumar_Bhattacharyya) and the [Bhattacharyya distance](https://en.wikipedia.org/wiki/Bhattacharyya_distance), which is used to compute the statistical overlap between two 3DGS shapes. Like Tiny-LoD it merges splats from bottom to top, but picks pairs of splats to merge based on how similar their shapes and colors are. It is intended for "off-line" use where we prioritize quality over speed.

Bhatt-LoD iterates over all the input splats organized into a priority queue, starting from the smallest splat and working up to the largest. For each splat it searches the neighborhood around the splat for other similar-sized splats and computes a metric . It picks the splat with best similarity metric (if one exists) and merges the two splats into a new splat that is inserted into the queue. This process continues until there is only one splat remaining in the priority queue, which will be the root splat.

To compare splats and we define . The shape similarity is captured by , the Bhattacharyya distance between the two 3D Gaussian density functions. The color similarity is calculated using , the squared delta between their colors .

Because Bhatt-LoD always merges pairs of splats, if we start with input splats we will end up with exactly LoD splats in the final tree. To simplify the tree we prune any interior LoD splats that aren't at least times larger than its children. By default for Bhatt-LoD we use , which results in a LoD splat tree approximately 30-40% larger than the original input splats.

## Progressive Streaming

Spark 2.0 defines a new file format.RAD (for RADiance fields) that compresses 3DGS data and enables random access streaming for progressive refinement as data is transferred across the Internet. 3DGS objects appear almost instantly as a coarse 64K splat approximation of all the splats. Data chunks are fetched to refine the coarsest visible LoD splats first, reprioritizing as the viewer moves around the scene.

### Problems with existing 3DGS file formats

The two most common file formats for 3DGS data are the original [.PLY](https://en.wikipedia.org/wiki/PLY_\(file_format\)) and [.SPZ](https://github.com/nianticlabs/spz), which represent two different styles of encoding data: [**row-oriented** and **column-oriented**](https://en.wikipedia.org/wiki/Data_orientation):

- **.PLY**: After a plain-text header, stores all the properties of splat 0, then all the properties of splat 1, and so on, known as **row-order**. Each value is encoded as a float32, nothing is compressed. 10M splats with SH0..3 may consume up to 2.3 GB of data.
- **.SPZ**: Encodes data in **column-order**, storing the center coordinates for all the splats first, then all the splat opacities, and so on. Each property is stored in fewer bits with reduced precision, for example 0..1 opacity is encoded as 0..255 in one byte. All the columns are concatenated and compressed as a GZ stream. 10M splats with SH0..3 consumes much less data at 200-250 MB.

Because a.PLY file is stored in row-order, we could progressively load splats by showing them as soon as the data is received. But it is uncompressed and its encoding precision is wasteful. An.SPZ file stores similar types of data together in column-order, which results in better compression. Unfortunately it can't be progressively loaded because the entire file must to be received before any splat has all its properties.

### .RAD file

To achieve our goal of a compressed, efficient, streamable format for 3DGS, we developed a new file format.RAD. Our goals were that it should be simple to encode/decode, extensible, have selectable encoding precision, and allow random access.

![](https://www.worldlabs.ai/images/rad-file.png)

RAD file structure, consisting of an extensible JSON metadata header followed by randomly-seekable chunks. Each chunk is a RADC file with JSON header and the properties of a 64K chunk of splats, encoded and compressed in column-order.

The file structure is simple: the header `RAD0`, the byte size of the header metadata, the metadata JSON, followed by one or more chunks of 64K splats. The header metadata contains the offsets and byte sizes of all the chunks, which allows us to fetch chunk data in any order.

Each chunk itself uses a similar format: a header `RADC`, the size of the chunk header metadata, the metadata JSON, followed by the compressed data for 64K splats. The splat properties are stored in column-order with customizable encodings specific to each property. Each property is compressed using Gzip, which performs well because similar data types are stored together.

Because the headers are encoded as JSON it ensures future extensibility using the `version` field and by adding new optional fields. Data type encodings and compression algorithms are selected using string names in the metadata, allowing new strings to be added in the future.

### Spatially partitioning LoD tree

An LoD splat tree can be thought of as existing across four dimensions: three spatial dimensions and a level-of-detail dimension. In order to do progressive refinement with streaming, we need to carefully organize the LoD splats into chunks in the.RAD file.

There are many strategies possible here, but Spark's method aims to have splat data spatially co-located by subdividing space into recursively smaller regions. Chunks of 64K are filled with splats within these spatial regions from largest to smallest, resolving as much detail as possible within the chunk.

The first chunk has no spatial bounds and starts with the root splat at index 0, then its children, then their children in order of decreasing size until we have reached 64K splats. We then subdivide space into [AABB](https://en.wikipedia.org/wiki/Bounding_volume) regions and recurse on each region, adding splats within the regions until they fill a 64K chunk. These regions are further subdivided and the process continues until all splats have been output to a chunk.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/lod-chunking.mp4" aria-label="Spatially partitioning LoD splats into chunks" controls=""></video>

Click to interact in 3D: Spark LoD trees are partitioned into chunks that are spatially co-located and resolve as much detail as possible within 64K splats.

With this strategy, the first 64K chunk of splats loaded are the largest 64K splats in the tree, allowing a coarse version of the splats to be rendered almost immediately. Each additional chunk of 64K splats delivers as much detail as possible within spatially subdivided regions. This way, we can fetch chunks corresponding to regions near the viewer and resolve details quickly.

### Traversing LoD trees with missing chunks

Spark streams.RAD files by loading entire chunks, but chunks may be loaded in any order after chunk 0 depending on which chunks best resolve detail in the viewport. By keeping track of whether each chunk has been loaded, the LoD traversal algorithm can quickly determine if a splat's children have been loaded. If not, the parent splat is rendered instead, awaiting further refinement once the data is received.

While traversing the LoD splat tree, Spark keeps a list of chunks visited and their order, including chunks not yet loaded. Since the traversal algorithm refines splats from largest screen size to smallest, chunks earlier in the list contain larger splats than chunks visited later.

Spark therefore uses this chunk visit order to prioritize.RAD file chunk fetching. It constantly recalculates the highest priority chunks based on camera viewpoint, and uses 3 parallel Web Workers to fetch and decode the data in the background.

## Virtual Memory

[Virtual Memory](https://en.wikipedia.org/wiki/Virtual_memory) is a memory management technique to provide access to huge amounts of idealized "virtual memory" through a fixed pool of real "physical memory". A [page table](https://en.wikipedia.org/wiki/Page_table) is used to map between the virtual and physical in fixed-size pages.

Spark 2.0 adapts this technique to 3DGS, allocating a fixed pool of 16M splats on the GPU and automatically managing the mapping between 64K splat GPU "pages" and virtual 64K chunks of.RAD files. Chunks are loaded into empty pages based on LoD traversal ordering. Chunks are evicted when the page table is full and their priority is lower than new chunks that need to be fetched, in a [least-recently-used](https://en.wikipedia.org/wiki/Page_replacement_algorithm#Least_recently_used) fashion.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/page-table.mp4" aria-label="3DGS virtual splat page table" controls=""></video>

Click to experience in 3D: 3DGS data is organized into chunks/pages of 64K splats, loaded as needed by the camera viewpoint into a unified GPU page table, swapped out for other data in an LRU fashion.

Spark's design allows multiple.RAD files to be fetched simultaneously and share the same page table. For each.RAD file we store its mappings from chunk to the page table, and the converse mappings from page table to file and chunk. During LoD splat traversal over multiple LoD splat trees, we keep track of chunk and file visit order. The result is a global priority ordering across all files and chunks together, allowing us to optimize fetching and storage across splats from all 3DGS objects in the scene.

## Summary

Spark is a 3DGS renderer for THREE.js and WebGL2 with a user-programmable GPU splat processing engine. The new Spark 2.0 adds a 3DGS Level-of-Detail system that adjust the detail level continuously, rendering splat details where you're looking. A new 3DGS file format.RAD enables streaming with progressive refinement, and a virtual splat paging system provides access to infinite splat worlds with a fixed GPU memory allocation.

## Explore Spark 2.0 creations

### Starspeed — by James Kane

Starspeed is a multiplayer spaceship shooter with a 10-song synthwave OST and gripping story by Webby-winning artist and developer James C. Kane. Made with World Labs' Marble and Spark.js plus Blender and three.js, STARSPEED streams kilometer-scale sci-fi environments made of 100,000,000+ gaussian splats directly through the browser with the.rad format.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/starspeed2.mp4" aria-label="Starspeed by James Kane" controls=""></video>

Click to play: Starspeed by James Kane

### Dormant Memories — by Hugues Bruyère

Dormant Memories is a series of interactive scans by Hugues Bruyère, co-founder and chief technologist at Dpt., a creative studio focused on interactive and immersive experiences. The work captures real-world locations and presents them as explorable environments built with Marble, connecting documented spaces with imagined ones to explore atmosphere, memory, and alternate versions of place.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/smallfry.mp4" aria-label="Dormant Memories by Hugues Bruyère" controls=""></video>

Click to experience: Dormant Memories by Hugues Bruyère

### Explore Massive Captured Spaces — by Fujiwara Ryo

Fujiwara Ryo of HoloLab Inc. Spatial Information Technology Division demonstrates large-scale 3D Gaussian splatting with multiple captured scenes containing up to 40 million splats. The experience runs smoothly on smartphones and is also compatible with Quest and Vision Pro.

<video src="https://wlt-ai-cdn.art/spark-2.0/260413/videos/Fujiwara.mp4" aria-label="Explore Massive Captured Spaces by Fujiwara Ryū" controls=""></video>

Click to experience: Explore Massive Captured Spaces by Fujiwara Ryū

## 3DGS + You?

If you found this post interesting, check out our [GitHub repo](https://github.com/sparkjsdev/spark/). Clone the repo and test out the [Spark examples](https://sparkjs.dev/examples/). Learn more about Spark and its capabilities in the [Spark docs](https://sparkjs.dev/docs/). Learn how to [get started with the new LoD system](https://sparkjs.dev/docs/lod-getting-started/).

Need some 3DGS data for your projects? Try World Labs [Marble](https://marble.worldlabs.ai/), where one line of text or an image is all you need to create a 3DGS world. Use [Marble Studio](https://marble.worldlabs.ai/projects) to join your creations into huge sprawling worlds. Make a 3D web app that renders your world using Spark LoD, and share it with all your friends!

If you're new to programming or graphics, modern LLMs are now capable of generating entire THREE.js games from a few lines of English text. Using Spark and Marble anyone can create a 3D experience for the web!

Spark aims to expand what's possible with 3DGS. If you found the technical details on Spark and its LoD system useful, it is open source and you can build on the code and ideas. Every 3D example in this post is a single-page HTML file with linked 3D assets. View Source in your browser and edit it to your liking!

Finally, if you found this write-up interesting and are excited by [World Labs' vision](https://www.worldlabs.ai/about), come [join us](https://job-boards.greenhouse.io/worldlabs)!