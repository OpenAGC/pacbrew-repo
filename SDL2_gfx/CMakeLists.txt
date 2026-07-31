cmake_minimum_required(VERSION 3.16)
project(SDL2_gfx VERSION 1.0.4 LANGUAGES C)

include(GNUInstallDirs)
find_path(SDL2_INCLUDE_DIR SDL.h REQUIRED PATH_SUFFIXES SDL2)

add_library(SDL2_gfx STATIC
  SDL2_framerate.c
  SDL2_gfxPrimitives.c
  SDL2_imageFilter.c
  SDL2_rotozoom.c)
target_include_directories(SDL2_gfx PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${SDL2_INCLUDE_DIR}>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/SDL2>)

install(TARGETS SDL2_gfx ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES
  SDL2_framerate.h
  SDL2_gfxPrimitives.h
  SDL2_gfxPrimitives_font.h
  SDL2_imageFilter.h
  SDL2_rotozoom.h
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/SDL2)
