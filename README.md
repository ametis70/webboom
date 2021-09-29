<img src="https://github.com/ianmethyst/webboom/raw/emscripten/prboom2/emscripten/logo.png" alt="WebBoom logo" style="width: 250px;" />

WebBoom is a [WebAssembly](https://webassembly.org/) port of Doom based on [PrBoom+](http://prboom-plus.sourceforge.net/).

[Play it now](https://webboom.surge.sh)

## Goals

- Remain compatible with [upstream](https://github.com/coelckers/prboom-plus) by changing as little as possible in the codebase/build scripts, which enables WebBoom to:
  + be compiled for desktop and WebAssembly targets
  + incorporate changes made to the original PrBoom+ after this fork was created
- Retain all singleplayer core features, including sound/music
- Keep the git history intact and provide documentation on how to compile the project

## Compiling

### Prerequisites

This project uses Emscripten as its compiler toolchain. To install it, follow [this instructions](https://emscripten.org/docs/getting_started/downloads.html) and activate the **2.0.30** version.

[CMake](https://cmake.org/download/) is needed to generate and run the build scripts, so go ahead and install that too.


### Bulding `prboom-plus.wad` and prboom-plus for PC 

To run prboom-plus (on desktop or web), you'll first need the `prboom-plus.wad` [PWAD](https://doomwiki.org/wiki/PWAD). It can be built off the `prboom2/data` directory on this repository using the CMake commands to compile a desktop build:

```sh
cd prboom2
cmake . # Generate the build scripts
cmake --build . # Build prboom-plus.wad, client executable and server executable
```

After this, the executable can found in `prboom2/` along `prboom-plus.wad`. To run it, use:

```sh
./prboom-plus
```

### Cleaning desktop build files

Copy the `prboom-plus.wad` above the repository root and then clean it using:

```sh
mkdir $HOME/prboom-data # Create a directory if you want
cp prboom-plus.wad $HOME/prboom-data # Copy the .wad there
git clean -xdf # WARNING: This will remove any untracked file added to the repository
```

### Compiling with Emscripten

After cleaning the repository, it's time to recreate the build files for WebAssembly. If there is an Emscripten version activated through `emsdk` and the env file was sourced, `emcc`, `emcmake` and `emmake` should be available in `$PATH`. If that's the case, proceed with:

```sh
cd prboom2
emcmake cmake . \
  -D BUILD_GL=0 \
  -D WITH_DUMB=0 \
  -D WITH_MAD=0 \
  -D WITH_FLUIDSYNTH=0 \
  -D WITH_VORBISFILE=0 \
  -D WITH_PORTMIDI=0 \
  -D WITH_ALSA=0 \
  -D WITH_PCRE=0 \
  -D WITH_NET=0 \
  -D PRBOOM_WAD="$HOME/prboom-data/prboom-plus.wad" # Or the path pointing to where the prboom-plus.wad is 
  -D PRELOAD_IWAD="$HOME/prboom-data/doom1.wad" # Or some other IWAD that is recognized by prboom 
  -D PRELOAD_CONFIG="$HOME/prboom-data/prboom-plus.cfg" # Optional. A path to a prboom-plus valid config
  -D CMAKE_BUILD_TYPE="Debug" # Or Release 

emmake cmake --build .
```

This will generate the following files: `index.html`, `index.js`, `index.wasm`, `index.wasm.map`, and `index.data`.

If you run `python -m "http.server"` or any web server in that directory, you should be able to play Doom on your browser!


#### Other notable CMake variables

``` sh
-D OUT_SUFFIX=".html" # emcc output format. Check "-o <target>" in the emmc reference
-D OUT_NAME="index" # Output name
-D EM_KEYBOARD_ELEMENT="\"doom\"" # DOM ID for the element that will capture mouse and keyboard events  
-D ADDITIONAL_FLAGS=""  #  Any flag accepted by emcc in the linking stage
```
For instance, this are the `ADDITIONAL_FLAGS` used in the [hosted](https://webboom.surge.sh) version:
```
 -D ADDITIONAL_FLAGS="-s MODULARIZE=1 -s EXPORT_NAME=prboom --shell-file='../emscripten/shell.html' -s INVOKE_RUN=0"
```

See also:
- [emcc reference](https://emscripten.org/docs/tools_reference/emcc.html)
- [settings.js](https://emscripten.org/docs/api_reference/advanced-apis.html#settings-js)

#### Differences between release and debug builds

Switching between `"Release"` and `"Debug"` in `CMAKE_BUILD_TYPE` affect some defaults flags:

| Type of release | Optimization | Output name | Output suffix | Other flags |
| --------------- | ------------ | ----------- | ------------- | ----------- |
| **Debug** | `-O0` (No optimization) | `index` | `.html` | `-gsource-maps --source-map-base /`. (Emit source maps. This will only work if the website is served from the `prboom2` directory) 
| **Release** | `-Oz` (Optimize the code with focus on size) | `prboom` | `.js` | None |


## Similar projects

The following projects served as inspiration for WebBoom:

| Project                                                               | Description | Play it |
|-----------------------------------------------------------------------|-------------|---------|
| [WebAssembly Doom](https://github.com/lazarv/wasm-doom)               | Based on Chocolate Doom/early Crispy Doom, and maybe other projects (git history erased). | [WadCommander](https://wadcmd.com/) |
| [Chocolate Doom WebAssembly](https://github.com/cloudflare/doom-wasm) | Based on Chocolate Doom with [multiplayer reimplementation](https://blog.cloudflare.com/doom-multiplayer-workers/) and no music. | [Silent Space Marine](https://silentspacemarine.com/) |
| [SDLDoom.wasm](https://github.com/Lorti/sdldoom.wasm)                 | Based on SDLDoom, one of the earliest source ports. | No link available |
| [1997 DooM to WebAssembly from Scratch](https://github.com/diekmann/wasm-fizzbuzz/tree/main/doom) | Based on LinuxDoom (original source code released). Has good documentation on the porting process and doesn't use [Emscripten](https://emscripten.org/) | [DooM](https://diekmann.github.io/wasm-fizzbuzz/doom/) |

## Todo

- [ ] Persist saves and settings
- [ ] Joystick support
- [ ] Improve UX
- [ ] Multiplayer?
