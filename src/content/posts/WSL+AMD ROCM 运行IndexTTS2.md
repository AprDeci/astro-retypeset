---
title: "WSL+AMD ROCM 运行IndexTTS2"
published: 2025-11-12
tags:
  - rocm
  - ai
  - tts
draft: false
toc: true
lang: zh
---
# WSL+AMD ROCM 运行IndexTTS2

## 前言

工作后组装了新电脑,在显卡的选择上犹豫再三,rtx5070,rtx5070ti,rx9070xt,想要N卡但是70ti略贵70显存小,9070xt很好但是没有CUDA.最终还是选择了性价比.

ROCM我也一直在关注,感觉越来越好了呢.

这次教程主要还是会讲原理,其实就是把依赖的pytorch修改成rocm适配的pytorch,并且加入一些其他操作即可.

我想现在rocm7出来了应该在**windows上原生运行**是没问题的,都是通用的,大家可以试试.

## 环境列表

| 环境    | 版本          |
| ------- | ------------- |
| 系统    | wsl2 ubuntu24 |
| Rocm    | 6.4           |
| pytorch | 2.6           |

emm,其实感觉也没什么特别要列出来的,环境应该不是特定的只要保证ROCM和pytorch的版本对应即可

## 正式教程

### 克隆indextts2 准备环境

这一步很简单,跟随官方仓库Readme就可以了,我不再多赘述

```sh
git lfs install
```

```
git clone https://github.com/index-tts/index-tts.git && cd index-tts
git lfs pull  # download large repository files
```

安装uv

[安装 | uv 中文文档](https://uv.doczh.com/getting-started/installation/)

## Ubuntu安装ROCM

这里就是按照官方教程安装,我只是复述一遍,原文在下

[Install Radeon software for WSL with ROCm — Use ROCm on Radeon and Ryzen](https://rocm.docs.amd.com/projects/radeon-ryzen/en/latest/docs/install/installrad/wsl/install-radeon.html)

**安装 AMD 统一驱动程序软件包存储库和安装脚本**

```sh
sudo apt update
wget https://repo.radeon.com/amdgpu-install/6.4.2.1/ubuntu/noble/amdgpu-install_6.4.60402-1_all.deb
sudo apt install ./amdgpu-install_6.4.60402-1_all.deb
```

```
amdgpu-install -y --usecase=wsl,rocm --no-dkms
```

**安装后验证检查**

```sh
rocminfo
```

如果能正确打印你的显卡信息就是安装完成了

## 修改pyproject.toml

这一步就是最关键的,我们要修改依赖中的torch依赖和其他依赖信息

**下载torch wheel**

rocm使用的pytorch是amd官方修改过的,所以我们要单独的下载下来再安装,indexTTS不需要torchvision所以我们不下载(更多版本下载地址看下面链接的仓库就可以了https://repo.radeon.com/rocm)

```sh
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.4.2/torch-2.6.0%2Brocm6.4.2.git76481f7c-cp312-cp312-linux_x86_64.whl
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.4.2/pytorch_triton_rocm-3.2.0%2Brocm6.4.2.git7e948ebf-cp312-cp312-linux_x86_64.whl
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.4.2/torchaudio-2.6.0%2Brocm6.4.2.gitd8831425-cp312-cp312-linux_x86_64.whl
```

然后修改pyproject.toml. 首先是python版本修改

requires-python = "==3.12.*"

然后是依赖,修改numba适配python3.12,修改torch安装为本地安装

![image-20251031225904743](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/img/image-20251031225904743.png)

将以下部分注释掉或者删掉

```toml
[tool.uv.sources]
# Install PyTorch with CUDA support on Linux/Windows (CUDA doesn't exist for Mac).
# NOTE: We must explicitly request them as `dependencies` above. These improved
# versions will not be selected if they're only third-party dependencies.
torch = [
  { index = "pytorch-cuda", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
]
torchaudio = [
  { index = "pytorch-cuda", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
]
torchvision = [
  { index = "pytorch-cuda", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
]
```

结尾添加如下

```toml

[tool.hatch.metadata]

allow-direct-references = true
```

最后修改如下

![image-20251031225226462](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/img/image-20251031225226462.png)

## 安装依赖

```sh
uv venv
```

```sh
uv sync --all-extras
```

如果是国内用户

```sh
uv sync --all-extras --default-index "https://mirrors.aliyun.com/pypi/simple"

uv sync --all-extras --default-index "https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
```

**这时候你可以运行如下命令检测程序是否可以检测到gpu**

```sh
uv run tools/gpu_check.py
```

下载模型

```sh
uv tool install "huggingface-hub[cli,hf_xet]"

hf download IndexTeam/IndexTTS-2 --local-dir=checkpoints
```

或

```sh
uv tool install "modelscope"

modelscope download --model IndexTeam/IndexTTS-2 --local_dir checkpoints
```

**如果你是wsl还需要做下面一步(要在虚拟环境里哦)**

```sh
location=$(pip show torch | grep Location | awk -F ": " '{print $2}')
cd ${location}/torch/lib/
rm libhsa-runtime64.so*
```

以上,就是所有操作了,当你下载完依赖和模型就可以运行以下命令启动了!

```
uv run webui.py
```

或

```sh
uv run webui.py --fp16
```

## 常见问题

如果你遇到了生成很慢,并且提示

```sh
MIOpen(HIP): Warning [IsEnoughWorkspace] [GetSolutionsFallback WTI] Solver <GemmFwdRest>, workspace required: 37527552, provided ptr: 0 size: 0
```

你可以尝试如下命令,亲测有效

```sh
export MIOPEN_FIND_MODE=FAST
export MIOPEN_USER_DB_PATH="~/tts/miopen_cache"
```

(来源:[[ROCm\] PyTorch 在 TTS 上运行缓慢 · 问题 #150168 · pytorch/pytorch --- [ROCm] PyTorch slow on TTS · Issue #150168 · pytorch/pytorch](https://github.com/pytorch/pytorch/issues/150168))

## 结尾

最后来一张实测图

![image-20251031231007601](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/img/image-20251031231007601.png)
