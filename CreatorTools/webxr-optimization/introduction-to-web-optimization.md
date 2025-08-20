---
description: A gentle introduction to optimizing 3D experiences for the Web
---

# Introduction to Optimizing for the Web

***
VIVERSE experiences run in web browsers. This allows 3D experiences to securely run on almost any hardware, as long as the web browser supports [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API). This documentation details the benefits of developing for the web, as well as some key differences with traditional game development and 3D software development.

## Why Build for the Web?

### Reach millions of users instantly

An estimated **3 billion people** have access to a 3D-capable web browser and high-speed internet connection [[0](https://datareportal.com/reports/digital-2025-global-overview-report), [1](https://ngital.com/bangladesh-internet-penetration-2025-data-insights/), [2](https://africa.businessinsider.com/local/lifestyle/african-countries-with-the-largest-internet-population-in-2025/871gpnf), [3](https://www.itu.int/itu-d/reports/statistics/2024/11/10/ff24-internet-use/), [4](https://datareportal.com/reports/digital-2025-sub-section-accelerated-access)]:

<figure><img src=".gitbook/assets/3D Capable Demographics.png" alt="" width="375"><figcaption><p>Population of 3D Capable Users</p></figcaption></figure>

### Launch on any device with a 3D-compatible web browser

3D web apps reach phones, desktop computers, laptop computers, and mixed reality (XR) headsets via the same URL. The vast majority of internet browsing happens on 3D-capable browsers (Chrome, Edge, Firefox, and Safari) [[5](https://gs.statcounter.com/browser-market-share)] running on 3D-capable operating systems (Android, iOS, OSX, Windows) [[6](https://www.gsma.com/r/wp-content/uploads/2024/10/The-State-of-Mobile-Internet-Connectivity-Report-2024.pdf)].

<figure><img src=".gitbook/assets/Browser Demographics.png" alt="" width="375"><figcaption><p>Browser Demographics</p></figcaption></figure>

<figure><img src=".gitbook/assets/OS Demographics.png" alt="" width="375"><figcaption><p>Operating System Demographics</p></figcaption></figure>

An estimated 99% of browsers in use support 3D experiences via the WebGL API [[7](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API#api.webgl2renderingcontext)]:

<figure><img src=".gitbook/assets/WebGL Support.png" alt="" width="375"><figcaption><p>WebGL Support Across Devices</p></figcaption></figure>

### Secure by default

The [web’s](https://developer.mozilla.org/en-US/docs/Web/Security) application sandbox, HTTPS, and permissions model let you consume powerful graphics, XR, and platform features with user consent and origin isolation.

### The same standards for everyone

APIs and technologies like [WebGL](https://registry.khronos.org/webgl/specs/latest/2.0/), [WebGPU](https://www.w3.org/TR/webgpu/), [WebXR](https://www.w3.org/TR/webxr/), [WebAssembly](https://www.w3.org/groups/wg/wasm/) (WASM), and [CSS](https://www.w3.org/Style/CSS/), and JavaScript are standardized by mature standards bodies and implemented across most major web browser engines. There is no store gatekeeping compared to native platforms.

## Optimizing 3D experiences for the web vs native platforms

The core principles of game development are the same across web and native, but there are key optimizations to make on the web platform to ensure that users have a great first impression and keep coming back:

### Load in fast

**Users expect web experiences to load more quickly than native experiences**. Native applications are downloaded ahead of launch, making it much easier to achieve fast loading times. Web applications must achieve better launch times while downloading assets after launch.

- Keep assets small - users have to download assets over their network on each page load. Keeping assets small makes your experience accessible to users with slower internet speeds and speeds up the loading time.
- Load in assets as you go - one benefit of the browser is that assets can be continuously fetched over the network. Rather than loading everything in at once, only download it when you need it.
- Show the user something as soon as you can - users will be visiting your game from a web page, which typically load very quickly. To keep the experience seamless, make sure to render a loading bar while resources are downloading.

### Prepare for cross-platform use

**The same build runs across mobile, desktop, and XR platforms**. Unlike native applications, where a separate build must be created for each platform, only one build is generated for the web. This simplifies deployment, but performance tuning must happen at runtime.

- Set performance settings on launch - web users expect that the default graphics settings will perform well on launch. Use lower graphics settings on mobile and XR than on desktop.
- Scale performance as you go - there is a large variance of 3D performance within device classes; a mobile user could be running on the latest iPhone or a decade-old budget phone. Track performance metrics throughout the app runtime, and scale graphics settings up or down according to the device's abilities.
- Determine the correct input method - make sure that the correct input format is selected for each platform you support: touch inputs for mobile; keyboard/mouse on desktop; and controllers on XR. Be prepared to detect gamepads and alternate input sources on each platform.
- Scale resolution according to the device - users expect UI elements and resolution to scale with the size of the screen. Mobile devices typically run at a lower resolution with larger UI elements, while desktops run at a higher resolution with proportionally smaller UI elements. Mobile devices also support both landscape and portrait mode; ensure that the experience renders well in portrait mode or notify the user that the game must run in landscape mode.

### Optimize scenes for the browser

**Developers must tailor their experience towards running in a WebGL context**. The browser adds some CPU overhead compared to native apps to ensure the security of WebGL experiences.

- Account for WebGL execution overhead - the browser adds security measures to ensure that APIs like [WebGL](https://registry.khronos.org/webgl/specs/1.0/#ATTRIBS_AND_RANGE_CHECKING) can't be used to run malicious code. This adds some CPU overhead to 3D web applications compared to native applications, giving slightly less frame time to 3D logic. GPU rendering performance is largely the same [compared to an OpenGL application](https://docs.unity3d.com/6000.2/Documentation/Manual/webgl-performance.html).
- Understand how [draw calls](https://howik.com/understanding-draw-calls) impact performance - CPU performance scales with the number of draw calls that occur in each frame. A draw call is a command that the engine sends to the GPU telling it to a draw a series of triangles or pixels. This operation happens quickly on the GPU, but is slow on the CPU, making it advantageous to reduce the number of draw calls.
- Reuse materials and merge meshes wherever possible.
- Leverage GPU instancing and level of detail (LOD) systems.
- Reduce scene complexity - reduce the mesh count, render animations at lower framerates, remove transparency and reflections.
- Reduce visual fidelity - lowering mesh and texture resolution, reduce the number of dynamic lights, simplify and remove post-processing effects, lower the output resolution.

### Select the best engine for the experience

**Different web engines are well-suited for different types of experiences**. There are numerous feature-packed 3D rendering engines for the web, like [PlayCanvas](https://playcanvas.com/), [Unity](https://docs.unity3d.com/6000.2/Documentation/Manual/webgl.html), [Three.js](https://threejs.org/), and [Babylon.js](https://www.babylonjs.com/). Each engine has different strengths and weaknesses. Here are some key things to consider when deciding:

- Double-check the supported features - Built-in engine features generally perform better than homebrewed solutions, since they can be tested and iterated on by a large developer base. If you need features like rigid-body physics or photorealistic lighting, it could be better to develop in an engine that supports them by default, like Unity.
- Developer experience matters - developers should use an engine that suits their strengths. If you're already comfortable with Unity, then continue with that engine. If you're looking to jump into JavaScript development, but still want an editor, PlayCanvas might be the right choice. For programming specialists, Three.js and Babylon.js provide more flexibility.
- Be careful with file size - Without optimization, fully-featured C++ engines like Unity can have much larger application sizes than JavaScript engines like Babylon.js, Three.js, and PlayCanvas. If you don't need all of the features of Unity, using a javascript-based engine may make bundle size optimization easier. Users on mobile devices typically prefer smaller app sizes, as they may be on a cellular network.
- Device support - If you plan to support mobile devices, some engines, like Unity and PlayCanvas, support touch controls out of the box, while for engines like Three.js, you may need to use a third-party solution or touch controls yourself. If you plan to support XR, make sure your chosen engine has WebXR support.

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

## Selected Optimized VIVERSE Experiences

- [Alfi's Adventures](https://worlds.viverse.com/EcxNxwe) - Fine-tuned performance across mobile and desktop platforms.
- [To the Limbs](https://worlds.viverse.com/TkZWCbJ) - An interactive music video that leverages progressive loading techniques to create a stable experience.
- [Pet Rescue](https://worlds.viverse.com/MMuLWbE) - An expansive world with intuitive controls on multiple devices.
- [MetaCities: Poolside Hangout](https://worlds.viverse.com/xUKt6dt) - Impressive lighting and water effects running with smooth and stable performance.
