# SDL3 PS5 Port Status

This package builds SDL3 from the `OpenAGC/SDL3` fork at commit
`642d5f82b`.

Implemented in the fork:

- PS5 VideoOut software framebuffer window and presentation path
- PS5 keyboard, IME, controller, and joystick event paths
- SceAudioOut audio backend
- PS5 filesystem and URL backends
- OSMesa context support for software OpenGL
- installable SDL3 headers, pkg-config metadata, and CMake targets

The pacbrew build uses `ps5-payload-mesa` (Mesa 22.1.7, OSMesa and `swrast`)
for the interim OpenGL path. Vulkan is deliberately disabled until
`ps5-vulkan` has a stable PS5 runtime and WSI interface.

Validated outside pacbrew packaging:

- Prospero cross-configuration completed successfully
- `libSDL3.a` built successfully
- CMake install/export completed successfully
- exported static target contains the required Sce library dependencies

Still pending on hardware or a deployable PS5 test harness:

- window presentation and framebuffer smoke test
- generated SceAudioOut tone
- keyboard/controller event smoke test
- OSMesa triangle rendering
- external PS5 CMake consumer build and link
