## Install Pixi
See: https://pixi.sh/latest/#installation


## Build Static Library (Windows)
To build Coal as a static library for use in other C++ projects (avoiding DLL conflicts):

### 1. Clean and Configure
```bash
# Clean any previous build
pixi run -e qhull clean

# Configure with Visual Studio generator (handles Windows SDK properly)
pixi run -e qhull configure
```

### 2. Build Static Library
```bash
# Build only the static library
cd build && cmake --build . --target coal_static --config Release
```

### 4. Integration with Other Projects

**Directory Structure:**
```
your-project/
├── libs/
│   └── coal/
│       ├── include/     # Coal headers
│       └── lib/         # coal_static.lib + Boost libraries
```

**Copy these files:**
1. `build/lib/Release/coal_static.lib` to `your-project/libs/coal/lib`
2. All headers under `include/coal` to `your-project/libs/coal/include`
3. Generated headers under `build/include` to `your-project/libs/coal/include`
4. Eigen lib (header only) under `.pixi\envs\qhull\Library\include\eigen3` to `your-project\libs\eigen3`
5. Boost lib under `.pixi/envs/default/Library/include/boost` to `/your-project/libs/boost`
6. **Boost libraries:**
   ```bash
   copy ".pixi\envs\qhull\Library\lib\boost_chrono.lib" "your-project\libs\coal\lib\"
   copy ".pixi\envs\qhull\Library\lib\boost_thread.lib" "your-project\libs\coal\lib\"
   copy ".pixi\envs\qhull\Library\lib\boost_date_time.lib" "your-project\libs\coal\lib\"
   copy ".pixi\envs\qhull\Library\lib\boost_serialization.lib" "your-project\libs\coal\lib\"
   copy ".pixi\envs\qhull\Library\lib\boost_filesystem.lib" "your-project\libs\coal\lib\"
   copy ".pixi\envs\qhull\Library\lib\boost_system.lib" "your-project\libs\coal\lib\"
   ```
7. assimp lib
   copy ".pixi\envs\qhull\Library\lib\assimp.lib" "your-project\libs\coal\lib\"
8. qhull lib and dll
   copy ".pixi\envs\qhull\Library\lib\qhull_r.lib" "your-project\libs\coal\lib\"
   copy ".pixi\envs\qhull\Library\lib\qhullcpp.lib" "your-project\libs\coal\lib\"
   copy ".pixi\envs\qhull\Library\bin\qhull_r.dll" "your-project\libs\coal\bin\"

**CMakeLists.txt Integration:**
```cmake
# tell coal that we are using static library build
target_compile_definitions(${PROJECT_NAME} PRIVATE COAL_STATIC)

# Add include directory
include_directories(${CMAKE_SOURCE_DIR}/libs/coal/include)
include_directories(${CMAKE_SOURCE_DIR}/libs/boost)
include_directories(${CMAKE_SOURCE_DIR}/libs/eigen3)

# Add link libraries
target_link_libraries(${PROJECT_NAME} PRIVATE
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/coal_static.lib
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/boost_chrono.lib
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/boost_thread.lib
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/boost_date_time.lib
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/boost_serialization.lib
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/boost_filesystem.lib
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/assimp.lib
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/qhull_r.lib
  ${CMAKE_SOURCE_DIR}/libs/coal/lib/qhullcpp.lib
)

# copy required files after build
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
    "${CMAKE_SOURCE_DIR}/libs/coal/bin/assimp.dll"
    $<TARGET_FILE_DIR:${PROJECT_NAME}>
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
    "${CMAKE_SOURCE_DIR}/libs/coal/bin/boost_filesystem.dll"
    $<TARGET_FILE_DIR:${PROJECT_NAME}>
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
    "${CMAKE_SOURCE_DIR}/libs/coal/bin/qhull_r.dll"
    $<TARGET_FILE_DIR:${PROJECT_NAME}>
)
```
