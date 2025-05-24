# Visual Studio 2022 Lua Complete Build Guide

## Overview
To build a complete Lua development environment, you need:
- `lua.exe` - Lua interpreter (you already have this)
- `lua.dll` - Dynamic library for linking
- `lua.lib` - Import library for Visual Studio projects
- `luac.exe` - Lua compiler (optional)

## Method 1: Using Visual Studio Project Files

### Step 1: Download Lua Source
1. Download Lua source from: https://www.lua.org/download.html
2. Extract to a directory (e.g., `C:\lua-src\lua-5.4.7`)

### Step 2: Create Visual Studio Solution

#### Create DLL Project:
1. Open Visual Studio 2022
2. Create New Project → "Dynamic-Link Library (DLL)"
3. Name: `lua` or `lua54`
4. Add all Lua source files EXCEPT `lua.c` and `luac.c`:
   ```
   lapi.c      lcode.c     lctype.c    ldebug.c    ldo.c
   ldump.c     lfunc.c     lgc.c       llex.c      lmem.c
   lobject.c   lopcodes.c  lparser.c   lstate.c    lstring.c
   ltable.c    ltm.c       lundump.c   lvm.c       lzio.c
   lauxlib.c   lbaselib.c  lcorolib.c  ldblib.c    liolib.c
   lmathlib.c  loslib.c    lstrlib.c   ltablib.c   lutf8lib.c
   loadlib.c   linit.c
   ```

#### Project Configuration:
1. **Configuration Properties → General**:
   - Configuration Type: Dynamic Library (.dll)
   - Target Name: lua54 (or lua)

2. **C/C++ → Preprocessor → Preprocessor Definitions**:
   Add: `LUA_BUILD_AS_DLL`

3. **Linker → General**:
   - Generate Import Library: Yes

### Step 3: Create EXE Project for lua.exe
1. Create New Project → "Console App"
2. Name: `lua-interpreter`
3. Add only `lua.c` to this project
4. Link against the DLL project

### Step 4: Build Configuration
1. Set Solution Configuration to "Release"
2. Build Solution (F7)

## Method 2: Using CMake (Recommended)

### Step 1: Create CMakeLists.txt
Create this file in your Lua source directory:

```cmake
cmake_minimum_required(VERSION 3.10)
project(lua VERSION 5.4.7)

# Set C standard
set(CMAKE_C_STANDARD 99)

# Lua core library sources
set(LUA_CORE_SOURCES
    lapi.c lcode.c lctype.c ldebug.c ldo.c ldump.c
    lfunc.c lgc.c llex.c lmem.c lobject.c lopcodes.c
    lparser.c lstate.c lstring.c ltable.c ltm.c
    lundump.c lvm.c lzio.c
)

# Lua library sources
set(LUA_LIB_SOURCES
    lauxlib.c lbaselib.c lcorolib.c ldblib.c liolib.c
    lmathlib.c loslib.c lstrlib.c ltablib.c lutf8lib.c
    loadlib.c linit.c
)

# Create shared library (DLL)
add_library(lua SHARED ${LUA_CORE_SOURCES} ${LUA_LIB_SOURCES})
target_compile_definitions(lua PRIVATE LUA_BUILD_AS_DLL)

# Create static library
add_library(lua_static STATIC ${LUA_CORE_SOURCES} ${LUA_LIB_SOURCES})
set_target_properties(lua_static PROPERTIES OUTPUT_NAME lua)

# Create lua interpreter
add_executable(lua_exe lua.c)
target_link_libraries(lua_exe lua)
set_target_properties(lua_exe PROPERTIES OUTPUT_NAME lua)

# Create lua compiler
add_executable(luac luac.c)
target_link_libraries(luac lua_static)

# Installation
install(TARGETS lua lua_static lua_exe luac
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

install(FILES lua.h luaconf.h lualib.h lauxlib.h lua.hpp
    DESTINATION include
)
```

### Step 2: Build with CMake
```cmd
cd C:\lua-src\lua-5.4.7
mkdir build
cd build
cmake .. -G "Visual Studio 17 2022" -A x64
cmake --build . --config Release
cmake --install . --prefix C:\lua
```

## Method 3: Manual MSBuild (Advanced)

### Create lua.vcxproj for DLL:
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <ConfigurationType>DynamicLibrary</ConfigurationType>
    <PlatformToolset>v143</PlatformToolset>
    <TargetName>lua54</TargetName>
  </PropertyGroup>
  <ItemGroup>
    <ClCompile Include="*.c">
      <ExcludedFromBuild Condition="'%(Filename)'=='lua' or '%(Filename)'=='luac'">true</ExcludedFromBuild>
    </ClCompile>
  </ItemGroup>
  <ItemDefinitionGroup>
    <ClCompile>
      <PreprocessorDefinitions>LUA_BUILD_AS_DLL;%(PreprocessorDefinitions)</PreprocessorDefinitions>
    </ClCompile>
  </ItemDefinitionGroup>
</Project>
```

## Expected Output Files

After successful build, you should have:
```
C:\lua\
├── bin\
│   ├── lua.exe
│   ├── luac.exe
│   └── lua54.dll
├── lib\
│   ├── lua54.lib
│   └── lua.lib (static, optional)
└── include\
    ├── lua.h
    ├── luaconf.h
    ├── lualib.h
    ├── lauxlib.h
    └── lua.hpp
```

## Troubleshooting

### Common Issues:
1. **Missing LUA_BUILD_AS_DLL**: Add this preprocessor definition
2. **Linker errors**: Ensure you're not including lua.c in the DLL project
3. **Runtime errors**: Make sure lua54.dll is in PATH or same directory as lua.exe

### Testing the Build:
```cmd
# Test interpreter
lua.exe -v

# Test DLL loading
lua.exe -e "print('Hello from DLL!')"

# Test in C++ project
# Link against lua54.lib and include lua headers
```

## Integration with Your Project

Once built, update your `setup-lua-environment.bat`:
```batch
set LUA_INSTALL_PATH=C:\lua\bin
set LUA_INCLUDE=C:\lua\include
set LUA_LIB=C:\lua\lib
