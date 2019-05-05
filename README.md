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
  * 8 servers: n1\~8; 4 ToR switches: t1\~4; 2 aggregation switches: a1\~2; 1 core switch: c1
  * The network is partitioned into two clusters
  * The links connecting to c1 are PPP, or the other networks are Ethernets, the networks’ capacities are shown on the topology graph.
  * All the end-end delays on the networks are 500ns.
  * IP address assignment is shown on the topology.
  * All the switches behaves like OSPF routers.
  
## Traffic patterns
 * Pattern 1: inter-cluster traffic
   * Each server communicates using TCP with another server that comes from different cluster
     * For example, 1-5, 6-2, 3-7, 8-4
 * Pattern 2: many-to-one traffic
   * Select one server as the sink, and all the other servers communicate to it
 * Simulate the two patterns separately, obtain the throughput that the network can achieve, and find out the network bottleneck, how to improve the network.
 
 ```diff
+ 鸟宿池边树，僧敲月下门
- 鸟宿池边树，僧推月下门
```
