## 问题跟踪系统参考示例

### 驱动类

| DCU    型号 | 问题描述                                                     | 解决方法                                                     |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Z100L       | 无法更新麒麟V10系统上Z100L的vbios                            | 手动使用  hygonvbflash_v1.6.0_build231109 工具，刷新vbios为 4.52.0003002.314977 版本 |
| Z100L       | ollama跑大模型会导致系统crash                                | 驱动版本导致，驱动降级到rock-5.2.0-5.16.22-1.x86_64可以使用，也可测试最新驱动。 |
| Z100L       | 设备中同时存在K100与Z100L情况下安装最新驱动显示安装成功，重启后lsmod  \| grep hydcu无信息 | 以centos  7.6为例，将ko文件复制到/lib/modules对应路径下：     cp -r /opt/hyhal/dkms/*  /lib/modules/3.10.0-957.el7.x86_64/updates/dkms/     然后重新加载hydcu：modprobe hydcu |
| Z100L       | gemm性能只有标称算力一半                                     | 升级DCU驱动为rock-4.5.2-5.11.40-V01.8.1.run                  |
| K100_AI     | X640  G50+8卡K100_AI机型安装驱动提示"Local GCC version 80301 does not match kernel  compiler GCC version 80201"，目前驱动安装失败。 | 研发修改编译失败的代码重新发布rock-kernel-refactory-rock-5.7.1-6.2.17-V1.1.1.aio.run驱动，重新按照驱动即可。 |
| /           | k8s集群环境，rocm-smi命令报错，在集群环境下，重装驱动无效，把集群reset后重装驱动可以正常使用。机器断电重启后再次出现error:Open  mkfd failed | 通过k8s-dcu-plugin.yaml，dcu-exporter.yaml，关闭相应插件，卸载驱动，在重装驱动，节点重启，可以解决。 |
| /           | OpenEuler22.03系统安装完驱动，lsmod  \| grep hydcu和hy-smi可查看到显卡信息，但是rocminfo会报Open kfd failed at Load | 升级驱动rock5.7  重启解决                                    |
| /           | Ubuntu20.04，初始内核版本  5.4.0-42，安装 DCU 驱动重启后，驱动无法正常加载； | 设置初始内核版本为默认启动项，重启后重新安装驱动并锁核       |
| Z100L       | 跑llama_tencentpretrain_pytorch训练报错                      | 客户机是z100sm  dtk23.10跑不了，改成安装dtk23.04，以及对应的torch torchvision deepspeed，解决了这个问题 |

### DTK类

| DCU型号 | 问题描述                                                     | 解决方法                                                     |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| /       | DTK24.04  libcudart.so.11.0 报错，找不到cuda                 | dtk-24.04  CUDA兼容模块“cudamocker”更名为“GPUfusion”，暂时以第三方库的形式独立DTK提供，链接如下。，下载后，可按照说明进行安装使用。https://forum.hpccube.com/thread/483     dtk-24.04.1恢复正常     1. 解压gpufusion     unzip -q gpufusion.zip     2. 加载DTK目录下的env.sh，用于设置DTK环境变量：     source dtk/env.sh     3. 加载cuda目录下的env.sh，用于设 |
| Z100sm  | 跑llama_tencentpretrain_pytorch训练报错                      | z100sm 不支持dtk23.10，改成安装dtk23.04，以及对应的torch  torchvision deepspeed，解决了这个问题 |
| Z100L   | DCU  Z100L 使用PaddleSeg、PaddleDet错误 Deeplabv3p、PaddleDet 的 yolov3、yolov5、yolov8出现错误 | 采用 dtk23.04 下的paddle-4.2 可以暂时绕过该问题，            |

### 操作系统类

| DCU    型号 | 问题描述                                                     | 解决方法                                                     |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| /           | DCU升级后重启系统，卡在了os的启动界面，操作系统为centos7.6   | 在采用如下操作后能够成功进入系统：强制重启——>进入BIOS——>禁用DCU——>启动系统（无法进入，卡在加载读条界面）——>强制重启——>进入BIOS——>启用DCU——>启动系统，成功进入操作系统。但仍未定位根本原因 |
| Z100L       | Ubuntu  22.04.4，kernel 6.5.0-26，A620-G30 服务器，2 Z100L，安装最新 DCU  驱动rock-5.2.0-5.16.29-V01.13，出现报错 gcc can not gen ko! linux-header not ready! | 降低  Ubuntu 内核（kernel建议 5.15.0），重新安装驱动。       |
| Z100L       | 安装centos7.6系统、中科方德4.0重启后无问题出现，安装DCU  驱动重启后重复打印cpu lock up stuck for XXs | 开机界面禁用显卡板载  。加modprobe.blacklist=ast     参考链接：https://blog.csdn.net/lindexin/article/details/105138061     进系统后：https://blog.csdn.net/zgl07/article/details/46493421     再重启机器恢复 |
| K100_AI     | X7840H0机器出场默认拓扑为common方式，改配安装8卡K100_AI后无法开机，开机代码卡在0x92处无法正常开机，BIOS版本CDSH051033。 | 通过抓取开机自检日志得出结论是PCIe地址分配问题，需要更新BIOS版本来解决，尝试将测试版本BIOS固件提供，更新到服务器上解决无法开机问题 |
| Z100L       | X7840H0（8*Z100L  32G）在UOS 1050u1a环境下使用Z100L和vllm推理qwen1.5，会报下面bug HIP kernel errors might be  asynchronously stacktrace below might be incorrect.For debugging consider  passing HIP LAUNCH BLOCKING 1.Compile with TORCH USE HIP DSA to enable  device-side assertions | 更换麒麟V10  SP2操作系统及对应驱动                           |
| Z100L       | Z100L在UOS 1050u1a环境上使用vllm推理qwen1.5模型报错          | 更换麒麟V10  SP2操作系统及对应驱动                           |
| Z100L       | 麒麟操作系统上安装IB网卡和DCU驱动，IB网卡restart  失败,卸载DCU驱动，网卡启动正常。 | 问题需要定位到是什么占用了ib_core。通过使用/etc/init.d/openibd  start 可以启动网卡，restart会报错。 |

 

### 系统环境类

| DCU型号 | 问题描述                                                     | 解决方法                                                     |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| /       | 在dtk-24.04中配置conda环境中的jax和jaxlib，运行程序报错：ImportError:  /lib64/libc.so.6: version GLIBC_2.27 not found等一系列问题。 | ImportError:  /lib64/libc.so.6: version GLIBC_2.27 not found     解决办法     wget http://ftp.gnu.org/gnu/glibc/glibc-2.29.tar.gz     tar -zxvf glibc-2.29.tar.gz     cd glibc-2.29     mkdir build     cd build/     ../configure --prefix=/usr/local --disable-sanity-checks     make -j32     make install |
| /       | 在使用gitlab上的fastllm工程编译时报错，CMAKE_CUDA_COMPILER  not set,after EnableLanguage。Configuring incomplete, errors occured! | 升级cmake版本，cmake  3.6.0版本低，不兼容cuda，升级至3.21.0后即可正常编译。 |
| /       | X640G40  MMIO设置                                            | 进入BIOS界面Socket  Configuration->Commom RefCode Configuration->MMIO High  Base将Base值由24T修改成1T,然后保存退出即可 |
| /       | KylinV10SP2在DTK24.04中improt  torch时,报错ImportError: libibverbs | 1.yum install -y libibverbs     improt torch,出现新的错误ImportError: /usr/lib64/libgobject-2.0.so.0: symbol  ffi_type_uint32 version LIBFFI_BASE_7.0 not defined in file libffi.so.7 with  link time reference     2.find / -name libffi.so.7，找出libffi.so.7文件路径     3.export LD_PRELOAD=/usr/lib64/libffi.so.7 |
| /       | 使用两卡Z100L环境部署ollama，测试llama2，启动server后，运行ollama  run llama2报错,error:Could not initialize Tensile host: No devices found | 设置环境变量HIP_VISIBLE_DEVICES（不论几卡环境）              |



### 大模型推理类

| DCU型号 | 问题描述                                                     | 解决方法                                                     |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Z100L   | dtk23.10或24.04下，vllm+qwen2  72b+int4量化环境下乱码        | http://10.0.15.207/summary/adapted-algorithm/qwen-adapt/tree/z100l_vllm0.3.3_qwen1.5_qwen2 |
| Z100L   | Z100L不支持Vllm适配Qwen1.5、Qwen2                            | http://10.0.15.207/summary/adapted-algorithm/qwen-adapt/tree/z100l_vllm0.3.3_qwen1.5_qwen2 |
| Z100L   | 1.  vllm-0.3.3在Z100L推理qwen-1.5-4B出现显存爆炸，对话结果乱回答2.Z100L的vllm无法提供对模型qwen-1.5-72B的支持； | https://forum.hpccube.com/thread/515                         |
| Z100L   | 使用24.04版本dtk,Z100L，在fastllm框架推理chatglm-6b,修改benchmark.cpp SetDeviceMap设置两卡，编译后推理，设置batch 64或者128, 两个卡均占用，显存占到70%，但是不出结果，几分钟后报oom。使用dtk23.10.1推理正常。 | 切换vllm使用新版dtk问题已解决。                              |
| Z100L   | 使用DTK2404的vllm环境测试Qwen1.5-72B-Chat推理回答乱码        | vllm+dtk24.04仅支持K100_AI，暂不支持其他型号。               |
| Z100L   | Z100L 使用 DBnet (24，3，768，1280) 推理报错，并且无法  export onnx 模型。     DBnet开源项目: https://github.com/MhLiao/DB | 暂未解决                                                     |

