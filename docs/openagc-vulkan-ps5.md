# OpenAGC and Vulkan-PS5 packages

The buildable graphics packages are built in this order:

1. `ps5-payload-vulkan-headers`
2. `ps5-payload-openagc`

Both recipes use immutable commit archives with SHA-256 checksums. OpenAGC is
configured for Prospero with position-independent code enabled and tests,
examples, and PSBC packaging disabled.

The `Vulkan-PS5` folder contains a pinned static-ICD recipe, but it is not yet
in `ci-libs.sh`. It requires `openagc_psbc.h` and
`libopenagc_psbc.a` to be installed in the payload SDK first. Upstream
`openagc-psbc` currently relies on external standalone compiler sources and
does not provide a self-contained immutable release archive, so this repository
does not pretend to build it from an unrelated source tree. Add Vulkan-PS5 to
the CI package list once that prerequisite has a reproducible package.

## Mesa 22.1.7 compatibility

`openagc-psbc` should remain on its upstream compiler snapshot. It does
not compile against or link to pacbrew's Mesa 22.1.7 package: it carries the
Mesa 26.2 compiler sources required by its private NIR, SPIR-V, ACO, and RADV
interfaces. Those internal interfaces are not stable across Mesa releases, so
substituting Mesa 22.1.7 is not supported without a dedicated compiler port.

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
- An installed OpenAGC PSBC Prospero header and archive are required before
  building the Vulkan-PS5 recipe.

Vulkan-PS5 uses installed dependencies from the payload sysroot. Its small
pacbrew patch replaces sibling-source assumptions with `find_package` and
installed header/library lookup, and avoids duplicating files owned by the PSBC
package.
