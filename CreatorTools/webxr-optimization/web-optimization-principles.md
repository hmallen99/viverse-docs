---
description: Learn the fundamentals of browser rendering and techniques for optimizing 3D web experiences
---

# The Principles of Web Optimization

## The Basics of 3D in the Browser

### WebGL, WebGPU, and HTMLCanvas

Modern web browsers, such as Safari, Chrome, Firefox, and Edge, implement a set of standardized Graphics APIs and HTML components that collectively support rendering 3D applications in a web page. The primary graphics API is [**WebGL**](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API), which is based on the native OpenGL graphics API. Over the past two years, browsers have also begun to support [**WebGPU**](https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API), a more modern graphics API that integrates tightly with powerful native APIs like Vulkan, Metal, and DirectX 12. The purpose of a graphics API is to provide an interface between the CPU, which constructs an abstract representation of a scene, and the GPU, which renders the abstract scene representation to an image. However, the GPU simply outputs this image as a big list of color values into a special memory allocation known as the "framebuffer". In order to actually display the image to the user, the browser provides a special HTML component called the [**canvas**](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/canvas). This canvas component occupies a rectangular area of a webpage, and essentially the output from the GPU. In practice, the developer constructs a canvas, and then accesses the appropriate graphics API (WebGL or WebGPU) through a reference to the canvas. The implementation details of rendering to the canvas are abstracted away from the user.

### The Rendering Pipeline

Graphics APIs like WebGL and WebGPU handle the construction of a "[rendering pipeline](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview)", the steps that the underlying native Graphics API (OpenGL, Vulkan, Metal, DirectX) take when rendering objects. At a *very* high level, this consists of:

1. Vertex processing. 3D scenes consist of many points in 3D space, referred to as vertices, which compose into triangles, which in turn compose into more complex shapes. In the first step of the rendering pipeline, each vertex in an array of vertices are processed by a "vertex shader", a small program that determines the position and color of a particular vertex given some input properties.
2. Primitive Assembly and Clipping. Each vertex may be part of one or more primitives (point, line, or triangle) in the scene. At this stage, the pipeline generates an array of primitives from the vertices and clips all primitives that extend outside of the camera view.
3. Rasterization. The processed primitives are then *rasterized*, or converted into a sequence of fragments. Whereas primitives represent a shape in 3D space, fragments represent the projection of 3D shapes into 2D space. Imagine taking a picture on a digital camera; the 3D scene you are capturing is recorded by a series of sensors, which record light information from 3D space. A fragment loosely corresponds to one of these sensors; it consists of 2D position data for a point and data interpolated from the vertices that contribute to that point. This fragment data is output to the next stage.
4. Fragment processing. Each fragment is processed by a fragment shader, which determines the final color of each value in the framebuffer.

This is very abstract. In the following example, we will show how WebGL is used to construct this pipeline.

### WebGL Example

In this example, we will render a triangle to the screen.

The Vertex Shader:
```js
const canvas = document.getElementById('myCanvas');
const gl = canvas.getContext('webgl');

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
    0.0,  0.5,  0.0,  1.0,  0.0,  0.0,
   -0.5, -0.5,  0.0,  0.0,  1.0,  0.0,
    0.5, -0.5,  0.0,  0.0,  0.0,  1.0,
];

const vertexBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);

var FSIZE = vertices.BYTES_PER_ELEMENT;
var positionAttribute = gl.getAttribLocation(shaderProgram, 'position');
gl.vertexAttribPointer(positionAttribute, 3, gl.FLOAT, false, FSIZE * 6, 0);
gl.enableVertexAttribArray(positionAttribute);

var colorAttribute = gl.getAttribLocation(shaderProgram, 'color');
gl.vertexAttribPointer(colorAttribute, 3, gl.FLOAT, false, FSIZE * 6, FSIZE * 3);
gl.enableVertexAttribArray(colorAttribute);

// Draw to the canvas
gl.clearColor(0.0, 0.0, 0.0, 1.0);
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, 3);
```

### Resources

1. Canvas Tutorial: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial
2. Learning WebGL: https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/By_example
3. WebGL Fundamentals: https://webglfundamentals.org/
4. WebGPU Fundamentals: https://webgpufundamentals.org/

### Performance Differences between WebGL and Native Rendering APIs

