# tensorflow-build
源码编译安装tensorflow GPU版本过程

## 硬件信息及版本
  * hw ：
    * Intel® Core™ i7-6700HQ CPU @ 2.60GHz
    * NVIDIA GTX1060 6G GDDR5
    * 8GB DDR4内存
  * OS ： Ubuntu 18.04.2 LTS
  * tensorflow branch : origin/r1.13
  * cuda : 10.0
  * cudnn: 7.4.2
  * nccl: 2.4.2
  * python ：2.7  
  * bazel: 0.19.2
 ```diff
+ 源码编译安装TensorFlow版本需要对应，不然可能会出现不可预期的错误！
``` 
  
## 查看GPU信息
### 命令
  * lspci | grep -i nvidia
  * sudo lshw -numeric -C display
### 输出
  * 06:00.0 VGA compatible controller: NVIDIA Corporation GK110B [GeForce GTX TITAN Z] (rev a1)
  * 06:00.1 Audio device: NVIDIA Corporation GK110 HDMI Audio (rev a1)
  * 07:00.0 3D controller: NVIDIA Corporation GK110B [GeForce GTX TITAN Z] (rev a1)

## 禁用nouveau驱动
```diff
  + 使用核显和独显，因为对NVIDIA显卡驱动支持不太好，很多在安装TensorFlow GPU版本时经常会遇到黑屏，进不到Linux系统中等问题； 因此在安装显卡驱动和CUDA前先禁用nouveau驱动
```
### 操作
  * sudo vim /etc/modprobe.d/blacklist-nouveau.conf
  * \# 输入以下内容
  * blacklist nouveau
  * options nouveau modset=0
### 执行
  * sudo update-initramfs -u
### 重启电脑查看是否成功
  * sudo lspci | grep nouveau  
  **如果没有内容，则禁用成功。**
  
## 安装cuda
+ 到[官网](https://developer.nvidia.com/cuda-downloads "cuda")下载对应版本的cuda安装包，我下载的是[cuda_10.0.130_410.48_linux.run](https://developer.nvidia.com/compute/cuda/10.0/Prod/local_installers/cuda_10.0.130_410.48_linux)版
### 执行以下命令
  * sudo /etc/init.d/lightdm stop  　　 \# 关闭X-server ctrl+alt+F1进入命令行
  * sudo init 3
  * chmod 777 cuda_10.0.130_410.48_linux.run
  * sudo ./cuda_10.0.130_410.48_linux.run
  * sudo vim ~/.bashrc
  * \# 添加以下三行内容
  * export CUDA_HOME=/usr/local/cuda-10.0
  * export PATH=/usr/local/cuda-10.0/bin:$PATH
  * export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64:$LD_LIBRARY_PATH
  * source ~/.bashrc  　　       \#使配置的环境变量生效 
  * sudo vim /etc/ld.so.conf.d/cuda-10-0.conf
  \# 添加以下二行内容
  * /usr/local/cuda-10.0/lib64
  * /usr/local/cuda-10.0/extras/CUPTI/lib64
  * sudo ldconfig
  * nvidia-smi      　　　    　    \#出现GPU相关信息表示驱动安装成功
  * nvcc --version      　　    \#查看cuda版本信息
  * sudo service lightdm start 
### 验证
  * cd /usr/local/cuda-10.0/samples/1_Utilities/deviceQuery  
  * sudo make  
  * sudo ./deviceQuery  
**保证编译没有error，执行deviceQuery输出信息的最后类似以下：**
```
deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.0, CUDA Runtime Version = 10.0, NumDevs = 2, Device0 = GeForce GTX TITAN Z, Device1 = GeForce GTX TITAN Z
Result = PASS
```

## 安装cuDNN
+ 到[官网](https://developer.nvidia.com/rdp/cudnn-download "cudnn")下载对应版本的cuDNN，我下载的是[cudnn-10.0-linux-x64-v7.4.2.24](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/v7.4.2/prod/10.0_20181213/cudnn-10.0-linux-x64-v7.4.2.24.tgz)版
### 执行以下命令
  * tar -xzvf cudnn-10.0-linux-x64-v7.4.2.24.tgz
  * sudo cp cuda/include/cudnn.h /usr/local/cuda/include
  * sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
  * sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
### 查看是否安装成功
  * nvcc -V
  
## nccl : 2.4.2
 * [下载链接](https://developer.nvidia.com/nccl/nccl-download#a-collapse242-100)
 * sudo dpkg -i nccl-repo-ubuntu1804-2.4.2-ga-cuda10.0_1-1_amd64.deb
 * sudo apt update
 * sudo apt install libnccl2 libnccl-dev
 * 如果希望保留较旧版本的CUDA，请指定特定版本，例如：
  * sudo apt-get install libnccl2=2.4.2-1+cuda10.0 libnccl-dev=2.4.2-1+cuda10.0
  
## bazel: 0.19.2
 * [下载链接](https://github.com/bazelbuild/bazel/releases/download/0.19.2/bazel-0.19.2-installer-linux-x86_64.sh)
 * chmod +xxx bazel-0.19.2-installer-linux-x86_64.sh
 * ./bazel-0.19.2-installer-linux-x86_64.sh --user
 
## python2.7.16: miniconda2
 * 首先安装miniconda2，可以自己搜索安装方法，在该创建虚拟下使用bazel编译安装TensorFlow
 * conda create -n python2 python=2.7 -y
 * source activate python2
 
## 编译命令：仅供参考
```
bazel build --config=opt --config=cuda --local_resources 4096,4,1.0 --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" //tensorflow/tools/pip_package:build_pip_package
```
使用上述命令，可能CPU或内存资源不够，导致卡死，出现错误：

```
bazel build --config=opt --config=cuda --local_resources 4096,4.0,1.0 -j 1 --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" //tensorflow/tools/pip_package:build_pip_package
```

# 祝君好运！！！
