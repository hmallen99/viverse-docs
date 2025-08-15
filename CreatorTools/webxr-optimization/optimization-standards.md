# Key Optimization Standards

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

## Key Metrics for 3D Optimization

1. Device Capabilities
2. Performance Metrics
3. Scene Metrics

## Device Capabilities

### CPU Speed

### CPU Core Count

### System RAM

### GPU VRAM

## Application Metrics

### Frame Times and Frames per Second (FPS)

### Dropped Frames

### Memory Usage

### Bundle Size

## Scene Metrics

### Draw Calls

### Mesh Count

### Material Count

### Shader Complexity

## Performance Targets for 3D Metrics
