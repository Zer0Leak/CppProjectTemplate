# CppProjectTemplate 

Tempate for easly start CPP Project

# Edit Template to your project

* Edit the first and second line for this file to match your Project
* Rename SubProject folder
* Default LICENSE is LGPL (change for you needs)
* Replace target name

```bash
export SUBPROJECT_NAME="TheRealSubProjectName"
export TARGET_NAME="the_real_subproject_target_name"

# 1. Rename the SubProject Folder
mv SubProject "$SUBPROJECT_NAME"

# 2. Rename Target Name
sed -i "s/cpp_template_subproject_target_name/$TARGET_NAME/g" "$SUBPROJECT_NAME/CMakeLists.txt" 
sed -i "s/cpp_template_subproject_target_name/$TARGET_NAME/g" "$SUBPROJECT_NAME/CMakePresets.json" 
sed -i "s/SubProject/$SUBPROJECT_NAME/g" "$SUBPROJECT_NAME/CMakeLists.txt"
sed -i "s/SubProject/$SUBPROJECT_NAME/g" "$SUBPROJECT_NAME/CMakePresets.json"
sed -i "s/SubProject/$SUBPROJECT_NAME/g" "$SUBPROJECT_NAME/.vscode/settings.json"

# 3. If not using RAM FS :-(
sed -i "s|# export BUILD_FOLDER=/mnt/ram/${SUBPROJECT_NAME}  # to build in RAM FS|# export BUILD_FOLDER=build|" "$SUBPROJECT_NAME/CMakeLists.txt"
sed -i "s#/mnt/ram/${SUBPROJECT_NAME}#\${sourceDir}/build#g" "$SUBPROJECT_NAME/CMakePresets.json"
sed -i "s#/mnt/ram/${SUBPROJECT_NAME}#\${workspaceFolder}/build#g" "$SUBPROJECT_NAME/.vscode/settings.json"
```
* Remember to copy .clang-format

# Setup for Ubuntu 24.04

# update

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

# Essentials and C++ libs

```bash
sudo apt-get install -y unzip build-essential cmake gdb git wget cmake libfmt-dev libblas-dev libopenblas-dev libfftw3-dev
sudo apt-get install -y python3 python3-pip python-is-python3 python3-venv
sudo apt-get install -y liblapacke-dev liblapack-dev libboost-all-dev libopencv-dev libhdf5-dev
sudo apt-get install -y libjson-c-dev libx11-dev openjdk-21-jdk ninja-build gnuplot vim libcpuinfo-dev libspdlog-dev
sudo apt-get install -y clang clang-tidy clang-format clangd mold lldb cmake ccache pkg-config
sudo apt-get install -y libstdc++-14-dev
sudo apt-get install -y libmlpack-dev mlpack-bin
sudo apt-get install -y libarmadillo-dev libensmallen-dev libcereal-dev libstb-dev
sudo apt-get install -y libfreeimage-dev libfontconfig1-dev libglu1-mesa-dev
```

- link with -lopenblas ou -lblas -llapack instead of -latlas

# In VS Code install Extensions

- C/C++ (ms-vscode.cpptools)
- CMake Tools (ms-vscode.cmake-tools)
- clangd (llvm-vs-code-extensions.vscode-clangd) – optional, great IntelliSense
- CodeLLDB (Vadim Chugunov)

# Linear Algebra 

```bash
sudo apt install -y libeigen3-dev xtensor-dev libxtensor-blas-dev libomp-dev

sudo apt remove "libarrayfire*"

sudo apt install -y libglfw3-dev libfreetype6-dev libgl1-mesa-dev
sudo apt install -y libdlib-dev libdlib-data
sudo apt install -y opencl-headers
sudo apt install -y intel-opencl-icd
sudo apt install -y clinfo
clinfo
afinfo
```

```bash
# install CUDA and build ArrayFire from source
lspci | grep -i nvidia
sudo ubuntu-drivers install
sudo reboot
nvidia-smi

# Get the repo keyring package for ubuntu2404
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb

sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update ; sudo apt-get upgrade

sudo apt-get install -y cuda-toolkit nvidia-cuda-samples

echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.zshrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.zshrc
```

```bash
# Install Arrayfire

# Core toolchain
sudo apt install -y build-essential cmake ninja-build git pkg-config

# Common deps (OpenCL headers/ICD loader; fonts; etc.)
sudo apt install -y ocl-icd-opencl-dev opencl-headers \
  libfftw3-dev liblapack-dev libblas-dev \
  libglfw3-dev libfontconfig1-dev \
  libpng-dev libjpeg-dev libtiff-dev

# (Optional but common) for nicer debugging / tooling
sudo apt install -y clang lld ccache

wget https://arrayfire.gateway.scarf.sh/linux/3.10.0/ArrayFire.sh
chmod a+x ArrayFire.sh
sudo mkdir -p /opt/arrayfire
sudo ./ArrayFire.sh --prefix=/opt/arrayfire --exclude-subdir
echo /opt/arrayfire/lib64 | sudo tee /etc/ld.so.conf.d/arrayfire.conf
sudo ldconfig


# can try with ubuntu libs for arrayfire setting

"env": {
    // 1. Forces the NVIDIA GPU to handle rendering for the hybrid (Optimus) system
    "__NV_PRIME_RENDER_OFFLOAD": "1", 
    
    // 2. Directs the GLX loader to specifically use the NVIDIA driver library
    "__GLX_VENDOR_LIBRARY_NAME": "nvidia", 
    
    // 3. Sets the preferred ArrayFire backend (CUDA) if the code doesn't specify one
    "AF_DEFAULT_BACKEND": "cuda",
    
    // 4. Forces ArrayFire to print detailed error messages and stack traces to the console
    "AF_PRINT_ERRORS": "1",
    
    // 5. Redundant but useful: ensures the CUDA backend is initialized
    "AF_BACKEND": "cuda"
},
```


# Create/Activate Python Env

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install torch==2.7.1 --index-url https://download.pytorch.org/whl/cpu
```

# Add this to your User Settings (JSON)

```json
{
    "workbench.colorTheme": "Default Dark+",
    "terminal.integrated.fontFamily": "JetBrainsMono Nerd Font",
    "terminal.integrated.fontSize": 16,
    "terminal.integrated.fontWeight": "bold",
    "github.copilot.nextEditSuggestions.enabled": true,
    // "editor.formatOnSave": true,
    "C_Cpp.formatting": "clangFormat",
    "C_Cpp.clang_format_path": "/usr/bin/clang-format"
}
```





