---
description: A gentle introduction to optimizing 3D experiences for the Web
---

# Introduction to Optimizing for the Web

***
VIVERSE experiences run in web browsers. This allows 3D experiences to securely run on almost any hardware, as long as the web browser supports [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API). This documentation details the benefits of developing for the web, as well as some key differences with traditional game development and 3D software development.

## Why Build for the Web?

### Reach millions of users instantly

An estimated **3 billion people** have access to a 3D-capable web browser.

### Launch on any device with a web browser

3D web apps reach phones, desktop computers, laptop computers, and Mixed Reality headsets via the same URL. Chrome alone accounts for ~68% of global browsing.

### Secure by default

The [web’s](https://developer.mozilla.org/en-US/docs/Web/Security) application sandbox, HTTPS, and permissions model let you consume powerful graphics, XR, and platform features with user consent and origin isolation.

### The same standards for everyone

WebGPU/WebGL 2, WebXR, WebAssembly (WASM), CSS, and JavaScript are standardized by mature standards bodies and implemented across most major web browser engines. There is no store gatekeeping compared to native platforms.

## By the Numbers: 3D Web Availability

<figure><img src=".gitbook/assets/3D Capable Demographics.png" alt="" width="375"><figcaption><p>Population of 3D Capable Users</p></figcaption></figure>

<figure><img src=".gitbook/assets/OS Demographics.png" alt="" width="375"><figcaption><p>Operating System Demographics</p></figcaption></figure>

<figure><img src=".gitbook/assets/Browser Demographics.png" alt="" width="375"><figcaption><p>Browser Demographics</p></figcaption></figure>

<figure><img src=".gitbook/assets/WebGL Support.png" alt="" width="375"><figcaption><p>WebGL Support Across Devices</p></figcaption></figure>

## Optimizing 3D experiences for the Web vs native platforms

### Load in fast

**Users expect web experiences to load more quickly than native experiences**. Native applications are downloaded ahead of launch, making it much easier to achieve fast loading times. Web applications must achieve better launch times while downloading assets after launch.

- Keep assets small - users have to download assets over their network on each page load. Keeping assets small makes your experience accessible to users with slower internet speeds and speeds up the loading time.
- Load in assets as you go - one benefit of the browser is that assets can be continuously fetched over the network. Rather than loading everything in at once, only download it when you need it.
- Show the user something as soon as you can - users will be visiting your game from a web page, which typically load very quickly. To keep the experience seamless, make sure to render a loading bar while resources are downloading.

### Expect cross-platform use

**The same build runs across mobile, desktop, and XR platforms**. Unlike native applications, where a separate build must be created for each platform, only one build is generated for the web. This simplifies deployment, but performance tuning must happen at runtime.

- Set performance settings on launch - web users expect that the default graphics settings will perform well on launch. Use lower graphics settings on mobile and XR than on desktop.
- Scale performance as you go - there is a large variance of 3D performance within device classes; a mobile user could be running on the latest iPhone or a decade-old budget phone. Track performance metrics throughout the app runtime, and scale graphics settings up or down according to the device's abilities.
- Determine the correct input method - make sure that the correct input format is selected for each platform you support: touch inputs for mobile; keyboard/mouse on desktop; and controllers on XR. Be prepared to detect gamepads and alternate input sources on each platform.
- Scale resolution according to the device - users expect UI elements and resolution to scale with the size of the screen. Mobile devices typically run at a lower resolution with larger UI elements, while desktops run at a higher resolution with proportionally smaller UI elements. Mobile devices also support both landscape and portrait mode; ensure that the experience renders well in portrait mode or notify the user that the game must run in landscape mode.

### Optimize scenes for the browser

**Developers must tailor their experience towards running in a WebGL context**. The browser adds some CPU overhead compared to native apps to ensure the security of WebGL experiences.

- Understand how draw calls impact performance - the CPU performance of a scene typically scales with the number of draw calls that occur in each frame. A draw call is a command that the engine sends to the GPU telling it to a draw a series of triangles or pixels. This operation happens very quickly on the GPU, but is very slow on the CPU, making it advantageous to batch draw calls together or strip them out entirely.
- Reuse materials and merge static meshes - By leveraging atlas textures and texture arrays, a single material can be used across multiple meshes. This helps to batch draw calls, which typically scale with the number of meshes and materials in the scene. If a mesh does not move in the scene, it can be combined with other static meshes and materials into a single super-mesh that renders in a single draw call.
- Leverage hardware instancing and level of detail (LOD) systems - while the browser adds some CPU overhead for security and automatic memory management, GPU performance is nearly identical to OpenGL native experiences. Thus, it is important to leverage hardware techniques like instancing to draw many copies of the same mesh without needing to process all of the copies on the CPU. This is particularly useful when rendering LOD systems, which can leverage instancing to render hundreds of background objects like plants, crowds, and buildings at lower detail at a very low CPU cost.
- Reduce scene complexity - If CPU performance is still sub-optimal in the browser with properly batched draw calls, a user may need to reduce the complexity of the scene. This can include reducing the number of meshes in the scene, rendering animations at lower framerates, removing transparent meshes, which can incur multiple additional draw calls, and reducing the number of lights in the scene.
- Reduce visual fidelity - If the app is GPU-bound, i.e. too much time is spent each frame drawing triangles to the canvas, consider lowering the quality of the scene. This can include: lowering the polygon count of meshes, reducing texture sizes, reducing the number of dynamic lights, removing expensive math functions from shaders, and removing post-processing effects like fog. Finally, users can lower the resolution of the scene, which typically scales with GPU performance.

### Select the best engine for the experience

**Different web engines are well-suited for different types of experiences**. While some native engines, like Unity, allow building experiences for the web, some developers may prefer to use an engine built specifically for the web.

- Developer experience matters: developers should use an engine that they enjoy developing in. If you're already comfortable with Unity, then it makes sense to continue with that engine. If you're looking to jump into JavaScript development, but still want an editor, PlayCanvas might be the right choice. For programming and rendering specialists, Three.js and Babylon provide the opportunity to build up a 3D framework from scratch, or to grab one off the shelf from the extensive communities.
- Be careful with file size: Without optimization, fully-featured C++ engines like Unity can have much larger application sizes than JavaScript engines like Babylon.js, Three.js, and PlayCanvas. If you don't need all of the features of Unity, using a javascript-based engine may make bundle size optimization easier. Users on mobile devices typically prefer smaller app sizes, as they may be on a cellular network.
- Device support: Web browsers are designed to support a wide range of devices, so developers generally do not need to worry about mobile and desktop rendering support. If you plan to support mobile devices, some engines, like Unity and PlayCanvas, support touch controls out of the box, while for engines like Three.js, you may need to use a third-party solution or touch controls yourself. If you plan to support XR, make sure your chosen engine has WebXR support.

## The Six Characteristics of an Optimized Experience

Many developers implicitly understand when an experience is properly optimized, but may have a hard time describing the exact characteristics that constitute a well-optimized scene and the numbers that back up those characteristics. Here are six characteristics that make up a well-optimized experience:

### Fast-loading

Extensive research shows that users are much more likely to return to applications that load quickly. In a well-optimized experience, users can expect to see a loading indicator in milliseconds and interact with the scene in seconds.

### Smooth

Smoothness is the characteristic that users typically associate with performance. Smoothness is defined by how a scene looks in motion; in a smooth scene, the user should not be able to perceive the gaps in time between each image, instead perceiving a continuous stream of imagery.

### Stable

A stable scene produces new images at a consistent rate, regardless of frequency. When there is a random, large gap between new images, the user's immersion is broken, like when a video buffers.

### Legible

A scene may be smooth and stable, but this does not matter if individual elements in the scene cannot be interpreted. Users must be able to read all UI elements and determine what models are in motion. This characteristic is related to the resolution of the screen and texture, as well as any aliasing, or blurriness, in the scene.

### Responsive

In a responsive experience, the scene updates immediately in response to a user's input. Users expect the camera to track directly with inputs, and can even experience nausea if there is too much latency. Beyond the camera movement, users expect UI elements to respond to hover and click interactions and player animations to trigger when a button is pressed. It is very easy to break immersion when an experience is not responsive.

### Scalable

Regardless of whether an experience runs in a mobile browser or desktop browser, users expect performance comparable to native applications on that platform. A scalable experience tunes its performance to the device it's running on.
