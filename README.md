# llama.cpp Android Tutorial

[中文](README_CN.md)

**Note: Current tutorial might be outdated, I will write a new version later**

llama.cpp link： https://github.com/ggerganov/llama.cpp

## Termux installation

Official Website: [termux](https://termux.dev/cn/index.html).

Change repo for faster speed (optional):

```bash
termux-change-repo
```

Check [here](https://wiki.termux.com/wiki/Package_Management) for more help.

## Init termux and ollama:
- reference blog: https://medium.com/@researchgraph/how-to-run-llama-3-2-on-android-phone-64be7783c89f

Note：
- The current ollama 0.4.0-rc has bugs: https://github.com/ollama/ollama/issues/7293, use `git checkout tags/v0.3.14` instead
- might need to `pkg install gzip`

## Install necessary code and packages

Download following packages in termux:

```bash
pkg install clang wget git cmake
```

Obtain llama.cpp source code:

```bash
git clone https://github.com/ggerganov/llama.cpp.git
```


## Importing language model

Type`termux-setup-storage` in termux terminal before importing model. Grant access for termux so that user could access files outside of termux. For details, please visit: https://wiki.termux.com/wiki/Termux-setup-storage

Use `adb push` command to import:

```
adb push \path\to\your\model\on\windows /storage/emulated/0/download
```

`~/storage/downloads` in termux home directory shares download files on Android system. Move it to `~/llama.cpp/models` 

```bash
mv ~/storage/downloads/model_name ~/llama.cpp/models
```

## Compile and build

**Strongly recommend to use cmake rather than make**

### Based on Android NDK（Non-OpenCL）

#### Install Pre-build NDK

Location: https://github.com/lzhiyong/termux-ndk/releases/tag/ndk-r23/

```bash
wget https://github.com/lzhiyong/termux-ndk/releases/download/ndk-r23/android-ndk-r23c-aarch64.zip
```

Unzip and set NDK PATH:

```bash
unzip YOUR_ANDROID_NDK_ZIP_FILE
export NDK=~/path/to/your/unzip/directory
```

#### Build

Build under `~/llama.cpp/build`:

```bash
mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-23 -DCMAKE_C_FLAGS=-march=armv8.4a+dotprod ..
make
```

Run:

```bash
cd bin/
./main YOUR_PARAMETERS
```

### Based on OpenCL + CLBlast（Recommend）

Download necessary packages: 

```bash
apt install ocl-icd opencl-headers opencl-clhpp clinfo libopenblas
```

Manually compile CLBlast and copy `clblast.h` into llama.cpp:

```bash
git clone https://github.com/CNugteren/CLBlast.git
cd CLBlast
cmake .
make
cp libclblast.so* $PREFIX/lib
cp ./include/clblast.h ../llama.cpp
```

Copy OpenBLAS files to llama.cpp:

```bash
cd ../llama.cpp
cp $PREFIX/include/openblas/cblas.h .
cp $PREFIX/include/openblas/openblas_config.h .
```

#### Build

```bash
cd ~/llama.cpp
mkdir build
cd build
cmake .. -DLLAMA_CLBLAST=ON
cmake --build . --config Release
```

Add `LD_LIBRARY_PATH` under `~/.bashrc`（Run program directly on physical GPU）：
ADD `egl` directory also if prompt error "cannot find libGLES_mali.so"
```bash
echo "export LD_LIBRARY_PATH=/vendor/lib64:/vendor/lib64/egl:$LD_LIBRARY_PATH" >> ~/.bashrc
```

Check GPU is available for OpenCL:

```bash
clinfo -l
```

If everything works fine, for Qualcomm Snapdragon SoC, it will display:

```bash
Platform #0: QUALCOMM Snapdragon(TM)
 `-- Device #0: QUALCOMM Adreno(TM)
```

Run:

```bash
cd ~/llama.cpp/build/bin/
./llama-cli -m your_model.gguf -p "You are a helpful assistant" -cnv
```

Note: The model ollama downloaded to local can be found via the following steps:

1. cd to `~/.ollama`，`cat ~/.ollama/models/manifests/registry.ollama.ai/library/llama3.2/3b`, find the filename which `mediaType` is `application/vnd.ollama.image.model`
2. get the file as `~/.ollama/models/blobs/sha256-xxxx`，which is the .gguf model


#### Results

- SoC: Qualcomm Snapdragon 8 Gen 2
- RAM: 16 GB
- Model: llama-2-7B-Chat-Q4_0.gguf（[Download](https://huggingface.co/Rabinovich/Llama-2-7B-Chat-GGUF)）
- Multiple long conversations
- Params:
  - Context size = 4096
  - Batch size = 16
  - Threads = 4

Result:

- Load time = 1129.17 ms
- 3.67 tokens per second
