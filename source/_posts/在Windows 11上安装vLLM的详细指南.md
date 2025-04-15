---
title: 在Windows 11上安装vLLM的详细指南
date: 2025-04-09
categories:
  - 
  - 
tags:
  - 大模型
  - vLLM
---

## 重要说明

vLLM官方不直接支持Windows，它主要设计用于Linux和类Unix系统。但是，您有几种方法可以在Windows 11上使用vLLM：

1. 使用WSL2 (Windows Subsystem for Linux)

2. 使用Docker容器
3. 尝试直接安装（有限支持，不推荐）

下面我将详细介绍前两种方法，它们是最可靠的选择。

## 方法1：通过WSL2安装vLLM（推荐）

WSL2提供了在Windows上运行Linux的完整环境，这是运行vLLM的最佳选择。

### 步骤1：安装WSL2

1. 以管理员身份

   打开PowerShell，运行：

   ```sh
   wsl --install
   ```

2. 重启计算机。

3. 重启后，系统会自动启动Ubuntu安装。创建用户名和密码。

4. 如果未自动安装Ubuntu，请手动安装：

打开PowerShell，运行

```sh
wsl --install -d Ubuntu-22.04
```

### 步骤2：配置WSL2以使用GPU

1. 下载并安装最新的NVIDIA驱动（确保选择支持WSL的版本）

2. 在PowerShell中检查WSL是否可以访问GPU：

打开PowerShell，运行

 ```sh
 wsl --list --verbose
 ```

确保显示的版本是"2"。

3. 在WSL Ubuntu中安装CUDA：

```bash
# 进入WSL
wsl
# 更新软件包
sudo apt update && sudo apt upgrade -y
# 安装依赖
sudo apt install -y build-essential wget
# 添加CUDA存储库
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
# 安装CUDA（不带显示驱动）
sudo apt install -y cuda-toolkit-12-3
```

4. 添加CUDA到路径：

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

5. 验证CUDA安装：

```bash
nvidia-smi
nvcc --version
```

### 步骤3：安装vLLM

1. 安装Python和虚拟环境：

```bash
# 安装Python和pip
sudo apt install -y python3 python3-pip python3-venv
# 创建并激活虚拟环境
mkdir -p ~/vllm-env
python3 -m venv ~/vllm-env/venv
source ~/vllm-env/venv/bin/activate
```

2. 安装PyTorch：

```bash
pip install torch torchvision torchaudio
```

3. 安装vLLM：

```bash
pip install vllm
#下载慢可用
pip install vllm -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 步骤4：运行vLLM服务器

1. 下载并运行模型：

```bash
# 激活虚拟环境（如果还没激活）
source ~/vllm-env/venv/bin/activate
# 运行vLLM与DeepSeek-r1 7B模型
python -m vllm.entrypoints.openai.api_server \
    --model deepseek/deepseek-r1-7b \
    --gpu-memory-utilization 0.9 \
    --max-model-len 8192 \
    --max-num-batched-tokens 4096 \
    --dtype half
```

2. 服务器将在http://localhost:8000启动。在Windows中，您可以通过此地址访问API。

## 方法2：使用Docker安装vLLM

如果您更喜欢Docker方式，这也是一个可行的选择。

### 步骤1：安装Docker Desktop

1. 从Docker官网下载并安装Docker Desktop。

1. 启动Docker Desktop，确保在设置中启用了WSL2集成。

1. 确保在Docker设置中启用了NVIDIA GPU支持。

### 步骤2：安装NVIDIA Container Toolkit

1. 在WSL2中运行以下命令：

```bash
# 进入WSL
wsl
# 添加NVIDIA Docker存储库
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
# 更新并安装
sudo apt update
sudo apt install -y nvidia-container-toolkit
# 重新启动Docker
sudo systemctl restart docker
```


### 步骤3：运行vLLM Docker容器

1. 在PowerShell或WSL中运行以下命令：

```bash
# 创建目录用于模型缓存
mkdir -p ~/models
# 运行vLLM容器
docker run --gpus all -p 8000:8000 \
--volume ~/models:/data \
ghcr.io/vllm-project/vllm:latest \
--model deepseek/deepseek-r1-7b \
--gpu-memory-utilization 0.9 \
--max-model-len 8192 \
--max-num-batched-tokens 4096 \
--dtype half \
--download-dir /data
```
2. 服务器将在http://localhost:8000启动。

## 测试vLLM是否工作正常

无论使用WSL2还是Docker，您都可以按以下方式测试vLLM：

1. 在Windows PowerShell中运行

```powershell
Invoke-WebRequest -Uri http://localhost:8000/v1/models -Method GET
```

2. 或者发送聊天请求：

```powershell
$body = @{
	model = "deepseek/deepseek-r1-7b"
	messages = @(
		@{
			role = "user"
			content = "简要解释下什么是数字孪生技术"
		}
	)
	temperature = 0.7
} | ConvertTo-Json
Invoke-WebRequest -Uri http://localhost:8000/v1/chat/completions -Method POST -Body $body -ContentType "application/json"
```
## 针对Windows的优化建议

1. WSL2内存管理：创建或修改.wslconfig文件限制WSL2内存使用：

```text
# 在Windows中的路径：C:\Users\<用户名>\.wslconfig
[wsl2]
memory=32GB
swap=8GB
processors=8
```

2. GPU监控：使用Windows任务管理器或NVIDIA控制面板监控GPU使用情况。

3. 自动启动脚本：创建批处理文件以简化vLLM启动：

```batch
@echo off
wsl -d Ubuntu-22.04 -u <用户名> bash -ic "source ~/vllm-env/venv/bin/activate && python -m vllm.entrypoints.openai.api_server --model deepseek/deepseek-r1-7b --gpu-memory-utilization 0.9 --max-model-len 8192 --max-num-batched-tokens 4096 --dtype half"
pause
```

## 故障排除

### 问题1：CUDA不可用或GPU未检测到

解决方案：

- 确保安装了最新的NVIDIA驱动

- 在WSL中运行nvidia-smi验证GPU可见

- 检查CUDA路径是否正确设置

```bash
# 手动测试CUDA
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.device_count())"
```

### 问题2：内存或显存不足错误

解决方案：

- 调整vLLM参数减少资源使用：

  ```bash
  # 减少资源使用的启动选项
  python -m vllm.entrypoints.openai.api_server \
      --model deepseek/deepseek-r1-7b \
      --gpu-memory-utilization 0.8 \
      --max-model-len 4096 \
      --max-num-batched-tokens 2048 \
      --dtype half
  ```

### 问题3：模型下载问题

解决方案：

- 手动下载模型并使用本地路径：

  ```bash
  # 使用Hugging Face CLI下载
  pip install huggingface_hub
  huggingface-cli download deepseek/deepseek-r1-7b --local-dir ~/models/deepseek-r1-7b
  # 使用本地模型
  python -m vllm.entrypoints.openai.api_server \
      --model ~/models/deepseek-r1-7b \
      --gpu-memory-utilization 0.9 \
      --max-model-len 8192 \
      --dtype half
  ```

  
