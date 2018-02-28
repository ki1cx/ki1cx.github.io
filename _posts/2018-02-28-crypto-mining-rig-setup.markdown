---
layout: single
title: "Crypto Mining Rig Setup"
description: Instructions on how to setup a Nvidia GTX 10 series based crypto mining rig with automated monitoring to ensure crashed GPUs are restored.
date:   2018-02-28 19:20:22 -0000
categories: linux cryptocurrency ethereum 
---

This is a guide on how to build and setup an Ethereum mining rig using Nvidia GTX 10 series GPUs using the Claymore Dual miner. You can easily mine other cryptos that the Claymore miner supports as well as install other miners to mine other coins. The github project can be found [here](https://github.com/ki1cx/crypto-miner-server).

## Compatibility

Tested on Ubuntu 16.04 Server LTS amd64 Xenial Xerus. Setup with [ubuntu-unattended](https://github.com/ki1cx/ubuntu-unattended)

## Hardware List

Here is the list of hardware that I've use to build the rig. 

* **Motherboard** - MSI Pro Series Intel Z270 DDR4 HDMI USB 3 SLI ATX Motherboard [Z270 SLI PLUS](https://www.amazon.com/MSI-Z270-SLI-Motherboard-PLUS/dp/B01MR32I8L/)

* **CPU** - Intel CPU [BX80662G3900](https://www.amazon.com/gp/product/B01B2PJRPA) Celeron G3900 2.80Ghz 2M LGA1151 2C/2T Skylake

* **Memory** - CORSAIR Vengeance LPX 8GB (2x4GB) DDR4 DRAM 3000MHz C15 Memory Kit - Black [CMK8GX4M2B3000C15](https://www.amazon.com/gp/product/B0123ZBPDA/) (only need a single 4GB)

* **Power Supply** - Corsair RMx Series, 850W, Fully Modular Power Supply, 80+ Gold Certified [RM850x](https://www.amazon.com/dp/B015YEI8JG)

* **SSD** - Transcend 64 GB SATA III MTS600 60 mm M.2 SSD [TS64GMTS600](https://www.amazon.com/gp/product/B00KLTPVJ0)

* **USB Wifi Dongle** - [OURLINK](https://www.amazon.com/gp/product/B018TX8IDA) 600Mbps mini 802.11ac Dual Band 2.4G/5G Wireless Network Adapter USB Wi-Fi Dongle

* **PCIe Risers** - [MintCell](https://www.amazon.com/gp/product/B06ZY2R85P) 6-Pack PCIe 6-Pin 16x to 1x Powered Riser Adapter Card w/ 60cm USB 3.0 Extension Cable & 6-Pin PCI-E to SATA Power Cable 

* **GPU** - Nvidia GTX 10 series GPUs

## On GPU selection

When choosing GPUs to build with, here are the things to consider.

### Brand - Nvidia vs AMD

I chose to use Nvidia as opposed to AMD. From my research, Nvidia is much easier to overclock when using Linux. Although AMD has been the king for mining for sometime, Nvidia has a much more power efficient architecture, and it will save you on energy cost over time.

### GPU's Cooling method - blow style (fully enclosed case) vs open air (open case)

| Cooling | Example | Pros | Cons |
|---|---|---|---|
|  Open-Air  |  <img src="https://images-na.ssl-images-amazon.com/images/I/71QpPE6HUxL._SX522_.jpg" alt="open air" style="width: 200px;"/> [^1] | quiet, greater supply | dust accumulation, distributor markup |
| Closed / Blower  |   <img src="https://images-na.ssl-images-amazon.com/images/I/41jAgpWLOoL.jpg" alt="blow style" style="width: 200px;"/> [^2]  | solid construction, directed air flow, fixed price when purchased from manufacturer | noisy |
| Closed / Water-Cooled  |   <img src="https://thumbor.forbes.com/thumbor/960x0/smart/https%3A%2F%2Fblogs-images.forbes.com%2Fmarcochiappetta%2Ffiles%2F2015%2F05%2Fevga-titan.jpg" alt="blow style" style="width: 200px;"/> [^3]  | solid construction, directed air flow, silent | expensive |
	
I have tried the first two mentioned in the above table. If you are going to have a single rig of ~ 6 GPUS running in your bedroom, then the open air design will keep the noise down, because the fan does not have to run as fast compared to the closed air (blower) design. 
	
<img src="http://cdn.shopify.com/s/files/1/1952/1205/products/DSC_0030_1024x1024.jpg" alt="open air" style="width: 200px;"/>
	
If you are planning on a larger operation with ~ 100s of GPUs... then you'll probably want the blower style GPUs so you can carefuly direct the heat away from the heat sensitive components. 
	
<img src="https://i.pinimg.com/736x/fe/e8/94/fee894f88897840885e4bd36d6b4420e--rigs.jpg" alt="open air" style="width: 200px;"/>

### Where to buy

Best place to buy Nvidia GPUs is to go to the [source](https://www.nvidia.com/en-us/geforce/products/10series/geforce-store/), since there are no additional markups. However, you'll likely need to wait until supply meets demand. If you want to get your hands on them sooner than later and don't mind the premium, then Amazon would be your best bet.

## On GPU Overclocking

### Controlling Memory speed

When mining Ethereum, you have to overclock the memeory speed to get performance boost. Modifying the memory transfer rate is easy the following. 

```bash
nvidia-settings -c :0 -a GPUMemoryTransferRateOffset[3]=<offset value to test>
```

Here is what I found to work in giving me a boost while remaining stable. I have not had a chance to try an 1080 Ti yet. 1080 (yes without the Ti) is widely known to be not good for mining due to it's GDDR5X memory.

| GPU | GPUMemoryTransferRateOffset |
|---|---|
| 1060 | 1300 | 
| 1070 | 1400 | 

### Controlling Power Draw

Giving the GPU enough is a crucial step in stablizing the GPU

```bash
//set persistence mode on
nvidia-smi -pm 1

//set upper power limit in watts
nvidia-smi -pl 75
```

The following chart shows the power limit I've used to stablize the GPUMemoryTransferRateOffset used above.

| GPU | Power Limit |
|---|---|
| 1060 | 75 | 
| 1070 | 100 | 

### Ethereum Hashrates

The GTX 1070s is a clear winner when it comes to mining Ethereum.

| GPU | Default | Overclocked | Price | cost per MH/s|
|---|---|---|---|---|
| 1060 | 18 MH/s | 23 MH/s| $299 | $13 |
| 1070 | 25 MH/s | 31 MH/s| $399 | $12 |
| 1080 Ti | 32 MH/s | 36 MH/s| $699 | $19 |

### Controling the fan speed

In order to control the fan speed, you'll need to set manual mode, which means the GPU will no longer automatically adjust the fan speed to the changing temperature. Make sure you are frequently running the custom script that adjusts the fan speed included in the repo or a custom script of your own.

```bash
//set fan speed across all GPUs
nvidia-settings -c :0 -a GPUFanControlState=1 -a GPUTargetFanSpeed=50

//set fan speed on a specific GPUs where <index> 
//should be replaced by the gpu you are targetting
nvidia-settings -c :0 -a [gpu:<index>]/GPUFanControlState=1 -a [fan:<index>]/GPUTargetFanSpeed=100
```

## On USB Wifi Adapters

Make sure you find one that is compatible with linux. Manufacturers may claim it works on Linux with the provided drivers, but I found them not trustworthy. I ended up finding a usable driver on github.

## On PCIe Risers

When choosing PCIe risers, make sure they are **VER 006C** or higher. Specifically, check if they have 4 high quality solid capacitors for voltage regulation and overcurrent protection. Also, chances are that 1 in 6 risers will be bad, so buy them in bulk to keep cost down.

## References

[^1]: https://www.amazon.com/EVGA-GeForce-Support-Graphics-06G-P4-6267-KR/dp/B01LYN9KK6
[^2]: https://www.nvidia.com/en-us/geforce/products/10series/geforce-store/
[^3]: https://www.forbes.com/sites/marcochiappetta/2015/05/29/evga-steps-out-with-custom-water-cooled-geforce-gtx-titan-x-graphics-card/#17db683c6787


