# CMakeList.txt : CMake project for vulkan_guide, include source and define
# project specific logic here.
#
cmake_minimum_required (VERSION 3.8)

project ("vulkan_guide")

find_package(Vulkan REQUIRED)

add_subdirectory(third_party)

#set (CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/bin")
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/bin/$<IF:$<CONFIG:Debug>,Debug,Release>")

add_subdirectory(assetlib)
add_subdirectory(asset-baker)
add_subdirectory(extra-engine)

if (WIN32)
  if (${CMAKE_HOST_SYSTEM_PROCESSOR} STREQUAL "AMD64")
    set(GLSL_VALIDATOR "$ENV{VULKAN_SDK}/Bin/glslangValidator.exe")
  else()
    set(GLSL_VALIDATOR "$ENV{VULKAN_SDK}/Bin32/glslangValidator.exe")
  endif()
elseif (UNIX)
  set(GLSL_VALIDATOR "/usr/bin/glslc")
endif()

file(GLOB_RECURSE GLSL_SOURCE_FILES
    "${PROJECT_SOURCE_DIR}/shaders/*.frag"  
    "${PROJECT_SOURCE_DIR}/shaders/*.vert"
    "${PROJECT_SOURCE_DIR}/shaders/*.comp"
)

foreach(GLSL ${GLSL_SOURCE_FILES})
  message(STATUS "BUILDING SHADER")
  get_filename_component(FILE_NAME ${GLSL} NAME)
  #set(SPIRV "${PROJECT_SOURCE_DIR}/shaders/${FILE_NAME}.spv")
  #set(SPIRV "$<TARGET_FILE_DIR:extra>/../../shaders/${FILE_NAME}.spv")
  set(SPIRV "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/../../shaders/${FILE_NAME}.spv")
  message(STATUS ${GLSL})
  if (WIN32)
    add_custom_command(
      OUTPUT  ${SPIRV}
      COMMAND ${GLSL_VALIDATOR} -V ${GLSL} -o ${SPIRV}
      DEPENDS ${GLSL})
  elseif (UNIX)
    add_custom_command(
      OUTPUT  ${SPIRV}
      COMMAND ${GLSL_VALIDATOR} ${GLSL} -o ${SPIRV}
      DEPENDS ${GLSL})
  endif()
  list(APPEND SPIRV_BINARY_FILES ${SPIRV})
endforeach(GLSL)

add_custom_target(
    Shaders 
    DEPENDS ${SPIRV_BINARY_FILES}
	  SOURCES ${GLSL_SOURCE_FILES}
)

#[Compiling GLSL to SPIR-V from CMake](https://gist.github.com/evilactually/a0d191701cb48f157b05be7f74d79396)
