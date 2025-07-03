Please note that there are some features depend on the gcc 11 feature. So you should make sure at least you have the gcc 11

To check the version: 
```bash
gcc --version
```
If you see any version lower than **11**, you should update your gcc version:
```bash
sudo apt update
sudo apt install gcc-11 g++-11
sudo apt upgrade libstdc++6
```
Otherwise, you can just skip this step.

## install
```bash
conda env create -f env.yml
conda activate vmdplugin

```

## build
```bash
mkdir build
cd build
cmake -DCMAKE_PREFIX_PATH="$CONDA_PREFIX" -DHDF5_ROOT="$CONDA_PREFIX"
make && make install    
```

