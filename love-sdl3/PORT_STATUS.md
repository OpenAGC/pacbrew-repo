# LÖVE SDL3 PS5 Port Status

This package is a separate SDL3-based LÖVE slice. The existing `love` package
remains unchanged while this target is validated.

## Validated

- LÖVE commit `853f1cad4bb65d63f02dbcd3fa31b0c622d3abbc` (12.0)
- SDL3 and OpenAL Soft built with the PS5 SDK
- LuaJIT, FreeType, HarfBuzz, ModPlug, Theora, Vorbis, Ogg, zlib, bzip2, and
  libpng from the active SDK prefix
- Static PS5 build and install of the `love` executable

## Interim limitations

- Graphics uses the SDL3/OpenGL path; Vulkan is disabled until `ps5-vulkan` is
  ready for integration.
- `liblove` is built static because the SDK LuaJIT archive contains generated
  unwind relocations that cannot be linked into a shared library.
- The bundled Lua HTTPS module uses its link-time no-backend loader on PS5.
  The pacbrew `curl` recipe is configured `--with-openssl` and is TLS-capable,
  but curl is not yet a dependency of this slice or wired into the LÖVE HTTPS
  module. Curl, libpsl, and OpenSSL are now installed in the active SDK.

## Pending hardware validation

- PS5 launch and window/input smoke test
- SDL3 audio/OpenAL playback test
- OpenGL renderer behavior on the PS5 GPU
- HTTPS once a PS5 curl/TLS backend is selected
