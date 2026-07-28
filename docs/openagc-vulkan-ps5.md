# OpenAGC and Vulkan-PS5 packages

The buildable graphics packages are built in this order:

1. `ps5-payload-vulkan-headers`
2. `ps5-payload-openagc`
3. `ps5-payload-openagc-psbc`
4. `ps5-payload-vulkan-ps5`

All four recipes use immutable commit archives with SHA-256 checksums. OpenAGC
is configured for Prospero with position-independent code enabled and tests,
examples, and bundled PSBC packaging disabled. The dedicated
`ps5-payload-openagc-psbc` recipe builds the runtime compiler archive and
installs its public header before Vulkan-PS5 is configured.

The current OpenAGC pin exposes explicit gfx1013 Wave32 and Wave64 compute
dispatch modes. The matching Vulkan-PS5 pin includes hardware-qualified shader
cull distance, extended image gather, fragment and vertex-pipeline stores and
atomics, and variable-pointer feature reporting. These two pins are updated as
a pair because Vulkan-PS5 consumes OpenAGC's public command-state API.

The pinned `openagc-psbc` archive now contains its Mesa compiler sources and
generated-header inputs directly. It does not use sibling source repositories,
symlinks, submodules, or pacbrew's Mesa package. Its Prospero build regenerates
the derived Mesa sources and produces `libopenagc_psbc.a` from the immutable
archive. Vulkan-PS5 and the compiler package are therefore both enabled in
`ci-libs.sh` in dependency order.

## Mesa 22.1.7 compatibility

`openagc-psbc` remains on its upstream compiler snapshot. It does not compile
against or link to pacbrew's Mesa 22.1.7 package: its immutable source archive
carries the Mesa 26.2 compiler sources required by its private NIR, SPIR-V,
ACO, and RADV interfaces. Those internal interfaces are not stable across Mesa
releases, so substituting Mesa 22.1.7 is not supported without a dedicated
compiler port.

The existing Mesa 22.1.7 package remains the independent swrast/OSMesa
implementation. Installing both packages is supported because neither package
declares or replaces the other's files. Linking both implementations into the
same process has not been validated by these recipes and should be tested on
the target application before being treated as supported.

## Requirements

- `ps5-payload-sdk` supplies the Prospero C/C++ toolchain and CMake wrapper.
- `ps5-payload-libcxx` supplies libc++, libc++abi, and libunwind for an
  externally installed PSBC runtime.
- `ps5-payload-vulkan-headers` supplies Vulkan headers and its CMake config.
- `ps5-payload-openagc` supplies the public graphics and VideoOut library.
- `ps5-payload-openagc-psbc` supplies the Prospero runtime compiler header and
  archive required by Vulkan-PS5.

Vulkan-PS5 uses installed dependencies from the payload sysroot. Its small
pacbrew patch replaces sibling-source assumptions with `find_package` and
installed header/library lookup, and avoids duplicating files owned by the PSBC
package.

## OpenAGC SDL2 consumers

The `ps5-payload-sdl2` recipe uses an immutable OpenAGC/SDL source archive and
builds it with `SDL_PS5_OPENAGC=ON`. It depends on
`ps5-payload-openagc`, uses the installed `OpenAGCConfig.cmake`, and installs
`prospero-sdl2-config` for packages that use SDL's traditional configure
interface. OpenAGC remains responsible for deciding whether the target can use
its renderer; SDL retains its software presentation fallback when accelerated
renderer creation fails.

The SDL2 extension recipes (`SDL2_gfx`, `SDL2_image`, `SDL2_mixer`, `SDL2_net`,
and `SDL2_ttf`) select that wrapper explicitly. `SDL2_kitchensink` and the
CMake-based consumers `devilutionx`, `imgui`, `openal`, and `rmlui` select the
installed SDL2 CMake package explicitly. The Autotools consumers `love`,
`mednafen`, and `scummvm` use the Prospero wrapper, while `ffmpeg` now enables
its SDL2 integration and declares SDL2 as a runtime dependency.

LakeSnes, offact, FBNeo, and EDuke32 already select
`prospero-sdl2-config` in their upstream PS5 build files, so installing this
repository's `ps5-payload-sdl2` package routes them to the same OpenAGC-enabled
SDL without recipe patches. LBreakoutHD no longer replaces
`SDL_RENDERER_ACCELERATED` with `SDL_RENDERER_SOFTWARE`, allowing SDL to select
the OpenAGC renderer on supported targets.
