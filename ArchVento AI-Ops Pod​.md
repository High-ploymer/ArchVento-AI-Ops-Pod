# 🚀 ArchVento AI-Ops Pod​
# 基于 Arch Linux 与 Ventoy 的便携式高性能 AI 运维平台

[![Arch Linux](https://img.shields.io/badge/OS-Arch%20Linux-blue?logo=archlinux)](https://archlinux.org/)
[![Ventoy](https://img.shields.io/badge/Boot-Ventoy-green?logo=usb)](https://www.ventoy.net/)
[![Docker](https://img.shields.io/badge/Container-Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Ollama](https://img.shields.io/badge/AI-Ollama-black)](https://ollama.com/)
[![Open WebUI](https://img.shields.io/badge/UI-Open%20WebUI-ececec)](https://github.com/open-webui/open-webui)

> **即插即用 · 跨硬件兼容 · 离线 GPU 加速 · 私有化 RAG 知识库**

---

## 📖 项目介绍 (Introduction)

本项目旨在解决传统 AI 运维和教育实训场景中对固定高性能硬件和网络环境的依赖问题我们设计并实现了一套**便携式、高性能的 AI 运维平台**

[cite_start]通过结合 **Arch Linux** 的滚动更新特性与 **Ventoy/vtoyboot** 的虚拟化迁移技术，我们将完整的操作系统封装于 U 盘等移动存储介质中该系统不仅支持在不同物理机上无缝启动，还能直接调用宿主机的 **NVIDIA GPU** 资源，通过 **Docker** 和 **Ollama** 部署本地大模型（LLM）及 **RAG（检索增强生成）** 应用，实现安全、离线、高效的 AI 交互 [cite: 327, 329, 332]

### ✨ 核心特性 (Key Features)

* [cite_start]**💾 便携化部署 (Portable)**: 系统封装于 `.vtoy` 文件中，通过 U 盘即插即用，无需破坏宿主机环境 [cite: 330]
* [cite_start]**🖥️ 跨硬件兼容 (Compatibility)**: 基于 vtoyboot 技术，解决 UUID 引导问题，支持在不同物理机（UEFI 环境）上启动 [cite: 331, 346]
* [cite_start]**🚀 本地 AI 算力 (Local GPU)**: 集成 NVIDIA 驱动与 CUDA 支持，容器化部署 Ollama 与 Open WebUI，充分释放本地 GPU 性能 [cite: 332]
* [cite_start]**🔒 私有知识库 (Private RAG)**: 支持本地文档上传与向量化，构建完全离线的私有知识库问答系统，保障数据隐私 [cite: 333]
* [cite_start]**🛠️ 滚动更新 (Rolling Release)**: 采用 Arch Linux，持续获取最新内核与驱动支持，适配新型 AI 硬件 [cite: 342]

---

## 🏗️ 技术栈 (Tech Stack)

| 类别 | 技术/工具 | 说明 |
| :--- | :--- | :--- |
| **OS** | Arch Linux | [cite_start]核心操作系统，轻量且高度可定制 [cite: 341] |
| **Boot** | Ventoy & vtoyboot | [cite_start]启动管理与 Linux-to-Go 迁移核心组件 [cite: 346] |
| **Virtualization** | Oracle VirtualBox | [cite_start]用于系统初始安装与环境配置 [cite: 355] |
| **Container** | Docker | [cite_start]应用隔离与环境部署 [cite: 351] |
| **Inference** | Ollama (CUDA) | [cite_start]本地 LLM 推理框架，支持 GPU 加速 [cite: 343] |
| **Frontend** | Open WebUI | [cite_start]提供友好的 AI 交互界面与 RAG 支持 [cite: 351] |

---

## 🛠️ 安装与制作指南 (Installation Guide)

### 第一阶段：制作 Linux to Go 基础介质

#### 1. 准备工作
* [cite_start]**软件**: Ventoy2Disk, Oracle VirtualBox, `archlinux-x86_64.iso`, `vtoyboot-1.0.36.iso` [cite: 355, 357, 417]
* [cite_start]**硬件**: 建议 128GB 以上の高速 U 盘 (USB 3.2 Gen2 推荐) [cite: 360, 506]

#### 2. 配置 Ventoy 引导盘
1.  [cite_start]使用 `Ventoy2Disk.exe` 安装 Ventoy 到 U 盘 [cite: 360]
2.  [cite_start](可选) 将 U 盘剩余空间格式化为 NTFS 以便于存储数据 [cite: 362]

#### 3. 创建虚拟机环境
* **新建虚拟机**: 类型选择 Linux / Arch Linux (64-bit)
* **关键设置**:
    * [cite_start]✅ **勾选 `Use EFI`** (必须，否则无法被 Ventoy 引导) [cite: 374]
    * [cite_start]✅ **硬盘选择 `预先分配全部空间`** (防止空间不足) [cite: 376]
    * [cite_start]✅ **关闭软驱**: 在系统设置-主板中，取消勾选软驱，并确保 UEFI 开启 [cite: 382]

#### 4. 安装 Arch Linux
1.  [cite_start]启动虚拟机，使用官方脚本 `archinstall` 进行自动化安装 [cite: 388]
2.  **配置建议**:
    * [cite_start]Disk configuration: `btrfs` (支持快照) [cite: 390]
    * Profile: `Desktop` -> `KDE` (或其他喜欢的桌面)
    * Driver: `nvidia open driver`
3.  [cite_start]**解决中文乱码**: 安装字体并刷新缓存 [cite: 397]
    ```bash
    sudo pacman -S adobe-source-han-sans-cn-fonts wqy-microhei wqy-zenhei noto-fonts-cjk
    fc-cache -fv
    ```

### 第二阶段：系统迁移配置 (关键步骤)

在虚拟机内执行以下操作，以确保系统能在物理机 U 盘上启动

#### 1. 修正 UUID
[cite_start]虚拟机默认使用 PARTUUID，迁移后会失效，需替换为文件系统 UUID [cite: 405]
```bash
# 查看 PARTUUID 和 UUID
cat /boot/loader/entries/*.conf | grep PARTUUID
blkid

# 替换配置 (将下方 UUID 替换为你实际查询到的值)
sudo sed -i.bak 's/PARTUUID=你的旧PARTUUID/UUID=你的新UUID/' /boot/loader/entries/*.conf
```
#### 2. 配置 GRUB
Ventoy 需要 Grub 引导 
```bash
sudo pacman -S --needed grub lvm2
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --removable
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
#### 3. 运行 vtoyboot
1.  挂载 `vtoyboot-1.0.36.iso` 到虚拟机光驱 
2.  解压并运行脚本：
```bash
tar -xzf vtoyboot-1.0.36.tar.gz
cd vtoyboot-1.0.36
sudo ./vtoyboot.sh
```
当看到 `vtoyboot process successfully finished` 即表示成功 
#### 4.迁移到 U 盘
1. 关闭虚拟机
2. 找到虚拟磁盘文件 `.vdi`，将其移动到 Ventoy U 盘中
3. 重命名后缀: 将 `.vdi` 改名为 `.vtoy` (例如 `archlinux.vtoy`) 

---
## 🤖 AI 环境搭建 (AI Environment Setup)
重启电脑，进入 BIOS 选择 U 盘启动，在 Ventoy 菜单中选择 `archlinux.vtoy` 进入系统 

1. **安装 Ollama (带 CUDA 支持)**   
使用 AUR 助手 (如 paru) 安装支持 GPU 加速的版本 
```bash
# 安装 ollama-cuda
paru -S ollama-cuda
# 启动服务
sudo systemctl start ollama
# 测试运行模型
ollama run olmo-3:7b
```
2. **配置 Docker 与 NVIDIA Container Toolkit**  
配置 Docker 以支持 NVIDIA GPU 调用
```bash
# 1. 安装 Docker
paru -S docker
sudo systemctl start docker
sudo usermod -aG docker $USER  # 将当前用户加入 docker 组，免 sudo 运行

# 2. 安装 NVIDIA 工具包
sudo pacman -S nvidia-container-toolkit

# 3. 配置 Docker 运行时
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```
配置完成后，检查 `/etc/docker/daemon.json` 确认 runtime 已更新 

3. **部署 Open WebUI**  
运行支持 GPU 的 Open WebUI 容器，并连接到宿主机的 Ollama 
```bash
docker run -d -p 3000:8080 --gpus all \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:cuda
```
⚠️ 重要提示: 默认情况下 Docker 容器无法访问宿主机的 Ollama需修改 Ollama 服务配置以允许外部调用 

编辑服务文件：`sudo systemctl edit ollama.service`

添加环境变量：`Environment="OLLAMA_HOST=0.0.0.0"`

重启服务：`sudo systemctl restart ollama`

---
## 💡 使用指南: 本地知识库 (RAG)
通过 RAG 技术，让 AI 基于您的私有文档回答问题 

1.  访问界面: 浏览器打开 http://localhost:3000 并注册管理员账号 

2.  创建知识库:

        进入 工作空间 -> 知识库 -> 点击 + 创建知识库

        填写名称（如“自我介绍”）并上传您的私有文档 (支持 .txt, .pdf, .md 等) 

3.  创建模型:

        进入 工作空间 -> 模型 -> 点击 + 创建模型
        基础模型: 选择 olmo-3:7b (或您下载的其他模型)
        知识库: 在选项中勾选刚才创建的库 

4.  开始对话:

        在主界面左上角选择新创建的模型
        提问与文档相关的内容，系统将检索文档并生成精准回答

--- 
## ❓ 常见问题 (FAQ)
### Q: 为什么启动时无法进入图形界面？

A: 请检查物理机的显卡驱动是否兼容本项目默认安装了 Nvidia 驱动，如果是 AMD 或 Intel 核显机器，可能需要手动安装对应的 Mesa 驱动 

### Q: Open WebUI 找不到 Ollama 的模型？
A: 这是网络权限问题请确保 Ollama 服务文件 (`/etc/systemd/system/ollama.service`) 中设置了 `Environment="OLLAMA_HOST=0.0.0.0"`，并执行了 `systemctl daemon-reload 和 systemctl restart ollama `

### Q: 系统运行速度慢？

A: 受限于 U 盘 I/O 速度建议使用固态 U 盘 (USSD) 或移动 SSD (USB 3.2/Type-C)，能显著提升模型加载和系统响应速度 

---
## 🔮 未来规划 (Roadmap)
[ ] 网络优化: 配置 Nginx 反向代理与 SSL 加密，支持安全的远程访问 

[ ] 集群扩展: 探索 Docker Swarm/K8s 多节点部署，实现算力聚合 

[ ] 硬件适配: 增加对 AMD GPU (ROCm) 的开箱即用支持
 
---

## 📄 附录：常用命令速查
```bash
# 刷新字体缓存 (解决乱码)
fc-cache -fv
# 重新生成 Grub 配置
sudo grub-mkconfig -o /boot/grub/grub.cfg
# 启动 Docker 服务
sudo systemctl start docker.service
# 查看 Docker 容器状态
docker ps
# 重启 Ollama 服务
sudo systemctl restart ollama
```
---
### Project Maintainer: 
👤 **Gao Guozhen (Project Leader)**

**👤 Qiao Keao (Member)**

🏫 **Qinghai University - School of Computer Technology and Application**