By default, rendering engines like Three.js, Babylon.js, Unity, and PlayCanvas use the [WebGL 2 API](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API#webgl_2) to take advantage of hardware-accelerated graphics in the browser. WebGL 2 largely conforms to the Open GL ES 2.0 standard and dispatches commands to the GPU, meaning that the performance of a WebGL API call is very close to the corresponding native OpenGL call. However, there are some limitations to WebGL 2 in comparison to native graphics APIs:

- **Dispatching Overhead**: There is [overhead when dispatching WebGL calls on the CPU](https://docs.unity3d.com/6000.3/Documentation/Manual/webgl-performance.html), as the WebGL API call must be translated into the correct native graphics API call. Further, the browser implements more security checks than native code to prevent things like [Out of Range Memory Accesses](https://www.khronos.org/webgl/wiki/Main%20Page/cms/security). This slower dispatching limits the number of draw calls that can be performed in the browser.
- **Missing Modern Features**: Because WebGL is based on OpenGL ES 2.0, it lacks many features that modern graphics APIs like DirectX 12, Metal, and [Vulkan can provide](https://github.com/KhronosGroup/Vulkan-Samples/blob/main/samples/vulkan_basics.adoc), limiting the theoretical performance of an experience. The WebGPU browser API does support many of these features, but WebGPU support is still developing in most major Web engines, like [Three.js](https://github.com/mrdoob/three.js/issues/28968), [Babylon.js](https://doc.babylonjs.com/setup/support/webGPU/webGPUStatus/#features-not-working-because-not-implemented-yet), and [Unity](https://docs.unity3d.com/Manual/WebGPU.html).

### Programming Language Support in 3D Web Applications

For historical reasons and security reasons, the only officially supported browser language is JavaScript. To execute non-JavaScript code directly in the browser, it must be compiled to a binary format known as [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly) (refer to the WebAssembly section below for more details), where it can be directly executed by a virtual machine in the browser.

There are therefore two main approaches to building rendering engines for the web:

- Compiling an existing engine written in a low-level languages (e.g. C, C++, Rust) to WebAssembly. This approach is taken by engines like Unity, Godot, and Bevy.
- Writing the engine entirely in JavaScript. This approach is used by engines like PlayCanvas, Three.js, and Babylon.js.

Each approach has certain tradeoffs. With the WebAssembly approach, one challenge is compatibility. For example, automatic memory management is constrained in WebAssembly to maintain security, leading to potential [out-of-memory errors](https://docs.unity3d.com/6000.1/Documentation/Manual/webgl-memory.html) that may not occur in a native application. Another challenge is interfacing with the browser. [Browser APIs are generally not available to the WebAssembly runtime](https://webassembly.org/docs/web/); to interface with the browser, WebAssembly code must call a JavaScript function that then calls the corresponding browser API. Sending commands and data over this reflection layer can be expensive, negating some of the benefits of the generally faster WebAssembly code. However, when properly optimized, low-level code compiled to WebAssembly runs much faster than the equivalent JavaScript code, approaching native speeds.

JavaScript-based engines come with some benefits outside of browser compatibility. Engine code can be directly profiled and debugged in the browser, making optimization and bug-fixing work much simpler. There is no additional compilation step or build configuration like there is for WebAssembly code, making it much easier to prototype and deploy an application. Any improvements to the browser's JavaScript engine will immediately benefit performance, while a WebAssembly application likely needs to be recompiled or rewritten to take advantage of browser improvements. However, while modern JavaScript engines like V8 and JavaScriptCore are fast and perfectly capable of running intense graphics applications, the language has a performance ceiling. In particular, the language is single-threaded, uses automatic memory management, and cannot leverage SIMD in the browser.

The choice of engine is not as clear-cut as performance vs. usability. Given the complex nature of rendering engines, it is very difficult to build comprehensive performance benchmarks proving one engine is faster than another. It is more important to choose an engine based on the type of experience you want to build and your previous development experience.

### Memory Management in 3D Web Applications

3D applications running in the browser can be very sensitive to JavaScript Garbage Collection (GC) pauses. [Garbage Collection](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)) is an automatic memory management technique used in many programming languages, including JavaScript. This technique helps avoid issues like memory leaks and dangling pointers, but has some performance overhead. When memory usage is high in a particular frame, the resulting garbage collection step will freeze the main frame until it completes. In WebXR, this freeze can result in dropped frames, affecting user experience. Although engines like Unity use garbage collection in C# scripting contexts [\[10\]](https://docs.unity3d.com/6000.1/Documentation/Manual/performance-garbage-collector.html), the underlying engine is written in C++, which can take advantage of manual memory management in native contexts. Engines written in JavaScript, like Three.js, do not have this advantage, and developers must be very careful about allocating memory and reusing resources like Vectors and Arrays.

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

## Optimization Strategies for the Web

## Platform-Specific Optimization
