# DPUCAHX8H -- the DPU for Alveo Accelerator Card with HBM

Xilinx DPU IP family for convolution nerual network (CNN) inference application supports Alveo accelerator cards with HBM now, including Alveo U50, U50LV and U280 cards. According to the latest Xilinx DPU naming rule, the DPU for Alveo-HBM card is named ***DPUCAHX8H***. The on-premise DPUCAHX8H overlays are released along with Vitis AI. A few variants of DPUCAHX8H are provided, which will be explained in later section. Please refer to the relevant parts for usages of different DPUCAHX8H overlays with [VART](../VART/README.md) and [Vitis-AI-Library](../Vitis-AI-Library/README.md) (you could search the keyword "for Cloud").

Following page will guide you through the Alveo card and on-premise overlays setup flow for Vitis AI.

## Alveo Card Setup

### Get and Install the Alveo Card Target Platform

#### XRT
Before you go through the next steps, please ensure the latest Xilinx runtime (XRT) is installed on your host, you can get XRT from these links:

CentOS/Redhat 7.4-7.7: [xrt_202010.2.6.655_7.4.1708-x86_64-xrt.rpm](https://www.xilinx.com/bin/public/openDownload?filename=xrt_202010.2.6.655_7.4.1708-x86_64-xrt.rpm)

Ubuntu 16.04: [xrt_202010.2.6.655_16.04-amd64-xrt.deb](https://www.xilinx.com/bin/public/openDownload?filename=xrt_202010.2.6.655_16.04-amd64-xrt.deb)

Ubuntu 18.04: [xrt_202010.2.6.655_18.04-amd64-xrt.deb](https://www.xilinx.com/bin/public/openDownload?filename=xrt_202010.2.6.655_18.04-amd64-xrt.deb)

#### Alveo U280 Card
For U280 card, DPUCAHX8H use the standard target platform released in the Xilinx website, please follow the instruction in the [U280 page](https://www.xilinx.com/products/boards-and-kits/alveo/u280) to get and install the required files. 

After the installation, please use the command below to update the dard flash, followed by a cold reboot.

~~~
sudo /opt/xilinx/xrt/bin/xbmgmt flash --update --shell xilinx_u280_xdma_201920_3 
~~~

#### Alveo U50 and U50LV Card
For U50/U50LV card, DPUCAHX8H use the gen3x4 version target platform instead of the standard gen3x16 platform. Please download and install the required gen3x4 target platform files from [dpuv3e_u50_platform_vai1.2.tgz](https://www.xilinx.com/bin/public/openDownload?filename=dpuv3e_u50_platform_vai1.2.tgz). 
After the installation, please use the commands below to update the card flash, followed by a cold reboot.

For Alveo U50:
~~~
sudo /opt/xilinx/xrt/bin/xbmgmt flash --update --shell xilinx_u50_gen3x4_xdma_base_2
~~~

For Alveo U50LV:
~~~
sudo /opt/xilinx/xrt/bin/xbmgmt flash --update --shell xilinx_u50lv_gen3x4_xdma_base_2
~~~

You could use the BASH script u50_shell_setup_preprod.sh to finish the download and installation of the U50 card platform. For example:

~~~
source ./u50_shell_setup_preprod.sh
~~~

You could use the BASH script u50lv_shell_setup_preprod.sh to finish the download and installation of the U50LV card platform. For example:

~~~
source ./u50lv_shell_setup_preprod.sh
~~~

---

## DPUCAHX8H Overlays Setup

Four kinds of DPUCAHX8H overlays are provided for different Alveo HBM card:
* U50-6E300M: two kernels, six engines, maximal core clock 300MHz
* U50LV-9E275M: two kernels, nine engines, maximal core clock 275MHz
* U50LV-10E275M: two kernels, ten engines, maximal core clock 275MHz
* U280-14E300M: three kernels, fourteen engines, maximal core clock 300MHz

### Get and Decompress Overlays Tarball
In the host or docker, get to the shared Vitis AI git repository directory and use following commands to download and decompress the overlays tarball. (the download link is not available yet for BASH stage)
~~~
cd ./Vitis-AI/alveo-hbm
wget https://www.xilinx.com/bin/public/openDownload?filename=alveo_xclbin-1.2.0.tar.gz -O alveo_xclbin-1.2.0.tar.gz
tar xfz alveo_xclbin-1.2.0.tar.gz
~~~

### Settle Down the Overlays
Start the docker, get into the shared Vitis AI git repository directory and use following command to settle down the overlay files for different Alveo card. Please note everytime you start a new docker container, you should do this step.

For Alveo U50, use U50-6E300M overlay:
~~~
cd ./Vitis-AI/alveo-hbm
sudo cp alveo_xclbin-1.2.0/U50/6E300M/* /usr/lib
~~~

For Alveo U50LV, use U50LV-9E275M overlay:
~~~
cd ./Vitis-AI/alveo-hbm
sudo cp alveo_xclbin-1.2.0//U50lv/9E275M/* /usr/lib
~~~

For Alveo U50LV, use U50LV-10E275M overlay:
~~~
cd ./Vitis-AI/alveo-hbm
sudo cp alveo_xclbin-1.2.0//U50lv/10E275M/* /usr/lib
~~~

For Alveo U280, use U280-14E300M overlay:
~~~
cd ./Vitis-AI/alveo-hbm
sudo cp alveo_xclbin-1.2.0/U280/14E300M/* /usr/lib
~~~

---
## DPUCAHX8H Overlay Frequency Scaling Down

The maximal core clock frequency listed in this section is the timing sign-off frequency of each overlays, and the overlays run at their maximal core clock by default. However, because of the power limitation of the card, all CNN models on each Alveo card cannot run at all the maximal frequencies listed here. Sometimes frequency scaling-down operation is necessary. For the safe working frequency on each card for the CNN models and corresponding performance, please refer to Chapter 7 of *Vitis AI Library User Guide* (ug1354). **Higher overlay frequencies then the recommendation in ug1354 could cause system reboot or other damage to your system because of the power consumption exceeding of Alveo card over the PCIe power supply limitation.**

The DPUCAHX8H core clock is generated from an internal DCM module driven by the platform Clock_1 with the default value of 100MHz, and the core clock is always linearly proportional to Clock_1. For example, in U50LV-10E275M overlay, the 275MHz core clock is driven by 100MHz clock source. So to set the core clock of this overlay to 220MHz, we need to set the frequency of Clock_1 to (220/275)*100 = 80MHz.

You could use XRT xbutil tools to scale down the running frequency of the DPUCAHX8H overlay before you run the VART/Library examples. Before the frequency scaling-down operation, the overlays should have been programmed into the FPGA first, please refer to the example commands below to program the FPGA and scale down the frequency. These commands will set the Clock_1 to 80MHz and could be run at host or in the docker.

~~~
/opt/xilinx/xrt/bin/xbutil program -p /usr/lib/dpu.xclbin
/opt/xilinx/xrt/bin/xbutil clock -d0 -g 80
~~~
d0 is the Alveo card device number. For more information about **xbutil** tool, please use refer to XRT documents.

---

## Brief Introduction to DPUCAHX8H

DPUCAHX8H is a high performance CNN inference IP optimized for throughput and data center workloads. DPUCAHX8H runs with highly optimized instructions set and supports all mainstream convolutional neural networks, such as VGG, ResNet, GoogLeNet, YOLO, SSD, FPN, etc. 

DPUCAHX8H is one of the fundamental IPs (Overlays) of Xilinx Vitis™ AI development environment, and the user can use Vitis AI toolchain to finish the full stack ML development with DPUCAHX8H. The user can also use standard Vitis flow to finish the integration of DPUCAHX8H with other customized acceleration kernal to realize powerful X+ML solution. DPUCAHX8H is provided as encrypted RTL or XO file format for Vivado or Vitis based integration flow.

The major supported Neural Network operators include:

- Convolution / Deconvolution
- Max pooling / Average pooling
- ReLU, ReLU6, and Leaky ReLU
- Concat
- Elementwise-sum
- Dilation
- Reorg
- Fully connected layer
- Batch Normalization
- Split

DPUCAHX8H is highly configurable, a DPUCAHX8H kernel consists of several Batch Engines, a Instruction Scheduler, a Shared Weights Buffer,  and a Control Register Bank. Following is the block diagram of a DPUCAHX8H kernel including 5 Batch Engines.

<img src = "./images/DPU Kernel Diagram.png" align = "center">

### Batch Engine
Batch Engine is the core computation unit of DPUCAHX8H. A Batch Engine can handle an input image at a time, so multiple Batch Engines in a DPUCAHX8H kenel can process sevel input images simultaneously. The number of Batah Engine in a DPUCAHX8H kernel can be configured based on FPGA resource condition and customer's  performance requirement. For example, in Alveo U280 card, SLR0 (with direct HBM connection) can contain a DPUCAHX8H kernel with maximal four Batch Engines while SLR1 or 2 can contain a DPUCAHX8H kernel with five Batch Engines. In Batch Engine, there is a convolution engine to handle regular convolution/deconvolution compution, and a MISC engine to handle pooling, ReLu, and other miscellaneous operations. MISC engine is also configurable for optional function according specific nerual network requirement. Each Batch Engine use a AXI read/write master interfaces for feature map data exchange between device memory (HBM).

### Instruction Scheduler
Similar to general purpose processor in concept, Instruction Scheduler carries out instruction fetch, decode and dispatch jobs. Since all the Batch Engines in a DPUCAHX8H kernel will run the same nerual network, so Instruction Shceduler serves all the Batch Engines with the same instruction steam. The instruction stream is loaded by host CPU to device memory (HBM) via PCIe interface, and Instruction Scheduler use a AXI read master interface to fetch DPU instruction for Batch Engine.

### Shared Weight Buffer
Shared Weight Buffer includes complex strategy and control logic to manage the loading of nerual network weight from Alveo device memory and transfering them to Batch Engines effeciently. Since all the Batch Engines in a DPUCAHX8H kernel will run the same nerual network, so the weights data are wisely loaded into on-chip buffer and shared by all the Batch Engines to eleminate unnecessary memory access to save bandwidth. Shared Weight Buffer use two AXI read master interfaces to load Weight data from device memory (HBM).

### Control Register Bank
Control Register Bank is the control interface between DPUCAHX8H kernel and host CPU. It implements a set of controler register compliant to Vitis development flow. Control Register Bank has a AXI slave interface.
