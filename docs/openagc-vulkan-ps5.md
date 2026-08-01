# OpenAGC and Vulkan-PS5 packages

The buildable graphics packages are built in this order:

1. `ps5-payload-vulkan-headers`
2. `ps5-payload-openagc`
3. `ps5-payload-openagc-psbc`
4. `ps5-payload-vulkan-ps5`
5. `ps5-payload-mesa-zink`
6. `ps5-payload-sdl2`

All recipes use immutable commit archives and SHA-256-checked integration
patches. OpenAGC
is configured for Prospero with position-independent code enabled and tests,
examples, and bundled PSBC packaging disabled. The dedicated
`ps5-payload-openagc-psbc` recipe builds the runtime compiler archive and
installs its public header before Vulkan-PS5 is configured.

The effective 2026-08-01 candidate revisions are OpenAGC `34cbceb`,
openagc-psbc `8d36ecc`, Vulkan-PS5 `6610cfd`, Mesa-Zink `6dbc12f`, and SDL2
`09ba4a9`. Each recipe reconstructs that revision from a reachable immutable
archive plus the SHA-256-checked patch in its package directory. These are
host-, sanitizer-, package-relocation-, Prospero-build-, FW 5.500.008-, and FW
11.600.005-qualified candidates. SDL is also published on the OpenAGC
`release-2.30.x-ps5` branch, which the SDL2 recipe uses as its immutable source
repository. The SDL2 extension recipes separately pin the current upstream
SDL2 maintenance revisions listed in their `PKGBUILD` files; they must not
follow the incompatible SDL3 default branches.

The exact artifacts used by three guarded FW 11.600.005 passes and the two
immediate FW 5.500.008 replays are:

- `testps5zink`: `95da10acf89da3e35865890874034b8bffef1c563417309a0e4bb98404540ad9`
- `libvulkan_ps5.so`: `eacc4cf3dd1c15983e9f78482d65b14250d073a161c0b02433087eaeb5b6d271`
- `libEGL.so.1.0.0`: `0d2922b30b3dbbe25f060331043bb4a4732272d0813023568381306528913fc1`
- `libgallium-26.3.0-devel.so`: `75f3c3fcd229387557d4649af9eee293ac485feeaea1904e48649370565b6b5f`

All five runs returned exact RGBA `64,128,191,255`, presented, released every
native child, self-exited, and immediately relaunched without reboot. The Mesa
pin retains `$ORIGIN` on `libEGL` and removes Gallium's redundant identical
RUNPATH, the packaging correction required for deterministic FW 5.50 loading.

The current OpenAGC pin exposes explicit gfx1013 Wave32 and Wave64 compute
dispatch modes. The matching Vulkan-PS5 pin completes the native runtime
migration and builds both the static implementation and `libvulkan_ps5.so`
for Mesa's runtime loader. It includes hardware-qualified shader
cull distance, extended image gather, fragment and vertex-pipeline stores and
atomics, and variable-pointer feature reporting. These two pins are updated as
a pair because Vulkan-PS5 consumes OpenAGC's public command-state API.

The OpenAGC 0.2.0 pin also provides `agcDriverShutdown` and releases
system-flexible allocations with `sceKernelReleaseFlexibleMemory`. SDL2 calls
that lifecycle boundary on normal destruction and partial renderer creation
failure, preventing repeated WebSrv launches from consuming the global
flexible-memory quota. The paired SDL2 pin uses GPU scanout copies, bounded
submission failure latching, and exact renderer-selection probes. The
Vulkan-PS5 pin fixes retirement of the active VideoOut image before swapchain
presentation resources are destroyed.

The effective `openagc-psbc` source contains its Mesa compiler sources and
generated-header inputs directly. It does not use sibling source repositories,
symlinks, submodules, or pacbrew's Mesa package. Its checked integration patch
adds the Zink descriptor/deref lowering proven by the pinned compiler commit;
the Prospero build regenerates the derived Mesa sources and produces
`libopenagc_psbc.a`. Vulkan-PS5 and the compiler package are therefore both
enabled in `ci-libs.sh` in dependency order.

## Mesa runtime separation

`openagc-psbc` remains on its upstream compiler snapshot. It does not compile
against or link to pacbrew's Mesa 22.1.7 package: its immutable source archive
carries the Mesa 26.2 compiler sources required by its private NIR, SPIR-V,
ACO, and RADV interfaces. Those internal interfaces are not stable across Mesa
releases, so substituting Mesa 22.1.7 is not supported without a dedicated
compiler port.

