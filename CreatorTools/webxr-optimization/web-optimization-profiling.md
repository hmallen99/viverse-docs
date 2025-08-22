---
description: Resources for profiling 3D web applications and rendering engines
---

# Profiling and Testing 3D Web Experiences

## Core Browser Profiling Tools

### Overview of Tools

Scripting Performance:

Native Traces:

Memory Usage:

Debugging:

Networking and Asset Loading Profiling:

Device Emulation:

Remote Inspection:

WebXR Emulation:

- [Meta Immersive Web Emulator](https://developers.meta.com/horizon/blog/webxr-development-immersive-web-emulator/)

## Profiling Major Web Browsers and Devices

### Edge (Desktop)

Core Tooling:

- [Performance Profiling](https://learn.microsoft.com/en-us/microsoft-edge/devtools/performance/overview)
- [Debugger](https://learn.microsoft.com/en-us/microsoft-edge/devtools/javascript/)
- [Network Profiling](https://learn.microsoft.com/en-us/microsoft-edge/devtools/network/)

Additional Tooling

- [Immersive Web Emulator Plugin](https://microsoftedge.microsoft.com/addons/detail/immersive-web-emulator/hhlkbhldhffpeibcfggfndbkfohndamj)

Mobile Tooling

- [Device Emulator](https://learn.microsoft.com/en-us/microsoft-edge/devtools/device-mode/)

### Chrome (Desktop & Mobile)

Core tools:

- [Performance Profiling](https://developer.chrome.com/docs/devtools/performance/overview)
- [Debugger](https://developer.chrome.com/docs/devtools/javascript)
- [Network Profiling](https://developer.chrome.com/docs/devtools/network/overview)

Additional tooling:

- [Tracing Tool](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/): Provides more introspection into the native side of chrome, showing JavaScript engine traces and GPU tracing.
- [Lighthouse](https://developer.chrome.com/docs/devtools/lighthouse): Tools for optimizing app start time and file size.
- [Immersive Web Emulator Plugin](https://chromewebstore.google.com/detail/immersive-web-emulator/cgffilbpcibhmcfbgggfhfolhkfbhmik?pli=1)
- [Desktop Frame Rate](https://devtoolstips.org/tips/en/display-current-framerate/)

Mobile Tools:

- [Remote Debugging](https://developer.chrome.com/docs/devtools/remote-debugging/local-server)
- [Device Emulator](https://developer.chrome.com/docs/devtools/device-mode/)

### Firefox (Desktop)

Core Tools:

- [Network Profiling](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/index.html)
- [Debugger](https://firefox-source-docs.mozilla.org/devtools-user/debugger/index.html)
- [Performance Profiling](https://profiler.firefox.com/docs/#/)

Mobile Tools:

- [Device Emulator](https://firefox-source-docs.mozilla.org/devtools-user/responsive_design_mode/index.html)

### Safari (Desktop & Mobile)

Core Tools:

- [Tools Overview](https://developer.apple.com/safari/tools/#current)
- [Performance Profiler](https://support.apple.com/guide/safari-developer/performance-overview-devf7aaca927/mac)
- [Debugger](https://support.apple.com/guide/safari-developer/debugging-overview-devd24689f72/mac)

Mobile Tools:

- [Remote Debugging](https://dev.to/nimajafari/remote-debugging-using-safari-on-ios-devices-with-macos-16p5)

## Strengths and Weakness of Web Rendering Engines

### Unity

Overview:
- Feature Set: 2D, 3D, Physics, Touch controls, PBR
- Licensing: Closed source, licensed
- Learning: Extensive documentation, huge community
- Developer Experience: Great Editor, great for teams, additional build/compile step
- Third-party integrations: Does not integrate well with JS libraries
- Profiling: Extensive native tools, fewer Web tools
- Positives: Good performance, scalability
- Negatives: Large file sizes
- Neutrals: Native engine, write in C#, opinionated engine

General Resources:

- [Documentation](https://docs.unity3d.com/Manual/index.html)
- [Web Documentation](https://docs.unity3d.com/Manual/webgl.html)
- [Publishing Unity Web Experiences](https://docs.unity3d.com/Manual/webgl-gettingstarted.html)
- [Building Unity Web Experiences](https://docs.unity3d.com/Manual/webgl-building-distribution.html)

Optimization Resources:

- [Technical Limitations](https://docs.unity3d.com/Manual/webgl-technical-overview.html)
- [Web Memory Optimization](https://docs.unity3d.com/Manual/webgl-memory.html)
- [Web Graphics Recommendations](https://docs.unity3d.com/Manual/web-graphics-apis-intro.html)
- [Texture Compression](https://docs.unity3d.com/Manual/webgl-texture-compression.html)
- [WebAssembly Optimization](https://docs.unity3d.com/Manual/wasm-2023-features.html)
- [Optimizing Web Builds](https://docs.unity3d.com/Manual/web-optimization.html)
- [Optimizing Web Builds for Mobile](https://docs.unity3d.com/Manual/web-optimization-mobile.html)
- [General Unity Optimization](https://docs.unity3d.com/Manual/analysis.html)
- [Optimizing Draw Calls](https://docs.unity3d.com/Manual/optimizing-draw-calls.html)
- [Optimizing Shaders](https://docs.unity3d.com/Manual/SL-ShaderPerformance.html)

Profiling Tools:

- [Unity Profiler](https://docs.unity3d.com/Manual/Profiler.html)
- [Other Profiling Tools](https://docs.unity3d.com/Manual/performance-profiling-tools.html)
- [Graphics Performance Profiling](https://docs.unity3d.com/Manual/graphics-performance-profiling.html)
- [Debug Web Builds](https://docs.unity3d.com/6000.3/Documentation/Manual/webgl-debugging.html)
- [How to profile Web Builds](https://unity.com/how-to/profile-optimize-web-build#the-importance-of-profiling)

### Three.js

Overview:
- Feature Set: Solid 3D by default, XR support. Extensive third-party libraries
- Licensing: Open Source, Free
- Learning: Good documentation, large community, many examples, less structured onboarding
- Developer Experience: Poor editor/playground support, integrates with Git
- Third-party integrations: Integrates very well with React, many third party libraries adding features
- Profiling: First-party plugin, integrates well with the browser
- Positives: Decent performance, small file sizes
- Negatives: easy to hit performance ceiling, barebones feature set by default
- Neutrals: JavaScript engine, not opinionated

General Resources:
- [Documentation](https://threejs.org/manual/#en/creating-a-scene)
- [DevTools Plugin](https://chromewebstore.google.com/detail/threejs-devtools/jechbjkglifdaldbdbigibihfaclnkbo)
- [Forum](https://discourse.threejs.org/)
- [Learning Resources](https://threejsresources.com/)

Optimization Guides:
- [Rendering Many Objects](https://threejs.org/manual/#en/optimize-lots-of-objects)
- [Rendering Many Animated Objects](https://threejs.org/manual/#en/optimize-lots-of-objects-animated)
- [Leveraging Offscreen Canvas](https://threejs.org/manual/#en/offscreencanvas)
- [Memory Management](https://threejs.org/manual/#en/cleanup)
- [InstancedMesh](https://threejs.org/docs/?q=inst#api/en/objects/InstancedMesh)
- [Building Efficient Three.js Scenes](https://tympanus.net/codrops/2025/02/11/building-efficient-three-js-scenes-optimize-performance-while-maintaining-quality/)

Profiling and Debugging Resources:
- [JavaScript Debugging](https://threejs.org/manual/#en/debugging-javascript)
- [GLSL Debugging](https://threejs.org/manual/#en/debugging-glsl)
- [Stats.js](https://github.com/mrdoob/stats.js/): JavaScript Performance Monitor

### PlayCanvas

Overview:
- Feature Set: 2D, 3D, Physics, XR, UI, Touch Input, GLTF Animation. Extensive third-party libraries
- Licensing: Open Source, free engine, paid integrations available
- Learning: Good documentation, decent onboarding, smaller community, learning pathways for multiple types of developers
- Developer Experience: Paid editor support, integrates with source control
- Third-party integrations: Integrates very well with React, few third party libraries for adding features
- Profiling: First-party plugin, integrates well with the browser
- Positives: Good performance, medium file sizes
- Negatives: Some paid editor features
- Neutrals: JavaScript engine, ECS architecture

General Resources:
- [PlayCanvas Docs](https://developer.playcanvas.com/)
- [Tutorials](https://developer.playcanvas.com/tutorials/)
- [Editor](https://playcanvas.com/products/editor)

Optimization Resources:
- [Optimization](https://developer.playcanvas.com/user-manual/optimization/)
- [Texture Compression](https://developer.playcanvas.com/user-manual/optimization/texture-compression/)
- [Batching](https://developer.playcanvas.com/user-manual/graphics/advanced-rendering/batching/)
- [Hardware Instancing](https://developer.playcanvas.com/user-manual/graphics/advanced-rendering/hardware-instancing/)
- [Indirect Drawing](https://developer.playcanvas.com/user-manual/graphics/advanced-rendering/indirect-drawing/)
- [Optimizing Load Time](https://developer.playcanvas.com/user-manual/optimization/load-time/)
- [Optimizing WebXR](https://developer.playcanvas.com/user-manual/xr/optimizing-webxr/)

Debugging Resources:

- [Browser Dev Tools](https://developer.playcanvas.com/user-manual/scripting/debugging/browser-dev-tools/)
- [GPU Profiling](https://developer.playcanvas.com/user-manual/optimization/gpu-profiling/)
- [Profiler](https://developer.playcanvas.com/user-manual/optimization/profiler/)

### Babylon.js

Overview:
- Feature Set: 3D, Physics, GLTF animations, XR, UI, Node editor for shaders
- Licensing: Open Source, Free
- Learning: Good documentation, decent onboarding, smaller community, medium barrier to entry
- Developer Experience: Decent playground, very simple editor, no first-party version control
- Third-party integrations: Integrates with React, Angular, Vue. Some third-party libraries
- Profiling: First-party plugin, integrates well with the browser
- Positives: Decent performance by default, high performance ceiling, First class WebXR, Large feature set
- Negatives: larger file sizes for a JavaScript Engine
- Neutrals: JavaScript engine, not opinionated

General Resources:
- [Documentation](https://doc.babylonjs.com/)
- [Playground](https://playground.babylonjs.com/)

Optimization Resources:
- [Optimization Guide](https://doc.babylonjs.com/features/featuresDeepDive/scene/optimize_your_scene)
- [Octree Optimization](https://doc.babylonjs.com/features/featuresDeepDive/scene/optimizeOctrees/)
- [Resource Caching](https://doc.babylonjs.com/features/featuresDeepDive/scene/optimizeCached/)
- [SceneOptimizer](https://doc.babylonjs.com/features/featuresDeepDive/scene/sceneOptimizer/)
- [Offscreen Canvas](https://doc.babylonjs.com/features/featuresDeepDive/scene/offscreenCanvas/)
- [Creating Loading Screens](https://doc.babylonjs.com/features/featuresDeepDive/scene/customLoadingScreen/)
- [Instancing](https://doc.babylonjs.com/features/featuresDeepDive/mesh/copies/)

Profiling and Debugging Resources:
- [Spector](https://github.com/BabylonJS/Spector.js)
- [Inspector](https://doc.babylonjs.com/toolsAndResources/inspector/)
