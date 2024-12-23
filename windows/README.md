# Packer Template for Microsoft Windows

## Introduction

The Packer templates in this directory creates Windows Server images for use with MAAS.
测试下来，发现ubuntu22.04版本对构建windwos镜像有部分未知为题，导致构建失败，最终选用24.04环境成功构建并适用maas。



## Prerequisites (to create the image)

* A machine running Ubuntu 18.04+ with the ability to run KVM virtual machines.
* qemu-utils, libnbd-bin, nbdkit and fuse2fs
* qemu-system
* ovmf
* cloud-image-utils
* [Packer](https://www.packer.io/intro/getting-started/install.html), v1.7.0 or newer
* Ubuntu 22.04+ is required to build Windows 11 images (swtpm package)


## Requirements (to deploy the image)

* [MAAS](https://maas.io) 3.2+
* [Curtin](https://launchpad.net/curtin) 21.0+


## Supported Microsoft Windows Versions

This process has been build and deployment tested with the following versions of
Microsoft Windows:

* Windows Server 2025
* Windows Server 2022
* Windows Server 2019
* Windows Server 2016
* Windows 10 PRO+
* Windows 11 PRO+


## Known Limitations

* The current process builds UEFI compatible images only.


## windows.pkr.hcl Template

This template builds a dd.tgz MAAS image from an official Microsoft Windows ISO/VHDX. 
This process also installs the latest VirtIO drivers as well as Cloudbase-init.


## Obtaining Microsoft Windows ISO images

You can obtains Microsoft Windows Evaluation ISO/VHDX images from the following links:

* [Windows Server 2025](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2025)
* [Windows Server 2022](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022)
* [Windows Server 2019](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
* [Windows Server 2016](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2016)
* [Windows 10 Enterprise](https://www.microsoft.com/en-us/evalcenter/download-windows-10-enterprise)
* [Windows 11 Enterprise](https://www.microsoft.com/en-us/evalcenter/download-windows-11-enterprise)



### 个人经验
只需修改http/Autounattend.xml.ISO.template
```shell
1.修改安装版本
<Value>@VERSION@</Value>
#@VERSION@修改指定版本，如下选择：
#windwos10/11：

Windows 10/11 PRO  #专业版,这个版本用的最多
Windows 10/11 Enterprise  #企业版，可选

windows sever:
Windows Server 2019/2022/2025 Datacenter（） #数据中心版本，功能最完善，建议选择
Windows Server 2019/2022/2025 Standard（）   #标准版本功能比Datacenter比有限制

2.修改key：
示例：
                <ProductKey>
                    <Key>WX4NM-KYWYW-QJJR4-XV3QB-6VM33</Key>  #2022版本key
                    <WillShowUI>Never</WillShowUI>
                </ProductKey>
多版本可用key：
2022标准版：CR9K8-RDN9C-VD8FD-WK49B-KD97B
2022数据版本：Q4N8P-8BV9D-8MR8W-3VY9F-CR7JH / WX4NM-KYWYW-QJJR4-XV3QB-6VM33
2025标准版：TVRH6-WHNXV-R9WG3-9XRFY-MY832
2025数据版本：D764K-2NDRG-47T6Q-P8T8W-YP6DF


3.语言修改，选择性修改
#两个组件<component></component>内修改语言：

简体中文：zh-CN
英文：en-US

4.修改登录账户密码：（可不改）
UserAccounts内修改：<Value>Passw0rd</Value>
默认密码为，可修改统一密码：Passw0rd
 
5.修改时区，默认时区是：UTC（选择性修改）
可选修改为：CST
      
````

### 建议修改Makefile：
``` shell
vim Makefile
PACKER_LOG ?= 1 #输出构建详细日志，建议开启，默认是0不开启
```

### Building the image

The build the image you give the template a script which has all the
customization:

```shell
sudo make windows ISO=<path-to-iso> VERSION=<windows-version>
```

Example:

```shell
sudo make ISO=/mnt/iso/Windows_Server_2025_SERVER_EVAL_x64FRE_en-us.iso VERSION=2025
```

### Makefile Parameters

#### BOOT

Currently uefi is the only supported value.

#### EDIT

The edition of a targeted ISO image. It defaults to PRO for Microsoft Windows 10/11
and SERVERSTANDARD for Microsoft Windows Servers. Many Microsoft Windows Server ISO
images do contain multiple editions and this prarameter is useful to build a particular
edition such as Standard or Datacenter etc.

#### HEADLESS

Whether VNC viewer should not be launched. Default is set to false.

#### ISO

Path to Microsoft Windows ISO image used to build the image.

#### PACKER_LOG

Enable (1) or Disable (0) verbose packer logs. The default value is set to 0.

#### PKEY

User supplied Microsoft Windows Product Key. When usimg KMS, you can obtain the
activation keys from the link below:

* [KMS Client Activation and Product Keys](https://learn.microsoft.com/en-us/windows-server/get-started/kms-client-activation-keys)

Please note that PKEY is an optional parameter but it might be required during
the build time depending on the type of ISO being used. Evaluation series ISO
images usually do not require a product key to proceed, however this is not
true with Enterprise and Retail ISO images.

#### TIMEOUT

Defaults to 1h. Supports variables in h (hour) and m (Minutes).

#### VHDX

Path to Microsoft Windows VHDX image used to build the image.

#### VERSION

Specify the Microsoft Windows Version. Example inputs include: 2025, 2022, 2019, 2016, 10
and 11.

个人经验:

## Uploading images to MAAS

```shell
maas login admin http:/maas-ip:5240/MAAS/api/2.0/ 
```

Use MAAS CLI to upload the image:

```shell
maas admin boot-resources create \
    name='windows/windows-server' \
    title='Windows Server' \
    architecture='amd64/generic' \
    filetype='ddtgz' \
    content@=windows-server-amd64-root-dd.gz
```
