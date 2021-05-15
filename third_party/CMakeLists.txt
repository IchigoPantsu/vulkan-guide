find_package(Vulkan REQUIRED)
#for nvdia-texture-tools
include(ExternalProject)

add_library(vkbootstrap STATIC)
add_library(glm INTERFACE)
add_library(vma INTERFACE)

add_library(stb_image INTERFACE)
add_library(json INTERFACE)
add_library(tinyobjloader STATIC)
add_library(spirv_reflect STATIC)

add_library(fmt_lib STATIC)

add_library(lz4 STATIC)


#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/libs")
#set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/libs")

add_subdirectory(gltf)

if (WIN32)
ExternalProject_Add(nvtools
SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/nvidia-texture-tools"
BINARY_DIR "${CMAKE_CURRENT_SOURCE_DIR}/nvlibs"
CMAKE_ARGS -DCMAKE_ARCHIVE_OUTPUT_DIRECTORY=${PROJECT_SOURCE_DIR}/libs -DCMAKE_LIBRARY_OUTPUT_DIRECTORY=${PROJECT_SOURCE_DIR}/libs -DCMAKE_CXX_STANDARD=17
)

add_library(nvtt INTERFACE)
target_include_directories(nvtt INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/nvidia-texture-tools/src/nvtt")
#set(nvttfolder "${PROJECT_SOURCE_DIR}/libs/Release")
set(nvttfolder "${PROJECT_SOURCE_DIR}/libs/$<IF:$<CONFIG:Debug>,Debug,Release>")

target_link_libraries(nvtt INTERFACE "${nvttfolder}/nvtt.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/bc6h.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/bc7.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/EtcLib.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/nvcore.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/nvimage.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/nvmath.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/nvsquish.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/nvthread.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/posh.lib")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/rg_etc1.lib")
#target_link_libraries(nvtt INTERFACE "${nvttfolder}/squish.lib")

elseif (UNIX)
set(nvttfolder "${PROJECT_SOURCE_DIR}/libs/$<IF:$<CONFIG:Debug>,Debug,Release>")

ExternalProject_Add(nvtools
SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/nvidia-texture-tools"
BINARY_DIR "${CMAKE_CURRENT_SOURCE_DIR}/nvlibs"
CMAKE_ARGS -DCMAKE_ARCHIVE_OUTPUT_DIRECTORY=${nvttfolder} -DCMAKE_LIBRARY_OUTPUT_DIRECTORY=${nvttfolder} -DCMAKE_CXX_STANDARD=17
#no Install nvidia-texture-tools
INSTALL_COMMAND cmake -E echo "skipping install step."
)

add_library(nvtt INTERFACE)
target_include_directories(nvtt INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/nvidia-texture-tools/src/nvtt")

#target_link_libraries(nvtt INTERFACE "${nvttfolder}/libnvcore.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libnvtt.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libnvimage.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libbc6h.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libbc7.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libEtcLib.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libnvcore.a")
#target_link_libraries(nvtt INTERFACE "${nvttfolder}/libnvimage.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libnvmath.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libnvsquish.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libnvthread.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/libposh.a")
target_link_libraries(nvtt INTERFACE "${nvttfolder}/librg_etc1.a")
#target_link_libraries(nvtt INTERFACE "${nvttfolder}/../libsquish.a")
endif()

#add_subdirectory(nvidia-texture-tools)

set(TRACY_DIR "${CMAKE_CURRENT_SOURCE_DIR}/tracy" 
    #CACHE FILEPATH "Path to Tracy"
)

add_library(tracy OBJECT "${TRACY_DIR}/TracyClient.cpp")
target_include_directories(tracy PUBLIC "${TRACY_DIR}")
target_compile_definitions(tracy PUBLIC TRACY_ENABLE)
target_sources(vkbootstrap PRIVATE 
    "${CMAKE_CURRENT_SOURCE_DIR}/vkbootstrap/VkBootstrap.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/vkbootstrap/VkBootstrap.cpp"
)


target_sources(spirv_reflect PRIVATE 
    "${CMAKE_CURRENT_SOURCE_DIR}/spv_reflect/spirv_reflect.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/spv_reflect/spirv_reflect.c"
)

target_sources(lz4 PRIVATE 
    "${CMAKE_CURRENT_SOURCE_DIR}/lz4/lz4.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/lz4/lz4.c"
)

target_include_directories(lz4 PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/lz4" )

target_include_directories(spirv_reflect PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/spv_reflect" )
target_include_directories(spirv_reflect PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/spv_reflect/include" )
target_include_directories(vkbootstrap PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/vkbootstrap" )
target_link_libraries(vkbootstrap PUBLIC Vulkan::Vulkan )

#both vma and glm and header only libs so we only need the include path
target_include_directories(vma INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/vma" )

target_include_directories(glm INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/glm" )

target_include_directories(json INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/nlohmann_json" )

target_sources(tinyobjloader PRIVATE 
    "${CMAKE_CURRENT_SOURCE_DIR}/tinyobjloader/tiny_obj_loader.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/tinyobjloader/tiny_obj_loader.cc"
)

target_include_directories(tinyobjloader PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/tinyobjloader" )

#
#set(sdl2_DIR "/usr" CACHE FILEPATH "Path to SDL2")
#add_library(sdl2 INTERFACE)
#target_include_directories(sdl2 INTERFACE "${sdl2_DIR}/include" "${sdl2_image_DIR}/include" )
#target_link_directories(sdl2 INTERFACE "${sdl2_DIR}/lib/x64" "${sdl2_image_DIR}/lib/x64")
#target_link_libraries(sdl2 INTERFACE SDL2 SDL2main)
#
#add_library(imgui STATIC)
#

# 

add_library(sdl2 INTERFACE)

if (WIN32)
set(sdl2_DIR "Path to SDL2" CACHE FILEPATH "Path to SDL2")
target_include_directories(sdl2 INTERFACE ${sdl2_DIR}/include ${sdl2_image_DIR}/include)
target_link_directories(sdl2 INTERFACE ${sdl2_DIR}/lib/x64 ${sdl2_image_DIR}/lib/x64)
elseif (UNIX)
set(sdl2_DIR "/usr" 
  #CACHE FILEPATH "/usr"
)
target_include_directories(sdl2 INTERFACE ${sdl2_DIR}/include/SDL2 ${sdl2_image_DIR}/include/SDL2)
target_link_directories(sdl2 INTERFACE ${sdl2_DIR}/lib ${sdl2_image_DIR}/lib)
endif()

target_link_libraries(sdl2 INTERFACE SDL2 SDL2main)

add_library(imgui STATIC)

target_include_directories(imgui PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/imgui")

target_sources(imgui PRIVATE 
"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui.h"
"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui.cpp"

"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_stdlib.h"
"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_stdlib.cpp"

"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_demo.cpp"
"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_draw.cpp"
"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_widgets.cpp"

"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_impl_vulkan.cpp"
"${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_impl_sdl.cpp"
)

target_link_libraries(imgui PUBLIC Vulkan::Vulkan sdl2 )


target_include_directories(stb_image INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/stb_image" )

add_library(assimp INTERFACE )

if (WIN32)
set(assimp_dir "${CMAKE_CURRENT_SOURCE_DIR}/assimp_build" )
set(assimp_dll "${CMAKE_CURRENT_SOURCE_DIR}/assimp_build/code/Release/assimp-vc142-mt.dll" )
set(assimp_lib "${CMAKE_CURRENT_SOURCE_DIR}/assimp_build/code/Release/assimp-vc142-mt.lib" )
target_include_directories(assimp INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/assimp/include" )

elseif (UNIX)
#set(assimp_dir "${CMAKE_CURRENT_SOURCE_DIR}/assimp" )
#set(assimp_lib "${CMAKE_CURRENT_SOURCE_DIR}/assimp/Release/bin/libassimp.so" )
set(assimp_lib "/usr/lib/libassimp.so" )
target_include_directories(assimp INTERFACE "$usr/include/assimp" )
endif()


#target_include_directories(assimp INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/assimp/include" )
target_link_libraries(assimp INTERFACE "${assimp_lib}" )

## FMT LIB
target_include_directories(fmt_lib PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/fmt_lib" )

file(GLOB FMT_FILES "${CMAKE_CURRENT_SOURCE_DIR}/fmt_lib/fmt/*" )

target_sources(fmt_lib PRIVATE "${FMT_FILES}" )

