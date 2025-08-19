---
description: Learn the fundamentals of browser rendering and techniques for optimizing 3D web experiences
---

# The Principles of Web Optimization

## The Basics of 3D Engines in the Browser

Rendering engines are an abstraction around rendering APIs that simplify the process of drawing meshes to the screen and simulating the lighting of the meshes. In the web, rendering engines are composed of:

- A [rendering pipeline](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview) built on top of a browser rendering API, which updates every frame.
- A developer-facing API for initializing and updating meshes, materials, textures, and lights, written in a browser-supported programming language.
- Abstractions for connecting the output of rendering pipeline to the browser window.
- Abstractions for updating the scene to user inputs received from browser APIs.

### Core Browser Technologies: WebGL and HTMLCanvas

Modern web browsers, such as Safari, Chrome, Firefox, and Edge, implement a set of standardized Graphics APIs and HTML components that collectively support rendering 3D applications in a web page. The primary graphics API is [**WebGL**](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API), which is based on the native OpenGL graphics API. Over the past two years, browsers have also begun to support [**WebGPU**](https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API), a more modern graphics API that integrates tightly with powerful native APIs like Vulkan, Metal, and DirectX 12. The purpose of a graphics API is to provide an interface between the CPU, which constructs an abstract representation of a scene, and the GPU, which renders the abstract scene representation to an image. However, the GPU simply outputs this image as a big list of color values into a special memory allocation known as the **[framebuffer](https://en.wikipedia.org/wiki/Framebuffer)**. In order to actually display the image to the user, the browser provides a special HTML component called the [**canvas**](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/canvas). This canvas component occupies a rectangular area of a webpage, and essentially the output from the GPU. In practice, the developer constructs a canvas, and then accesses the appropriate graphics API (WebGL or WebGPU) through a reference to the canvas. The implementation details of rendering to the canvas are abstracted away from the user.

### Drawing Triangles in the Browser

Graphics APIs like WebGL and WebGPU are used to construct a **[rendering pipeline](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview)**, a program that defines the steps that the underlying native Graphics API (OpenGL, Vulkan, Metal, DirectX) must take when rendering objects. At a _very_ high level, this consists of:

1. Vertex processing. 3D scenes consist of many points in 3D space, referred to as vertices, which compose into triangles, which in turn compose into more complex shapes. In the first step of the rendering pipeline, each vertex in an array of vertices are processed by a "vertex shader", a small program that determines the position and color of a particular vertex given some input properties.
2. Primitive Assembly and Clipping. Each vertex may be part of one or more primitives (point, line, or triangle) in the scene. At this stage, the pipeline generates an array of primitives from the vertices and clips all primitives that extend outside of the camera view.
3. Rasterization. The processed primitives are then _rasterized_, or converted into a sequence of fragments. Whereas primitives represent a shape in 3D space, fragments represent the projection of 3D shapes into 2D space. Imagine taking a picture on a digital camera; the 3D scene you are capturing is recorded by a series of sensors, which record light information from 3D space. A fragment loosely corresponds to one of these sensors; it consists of 2D position data for a point and data interpolated from the vertices that contribute to that point. This fragment data is output to the next stage.
4. Fragment processing. Each fragment is processed by a fragment shader, which determines the final color of each value in the framebuffer.

Consider the following example of a WebGL rendering pipeline, written in JavaScript, which draws a multi-colored triangle to the screen:

```js
const canvas = document.getElementById("myCanvas");
const gl = canvas.getContext("webgl");

// Determines the position and color of each vertex
const vertexShaderSource = `
  // Inputs
  attribute vec4 position;
  attribute vec4 color;

  // Output
  varying vec4 vColor;

  void main() {
    gl_Position = position;
    vColor = color;
  }
`;

// Determines the color of each fragment
// vColor is interpolated based on the distance from each triangle vertex
const fragmentShaderSource = `
  precision highp float;
  varying vec4 vColor;

  void main() {
    gl_FragColor = vColor;
  }
`;

// Compile Vertex and Fragment Shaders for the rendering pipeline
const vertexShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vertexShader, vertexShaderSource);
gl.compileShader(vertexShader);

const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fragmentShaderSource);
gl.compileShader(fragmentShader);

const shaderProgram = gl.createProgram();
gl.attachShader(shaderProgram, vertexShader);
gl.attachShader(shaderProgram, fragmentShader);
gl.linkProgram(shaderProgram);
gl.useProgram(shaderProgram);

// Allocate buffers to pass to the rendering pipeline
const vertices = [
  //  x     y     z     r     g     b
  0.0, 0.5, 0.0, 1.0, 0.0, 0.0, -0.5, -0.5, 0.0, 0.0, 1.0, 0.0, 0.5, -0.5, 0.0, 0.0,
  0.0, 1.0,
];

const vertexBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);

var FSIZE = vertices.BYTES_PER_ELEMENT;
var positionAttribute = gl.getAttribLocation(shaderProgram, "position");
gl.vertexAttribPointer(positionAttribute, 3, gl.FLOAT, false, FSIZE * 6, 0);
gl.enableVertexAttribArray(positionAttribute);

var colorAttribute = gl.getAttribLocation(shaderProgram, "color");
gl.vertexAttribPointer(
  colorAttribute,
  3,
  gl.FLOAT,
  false,
  FSIZE * 6,
  FSIZE * 3
);
gl.enableVertexAttribArray(colorAttribute);

// Draw to the canvas
gl.clearColor(0.0, 0.0, 0.0, 1.0);
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, 3);
```

### Core Engine Abstractions: Meshes, Materials, Cameras, and Lights

Rendering API code is often regarded as verbose and unapproachable, so some abstractions are built on top to make things easier. At the highest level of abstraction is the **scene**, a tree-like data structure that defines the hierarchy of elements that are rendered to the screen. The scene is composed of **meshes**, collections of triangles that from a shape, and have some position, scale, and rotation in the world. Each mesh has a **material**, which defines how the mesh responds to **lights** in the scene. Materials may render just a solid color, can render a **texture**, or can imitate real-life material properties, like wood or skin. A **camera** renders the scene from a particular perspective, which is then output to the browser window via the Canvas API.

A typical rendering pipeline composes abstractions in the [following way](https://toji.dev/webgpu-gltf-case-study/#rendering-with-webgl):

```js
// Heavily-abstracted render loop. This runs every frame.
function render(scene) {
  for (let material of scene.materials) {
    // Initialize Shaders using gl.createShader(), gl.compileShader(), etc.
    InitializeShadersAndTextures(material, scene.camera, scene.lights);

    for (let mesh of scene.meshes.filter(
      (mesh) => mesh.material === material
    )) {
      // Create a WebGL buffer for the mesh vertices using gl.createBuffer()
      InitializeMeshBuffers(mesh);

      for (let instance of mesh.instances.values()) {
        // For each instance of the mesh, set the appropriate color and position
        SetInstanceValues(instance);

        // Render the instance to the screen using gl.drawArrays()
        Draw();
      }
    }
  }

  requestAnimationFrame(function () {
    render(scene);
  });
}

// Example of initializing a scene and starting the render loop
function runApp() {
  const scene = engine.createScene();

  const camera = scene.createCamera();
  camera.position = { x: 0, y: 4, z: -15 };
  camera.lookAt({ x: 0, y: 0, z: 0 });

  const light = scene.createAreaLight();
  light.color = { r: 1, g: 1, b: 1 };

  const basicMaterial = scene.createStandardMaterial();
  basicMaterial.color = { r: 1, g: 0, b: 0 };

  const box = scene.createBox();
  box.material = basicMaterial;

  const boxInstance1 = box.createInstance();
  boxInstance1.position = { x: 1, y: 1, z: 0 };
  boxInstance1.scale = { x: 2, y: 1, z: 0 };

  const boxInstance2 = box.createInstance();
  boxInstance2.position = { x: -1, y: -1, z: 0 };

  render(scene);
}
```

### Responding to User Inputs in the Browser

Generally, users need to interact with the program in some way. In a desktop first-person video game, for example, the camera moves forward, left, back, and right with the W, A, S, and D keys, respectively, and the the camera pans up, down, left, and right by moving the mouse in the respective direction. The browser exposes a few APIs to allow controlled access to the mouse, keyboard, touch, and gamepad events:

- On mobile devices, the [Touch Events API](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events) is used to respond to swipe, tap, and multitap interactions.
- On desktop computers, the [KeyboardEvent API](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent) and [MouseEvent API](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) provide functionality for responding to key strokes, mouse clicks, and mouse movements.
- Developers can use the [Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API/Using_the_Gamepad_API) to detect when a controller is connected to a computer and determine the layout of the controller.
- For XR devices, the [WebXR API](https://developer.mozilla.org/en-US/docs/Web/API/WebXR_Device_API) provides abstractions for handling headset movement and hand/gamepad inputs.

### A Note On Programming Language Support in Web Rendering Engines

For historical reasons and security reasons, the only officially supported browser language is [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript). To execute non-JavaScript code directly in the browser, it must be compiled to a binary format known as [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly), where it can be directly executed by a virtual machine in the browser.

There are therefore two main approaches to building rendering engines for the web:

- Compiling an existing engine written in a low-level languages (e.g. C, C++, Rust) to WebAssembly. This approach is taken by engines like Unity, Godot, and Bevy.
- Writing the engine entirely in JavaScript. This approach is used by engines like PlayCanvas, Three.js, and Babylon.js.

Each approach has certain tradeoffs. With the WebAssembly approach, one challenge is compatibility. For example, automatic memory management is constrained in WebAssembly to maintain security, leading to potential [out-of-memory errors](https://docs.unity3d.com/6000.1/Documentation/Manual/webgl-memory.html) that may not occur in a native application. Another challenge is interfacing with the browser. [Browser APIs are generally not available to the WebAssembly runtime](https://webassembly.org/docs/web/); to interface with the browser, WebAssembly code must call a JavaScript function that then calls the corresponding browser API. Sending commands and data over this reflection layer can be expensive, negating some of the benefits of the generally faster WebAssembly code. However, when properly optimized, low-level code compiled to WebAssembly runs much faster than the equivalent JavaScript code, approaching native speeds.

JavaScript-based engines come with some benefits outside of browser compatibility. Engine code can be directly profiled and debugged in the browser, making optimization and bug-fixing work much simpler. There is no additional compilation step or build configuration like there is for WebAssembly code, making it much easier to prototype and deploy an application. Any improvements to the browser's JavaScript engine will immediately benefit performance, while a WebAssembly application likely needs to be recompiled or rewritten to take advantage of browser improvements. However, while modern JavaScript engines like V8 and JavaScriptCore are fast and perfectly capable of running intense graphics applications, the language has a performance ceiling. In particular, the language is single-threaded, uses automatic memory management, and cannot leverage [Single-Instruction Multiple Data](https://en.wikipedia.org/wiki/Single_instruction%2C_multiple_data) (SIMD) parallelization in the browser.

The choice of engine is not as clear-cut as performance vs. usability. Given the complex nature of rendering engines, it is very difficult to build comprehensive performance benchmarks proving one engine is faster than another. It is more important to choose an engine based on the type of experience you want to build and your previous development experience.

### Learning Resources

General WebGL and WebGPU tutorials:

1. Canvas Tutorial: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial
2. Learning WebGL: https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/By_example
3. WebGL Fundamentals: https://webglfundamentals.org/
4. WebGPU Fundamentals: https://webgpufundamentals.org/

## Quantifying the Characteristics of Good Optimization

### Loading times

### Smoothness

### Stability

### Legibility

### Responsiveness

### Scalability

## Target Metrics for Optimization

### Average Framerate

### Minimum Framerate

### Time to First Paint

### Time to First Interaction

### Framebuffer Scaling

### Input Latency

## The Challenges of Optimizing for the Browser

### Performance Differences between WebGL and Native Rendering APIs

By default, rendering engines like Three.js, Babylon.js, Unity, and PlayCanvas use the [WebGL 2 API](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API#webgl_2) to take advantage of hardware-accelerated graphics in the browser. WebGL 2 largely conforms to the Open GL ES 2.0 standard and dispatches commands to the GPU, meaning that the performance of a WebGL API call is very close to the corresponding native OpenGL call. However, there are some limitations to WebGL 2 in comparison to native graphics APIs:

- **Dispatching Overhead**: There is [overhead when dispatching WebGL calls on the CPU](https://docs.unity3d.com/6000.3/Documentation/Manual/webgl-performance.html), as the WebGL API call must be translated into the correct native graphics API call. Further, the browser implements more security checks than native code to prevent things like [Out of Range Memory Accesses](https://www.khronos.org/webgl/wiki/Main%20Page/cms/security). This slower dispatching limits the number of draw calls that can be performed in the browser.
- **Missing Modern Features**: Because WebGL is based on OpenGL ES 2.0, it lacks many features that modern graphics APIs like DirectX 12, Metal, and [Vulkan can provide](https://github.com/KhronosGroup/Vulkan-Samples/blob/main/samples/vulkan_basics.adoc), limiting the theoretical performance of an experience. The WebGPU browser API does support many of these features, but WebGPU support is still developing in most major Web engines, like [Three.js](https://github.com/mrdoob/three.js/issues/28968), [Babylon.js](https://doc.babylonjs.com/setup/support/webGPU/webGPUStatus/#features-not-working-because-not-implemented-yet), and [Unity](https://docs.unity3d.com/Manual/WebGPU.html).

### Memory Management in 3D Web Applications

3D applications running in the browser can be very sensitive to JavaScript Garbage Collection (GC) pauses. [Garbage Collection](<https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)>) is an automatic memory management technique used in many programming languages, including JavaScript. This technique helps avoid issues like memory leaks and dangling pointers, but has some performance overhead. When memory usage is high in a particular frame, the resulting garbage collection step will freeze the main frame until it completes. In WebXR, this freeze can result in dropped frames, affecting user experience. Although engines like Unity use garbage collection in C# scripting contexts [\[10\]](https://docs.unity3d.com/6000.1/Documentation/Manual/performance-garbage-collector.html), the underlying engine is written in C++, which can take advantage of manual memory management in native contexts. Engines written in JavaScript, like Three.js, do not have this advantage, and developers must be very careful about allocating memory and reusing resources like Vectors and Arrays.

### App Startup Time

Startup time is limited by network bandwidth in the browser. This contrasts with native apps, where assets and source code are typically downloaded on the initial install, or in explicit updates to the app. Refer to [Optimizing for the Web](../optimization.md) for more details on optimizing assets for the web.

### Optimizing for Mobile Devices

### Optimizing for XR Devices

## Optimization Strategies for the Web

Given these constraints, how should a developer approach optimizing for Web in comparison to optimizing for a native context?

Optimization techniques for Web experiences typically fall into 3 buckets:

### 1. Leveraging Browser Technologies

WebXR development optimization requires the use of unique browser APIs, such as WebGL, WebWorkers, and WebAssembly. See the section below for leveraging browser APIs in 3D experiences.

### 2. Optimizing the Scene for the Browser

A developer may need to tailor their experience specifically for mobile chipsets, using lower polygon counts, reducing draw calls, enabling GPU instancing, and reducing the number of dynamic lights in a scene. Assets should be optimized, as they need to be downloaded over the network on application start.

### 3. Engine-specific Optimizations

Many optimization techniques from traditional game development still apply to Web engines; for instance, [object pooling](https://en.wikipedia.org/wiki/Object_pool_pattern), [shader optimization](https://docs.unity3d.com/6000.1/Documentation/Manual/SL-ShaderPerformance.html), and instancing are still valid techniques. However, WebXR applications require even more optimization because of the higher minimum framerates and lower available compute time. Explicit techniques typically vary for each game engine; for example, object pooling may be more effective in some engines than others. Refer to subsequent pages for specific optimization techniques for major WebXR engines.

## Leveraging Browser APIs in WebXR Experiences

### Multi-threading

The [Web Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) allows for complex, non-rendering work to be run on a background thread, enabling basic multi-threading. This is particularly useful for computationally expensive tasks like AI pathfinding.

WebGL Rendering can also be performed in a worker thread using the [OffscreenCanvas](https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvas) API. This is particularly useful if the main thread is very busy with user interactions and/or animations.

### Caching Assets

The browser exposes two key APIs for caching source code and assets:

- The [Service Worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) API is useful for caching game assets. It acts as a local proxy server between the application and asset CDN, intercepting potentially expensive asset download requests and returning a cached response. This can enable near-native loading performance and can reduce network bandwidth usage for users that may have service-provider imposed data caps.
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) is useful for storing large amounts of serializable text data across sessions. This can be used to store game save files and configuration files locally, rather than replicating them to a server: Rendering engines like Babylon [\[13\]](https://doc.babylonjs.com/features/featuresDeepDive/scene/optimizeCached) and Unity [\[14\]](https://docs.unity3d.com/6000.1/Documentation/Manual/webgl-caching.html) also allow caching assets in IndexedDB for faster loading times.

### WebAssembly

[WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly) (or Wasm) is a special binary instruction set that can be executed in all major browsers. Non-browser compatible languages like C++, Rust, and C# can be compiled to this binary format and then run in the browser, often leading to dramatic performance improvements when compared to JavaScript. Real-time 3D engines written in C++ or Rust like Unity and Bevy make use of Wasm to run games in the browser at ["near-native speed"](https://developer.mozilla.org/en-US/docs/WebAssembly/Guides/Concepts#what_is_webassembly).

In Unity, developers may need to opt into particular Wasm configurations to ensure the best performance:

- [Enabling WebAssembly 2023](https://docs.unity3d.com/6000.1/Documentation/Manual/webassembly-2023.html)
- [Compiling native plug-ins to Wasm](https://docs.unity3d.com/6000.1/Documentation/Manual/webgl-native-plugins-with-emscripten.html)

JavaScript-based engines like Three.js, Babylon.js, and PlayCanvas can leverage Wasm for computationally expensive features like physics simulation, texture decompression, and mesh optimization:

- [MeshOptimizer](https://www.npmjs.com/package/meshoptimizer)
- [Basis Universal](https://github.com/BinomialLLC/basis_universal/blob/master/webgl/encoder/README.md) texture compression
- [Rapier](https://rapier.rs/docs/user_guides/javascript/getting_started_js) physics engine

First-party code that is not well-suited to JavaScript can be re-written in a language like C, C++, or Rust and compiled to Wasm using tools like [Emscripten](https://emscripten.org/).

#### Limitations

There are some limitations to Wasm in comparison to native code:

- Startup Time: Like all web app source code, Wasm bytecode must be downloaded by the browser before it can begin running. This can result in worse startup time in comparison to native apps, where source is downloaded ahead of time.
- Garbage Collection In Unity WebGL, garbage collection only runs at the end of each frame. This means that allocating many temporary values in a single frame can lead to ["temporary quadratic memory growth pressure for the garbage collector"](https://docs.unity3d.com/2022.3/Documentation/Manual/webgl-memory.html).
- Multi-threading: Threading support in Wasm is constantly evolving. Although the Wasm supports multi-threading and SIMD instructions, engines must explicitly support these Wasm features. For instance, [Unity does not support C# multithreading](https://docs.unity3d.com/Manual/webgl-technical-overview.html).

WebAssembly does not always guarantee the [best performance](https://ianjk.com/webassembly-vs-javascript/). It may be more useful to optimize JavaScript code than to introduce Wasm into your project.

### WebGPU

Though not all engines fully support WebGPU for all rendering tasks, WebGPU can still be leveraged for general purpose GPU computations, allowing non-rendering work to be done on the GPU. This enables highly parallelizable computations like AI pathfinding and animations to be performed asynchronously on the GPU [\[15\]](https://surma.dev/things/webgpu/) [\[16\]](https://webgpufundamentals.org/webgpu/lessons/webgpu-compute-shaders.html).

## Optimizing Scenes for Browsers

In addition to leveraging browser APIs to improve performance, developers must also tailor their experiences towards the browser and the devices that WebXR applications typically run on. In the next pages we will cover engine-specific techniques for implementing these optimizations.

### Reducing and Batching Draw Calls

The first technique to try before lowering the quality of a scene is reducing the number of draw calls per frame. In graphics programming, a draw call is a command sent to the GPU telling it to render a set of triangles. Because GPUs excel at rendering large batches of triangles, it is generally more expensive to generate and submit a draw call on the CPU than it is to execute it on the GPU. As a result, it is advantageous to perform few draw calls with more triangles per draw call than to submit many draw calls with few triangles. In general, draw calls can be reduced by:

- Reusing a single material across different meshes by using atlas textures and texture arrays
- Merging static meshes that use the same material
- Using hardware instancing to draw meshes with the same geometry in a single draw call
- Leveraging level of detail (LOD) systems
- Culling meshes that are not visible

### Reducing Scene Complexity

If draw calls can not be batched further and performance does not match expectations, the developer should consider reducing the complexity of the scene. This can include:

- Reducing the meshes from the scene
- Throttling animation frame rates
- Removing transparency from meshes

### Reducing Visual Fidelity

If the application is GPU bound, i.e. the application spends a significant portion of time on the GPU, the developer should reduce the visual fidelity of the app. This can include:

- Lowering the polygon count of mehses
- Reducing the size of textures
- Reducing the number of dynamic lights in the scene
- Simplifying shaders
- Removing complex post-processing shaders like fog
