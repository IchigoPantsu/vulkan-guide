## Description:  
buildable [GPU Driven Rendering - Vulkan Guide](https://vkguide.dev/docs/gpudriven) for Linux.

## requirements:
### Linux (install command in Arch Linux):
  + vulkanSDK (sudo pacman -S vulkan-devel)
  + glslc (sudo pacman -S shaderc)
  + sdl2 (sudo pacman -S sdl2)
  + assimp (sudo pacman -S assimp)
### Windows:
  + TODO:

## usage (Linux):
```
$cmake -H. -BDebug -DCMAKE_BUILD_TYPE=<<Debug or Release>> -DCMAKE_EXPORT_COMPILE_COMMANDS=YES -DCMAKE_CXX_COMPILER=clang++
$cd ./<<Debug or Release>>
$make all
$make Shaders
$cd ../bin/<<Debug or Release>>
$./baker ../../assets
$mv ../../assets/assets_export ../../
$mv ../../assets_export/FlightHelmet.pfb  ../../assets_export/
$./extra
```
