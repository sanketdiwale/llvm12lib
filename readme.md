A collection of llvm12 binaries for convenience (until the official release is made).
An automated build script is included in .github/workflows

# To build 
On Mac and Linux
```bash
mkdir build && cd build
cmake ..
cmake --build .
cmake --install .
```

On Windows (MSVC compiler)
```bash
mkdir build && cd build
cmake ..
cmake --build . --config Release 
cmake --install .
```