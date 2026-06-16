## Prerequisites
- C++ compiler with C++17 support (GCC 7+, Clang 5+, or MSVC 2017+)
- CMake 3.10 or newer
- HDF5 development libraries
- VMD installation

### Install dependencies (Ubuntu/Debian):
```bash
sudo apt-get update
sudo apt-get install build-essential cmake libhdf5-dev pkg-config
```

### Install dependencies (CentOS/RHEL):
```bash
sudo yum install gcc-c++ cmake hdf5-devel pkgconfig
# or for newer versions:
sudo dnf install gcc-c++ cmake hdf5-devel pkgconfig
```

### Install dependencies (macOS):
```bash
brew install cmake hdf5 pkg-config
```

## Build
```bash
mkdir build
cd build
cmake ..
make && make install    
```

## Alternative conda build (Only if first not work)
```bash
conda env create -f env.yml
conda activate vmdplugin
mkdir build
cd build
cmake ..
make && make install
```

## Installing into a specific VMD version (when several VMDs are installed)

By default CMake auto-detects VMD by finding the `vmd` launcher on your `PATH` and
reading its `defaultvmddir`. So a plain `cmake ..` targets whichever `vmd` comes
first on your `PATH`. If you have multiple VMD versions, build once per version and
point CMake at each one explicitly with the `VMD_ROOT_DIR` cache variable.

### 1. Find your VMD install directories
Each VMD launcher script defines `defaultvmddir` (the install root):
```bash
which -a vmd vmd2 vmd201a1        # launchers on your PATH
grep -m1 defaultvmddir $(which vmd)   # the dir a given launcher uses
ls -d /usr/local/lib/vmd*         # or list installs directly
```
The plugin you will replace lives at `<VMD_ROOT_DIR>/plugins/<PLATFORM>/molfile/lmplugin.so`
(`<PLATFORM>` is `LINUXAMD64` on Linux, `MACOSX86_64` on macOS).

### 2. Build + install into the chosen version
Use a **separate build directory per VMD version** so builds don't clobber each other:
```bash
# example: target a VMD 2.0.1-alpha installed at /usr/local/lib/vmd201a1
mkdir build_vmd201a1 && cd build_vmd201a1
cmake -DVMD_ROOT_DIR=/usr/local/lib/vmd201a1 ..
make
make install     # installs lmplugin.so into that VMD's molfile dir
```
`cmake` prints the resolved paths — confirm them before `make install`:
```
-- VMD_ROOT_DIR: /usr/local/lib/vmd201a1
-- VMD_PLUGIN_DIR: /usr/local/lib/vmd201a1/plugins/LINUXAMD64/molfile
```

### 3. Back up the existing plugin first (recommended)
`make install` overwrites the existing `lmplugin.so`. Keep a copy so you can revert:
```bash
PLUGDIR=/usr/local/lib/vmd201a1/plugins/LINUXAMD64/molfile
cp -n "$PLUGDIR/lmplugin.so" "$PLUGDIR/lmplugin.so.bak"   # -n: never overwrite an existing backup
```
If that directory is not writable by your user, prefix `make install` (and the
backup copy) with `sudo`.

### 4. Verify the correct plugin is active
```bash
echo 'mol new your.lm; quit' | /usr/local/bin/vmd201a1 -dispdev text
```
A good load prints `Using plugin LM for structure file ...` with no `HDF5` /
`Wrong HDF5 data type` errors. To revert, copy `lmplugin.so.bak` back over
`lmplugin.so`.

> Load `.lm` files with `mol new file.lm` (auto-detect) or `mol new file.lm type LM`
> — the plugin name is **`LM`** (uppercase); `type lm` (lowercase) is rejected with
> `Cannot read file of type lm`.