The existing `ps5-payload-mesa` 22.1.7 package remains the independent
swrast/OSMesa implementation. The new `ps5-payload-mesa-zink` package pins the
Mesa 26.3 development revision used by SDL's strict Zink work and installs only
`libEGL.so` and its versioned Gallium/Zink module. It deliberately does not
install GL/EGL headers or `libGL.so`, so it does not replace files owned by the
OSMesa package.

Installing both packages is supported. SDL still defaults to OSMesa; selecting
Zink requires the exact `SDL_HINT_PS5_OPENGL_DRIVER=zink` hint. The Zink path
loads the separately packaged `libvulkan_ps5.so`, while openagc-psbc continues
to build its private Mesa 26.2 compiler snapshot. These three Mesa-derived
components do not link to each other's private compiler interfaces.

## Requirements

- `ps5-payload-sdk` supplies the Prospero C/C++ toolchain and CMake wrapper.
- `ps5-payload-libcxx` supplies libc++, libc++abi, and libunwind for an
  externally installed PSBC runtime.
- `ps5-payload-vulkan-headers` supplies Vulkan headers and its CMake config.
- `ps5-payload-openagc` supplies the public graphics and VideoOut library.
- `ps5-payload-openagc-psbc` supplies the Prospero runtime compiler header and
  archive required by Vulkan-PS5.
- `ps5-payload-vulkan-ps5` supplies the shared ICD loaded by Zink.
- `ps5-payload-mesa-zink` supplies the pinned EGL/Zink runtime loaded by SDL.

Vulkan-PS5 uses installed dependencies from the payload sysroot. Its small
pacbrew patch replaces sibling-source assumptions with `find_package` and
installed header/library lookup, and avoids duplicating files owned by the PSBC
package.

## OpenAGC SDL2 consumers

The `ps5-payload-sdl2` recipe uses an immutable SDL source archive plus the
SHA-256-checked OpenAGC/Zink integration patch and builds it with
`SDL_PS5_OPENAGC=ON` and `SDL_PS5_ZINK=ON`. It depends on OpenAGC,
Vulkan-PS5, and Mesa-Zink, uses the installed `OpenAGCConfig.cmake`, and installs
`prospero-sdl2-config` for packages that use SDL's traditional configure
interface. OpenAGC remains responsible for deciding whether the target can use
its native renderer. SDL retains its software and OSMesa paths; the Zink path
is explicit, fail-closed, and qualified for EGL readback, visible WSI
presentation, complete teardown, and immediate relaunch on FW 5.500.008 and FW
11.600.005.

`ci-libs.sh` builds Mesa-Zink after Vulkan-PS5 and before SDL2. The older Mesa
22.1.7 package remains later in the list because it supplies the separate
OSMesa fallback without owning any Mesa-Zink runtime file.

The SDL2 extension recipes (`SDL2_gfx`, `SDL2_image`, `SDL2_mixer`, `SDL2_net`,
and `SDL2_ttf`) select that wrapper explicitly. `SDL2_kitchensink` and the
CMake-based consumers `devilutionx`, `imgui`, `openal`, and `rmlui` select the
installed SDL2 CMake package explicitly. The Autotools consumers `love`,
`mednafen`, and `scummvm` use the Prospero wrapper, while `ffmpeg` now enables
its SDL2 integration and declares SDL2 as a runtime dependency.

The six `SDL2_*` recipes are pinned to the latest commit of their SDL2,
main, or master maintenance branch as appropriate, with immutable archive
checksums. On 2026-08-01 the four libsdl-org recipes were verified against the
`SDL2` branch rather than their incompatible SDL3 default branch; their
existing pins were already current. SDL2_gfx was also already at upstream
HEAD, while SDL2_kitchensink advanced to `b01841c`. SDL2_gfx's latest commit
no longer carries an Autotools build, so
its recipe supplies a minimal checked CMake build for the four upstream source
files and installs the same public headers and static `libSDL2_gfx.a` surface.

LakeSnes, offact, FBNeo, and EDuke32 already select
`prospero-sdl2-config` in their upstream PS5 build files, so installing this
repository's `ps5-payload-sdl2` package routes them to the same OpenAGC-enabled
SDL without recipe patches. LBreakoutHD no longer replaces
`SDL_RENDERER_ACCELERATED` with `SDL_RENDERER_SOFTWARE`, allowing SDL to select
the OpenAGC renderer on supported targets.
