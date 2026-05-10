# Building aacgain on macOS (Universal Binary)

## Prerequisites

- Xcode Command Line Tools: `xcode-select --install`
- [Homebrew](https://brew.sh), then install build tools:

```sh
brew install cmake autoconf automake libtool
```

## Build

Clone with submodules:

```sh
git clone --recurse-submodules <repo-url>
cd aacgain
```

Build for each architecture into separate directories under `build/`:

```sh
cmake -B build/arm64  -S . -DCMAKE_OSX_ARCHITECTURES=arm64
cmake --build build/arm64

cmake -B build/x86_64 -S . -DCMAKE_OSX_ARCHITECTURES=x86_64
cmake --build build/x86_64
```

Combine into a universal binary:

```sh
lipo -create build/arm64/aacgain/aacgain \
             build/x86_64/aacgain/aacgain \
     -output build/aacgain
```

## Verify

```sh
lipo -info build/aacgain
# Architectures in the fat file: build/aacgain are: x86_64 arm64

./build/aacgain
# prints usage
```
