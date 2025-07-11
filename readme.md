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

