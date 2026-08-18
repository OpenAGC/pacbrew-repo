# OpenAL Soft SDL3 PS5 Port Status

This package builds OpenAL Soft `1.25.2` with only its SDL3 audio backend
enabled. The backend uses `ps5-payload-sdl3`, whose PS5 SDL3 audio driver sends
PCM data through `SceAudioOut`.

The package is static and embeds the HRTF data. SDL2, desktop audio backends,
utilities, examples, tests, and runtime configuration files are disabled so
the package does not pull the existing SDL2 OpenAL stack into an SDL3-based
LÖVE build.

The installed OpenAL CMake configuration imports SDL3 before its static target,
allowing downstream projects to use `find_package(OpenAL CONFIG)` and
`OpenAL::OpenAL` directly.

Validated by the package recipe:

- OpenAL Soft source tarball checksum is pinned
- SDL3 backend is required at configure time
- SDL2 backend is explicitly disabled
- PS5 cross-build and install are the required validation targets

Still pending on PS5 hardware or a deployable test harness:

- opening an OpenAL playback device through SDL3
- generated PCM tone playback through `SceAudioOut`
- OpenAL source creation, positional attenuation, and cleanup
- LÖVE audio initialization using this package
