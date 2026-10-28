# How to build Mighty Mike

## The easy way: build.py (automated build script)

`build.py` can produce a game executable from a fresh clone of the repo in a single command. It will work on macOS, Windows and Linux, provided that your system has Python 3, CMake, and an adequate C++ compiler.

```
git clone --recurse-submodules https://github.com/jorio/MightyMike
cd MightyMike
python3 build.py
```

If you want to build the game **manually** instead, the rest of this document describes how to do just that on each of the big 3 desktop operating systems.

## How to build the game manually on macOS

1. Install the prerequisites:
    - Xcode (preferably the latest version)
    - [CMake](https://formulae.brew.sh/formula/cmake) 3.21+ (installing via Homebrew is recommended)
1. Clone the repo **recursively**:
    ```
    git clone --recurse-submodules https://github.com/jorio/MightyMike
    cd MightyMike
    ```
1. Download [SDL3-3.2.2.dmg](https://libsdl.org/release/SDL3-3.2.2.dmg), open it, then browse to SDL3.xcframework/macos-arm64_x86_64. In that folder, copy **SDL3.framework** to the game's **extern** folder.
1. Prep the Xcode project:
    ```
    cmake -G Xcode -S . -B build
    ```
1. Now you can open `build/MightyMike.xcodeproj` in Xcode, or you can just go ahead and build the game:
    ```
    cmake --build build --config RelWithDebInfo
    ```
1. The game gets built in `build/RelWithDebInfo/MightyMike.app`. Enjoy!

## How to build the game manually on Windows

1. Install the prerequisites:
    - Visual Studio 2022 with the C++ toolchain
    - [CMake](https://cmake.org/download/) 3.21+
1. Clone the repo **recursively**:
    ```
    git clone --recurse-submodules https://github.com/jorio/MightyMike
    cd MightyMike
    ```
1. Download [SDL3-devel-3.2.2-VC.zip](https://libsdl.org/release/SDL3-devel-3.2.2-VC.zip), extract it, and copy **SDL3-3.2.2** to the **extern** folder. Rename **SDL3-3.2.2** to just **SDL3**.
1. Prep the Visual Studio solution:
    ```
    cmake -G "Visual Studio 17 2022" -A x64 -S . -B build
    ```
1. Now you can open `build/MightyMike.sln` in Visual Studio, or you can just go ahead and build the game:
    ```
    cmake --build build --config Release
    ```
1. The game gets built in `build/Release/MightyMike.exe`. Enjoy!

## How to build the game manually on Linux et al.

1. Install the prerequisites from your package manager:
    - Any C++20 compiler
    - CMake 3.21+
    - SDL3 development library (e.g. "libsdl3-dev" on Ubuntu, "sdl3" on Arch, "SDL3-devel" on Fedora)
    - OpenGL development libraries (e.g. "libgl1-mesa-dev" on Ubuntu)
1. Clone the repo **recursively**:
    ```
    git clone --recurse-submodules https://github.com/jorio/MightyMike
    cd MightyMike
    ```
1. Build the game:
    ```
    cmake -S . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
    cmake --build build
    ```
    If you'd like to enable runtime sanitizers, append `-DSANITIZE=1` to the **first** `cmake` call above.
1. The game gets built in `build/MightyMike`. Enjoy!

## Local browser build (WebAssembly)

Tested with **Emscripten 4.0.23**, SDL **3.2.28**, and Chromium **150** on Linux.

### Set up the toolchain

Run these commands from the project root. Skip the SDK installation if you already
have an activated Emscripten installation.

```sh
git submodule sync
git submodule update --init --recursive
git clone --depth 1 --branch 4.0.23 https://github.com/emscripten-core/emsdk.git cache/emsdk
cache/emsdk/emsdk install 4.0.23
cache/emsdk/emsdk activate 4.0.23
. cache/emsdk/emsdk_env.sh
```

Activate the SDK again in each new terminal with `. cache/emsdk/emsdk_env.sh`.
CMake, a build tool such as Make, Git, and Python 3 are also required.

### Build and play

```sh
emcmake cmake -S . -B build-web -DCMAKE_BUILD_TYPE=Release
cmake --build build-web --parallel 8
python3 -m http.server 8002 --bind 127.0.0.1 --directory build-web
```

Open **http://127.0.0.1:8002/MightyMike.html**; `file://` does not work.
The game starts after loading. Click **Enable sound & fullscreen** if sound is blocked.

CMake fetches SDL 3.2.28; the pinned [Pomme fork](https://github.com/Rage997/Pomme)
reuses that SDL target. To use a local SDL checkout, add
`-DFETCHCONTENT_SOURCE_DIR_SDL3=/path/to/SDL`.

Artwork is in `packaging/web-assets/`; the launcher is `packaging/web.html`.

### Controls and storage

- **Arrows / WASD:** movement; **Space:** fire.
- **Left Shift / E:** next weapon; **Right Shift / Q:** previous weapon.
- **R / Tab:** bunny radar; **Esc:** pause; **Enter:** menu confirmation.
- Use the page's **Fullscreen** button instead of desktop display settings.
- Saves, preferences, and scores stay in this browser. Keep the same hostname and
  port, and let saves finish before closing the tab. Storage errors are shown on the page.
- Quitting returns to the launcher. **Play again** reloads the game.

For static hosting, keep `MightyMike.html`, `MightyMike.js`, `MightyMike.wasm`,
`MightyMike.data`, `MightyMike.png`, `web-license.txt`, and `web-assets/` together.
No cross-origin isolation headers are needed.

Tested in Chromium only; Firefox, Safari, and physical gamepads have not been
exercised. Touch controls are not provided.

