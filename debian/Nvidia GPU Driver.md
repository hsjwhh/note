在 Ubuntu Server 上安装 NVIDIA 显卡驱动，通常可以使用 `ubuntu-drivers` 或直接从 NVIDIA 官方网站下载驱动进行安装。以下是详细步骤：

---

## **方法 1：使用 Ubuntu 官方驱动（推荐）**
Ubuntu 提供了官方的 `ubuntu-drivers` 工具来自动安装推荐的 NVIDIA 驱动，适用于大多数情况。

### **1. 更新系统**
```bash
sudo apt update && sudo apt upgrade -y
```

### **2. 检查可用的 NVIDIA 驱动**
```bash
ubuntu-drivers devices
```
示例输出：
```
== /sys/devices/pci0000:00/0000:00:1c.0/0000:01:00.0 ==
vendor   : NVIDIA Corporation
model    : TU104 [GeForce RTX 2080 SUPER]
driver   : nvidia-driver-535 - distro non-free recommended
driver   : nvidia-driver-470 - distro non-free
```
`recommended` 说明是推荐的驱动。

### **3. 安装推荐的驱动**
```bash
sudo apt install -y nvidia-driver-535
```
或自动安装最佳驱动：
```bash
sudo ubuntu-drivers install
```

### **4. 重启服务器**
```bash
sudo reboot
```

### **5. 验证驱动安装**
```bash
nvidia-smi
```
如果驱动安装成功，会显示 NVIDIA GPU 详情和驱动版本。

---

## **方法 2：使用 NVIDIA 官方驱动（适用于新款 GPU 或特殊需求）**
如果 Ubuntu 官方源的驱动版本较旧，或者你需要更高级的 CUDA 计算功能，可以直接从 NVIDIA 官网安装驱动。

### **1. 禁用 Nouveau**
```bash
sudo bash -c "echo -e 'blacklist nouveau\noptions nouveau modeset=0'" > /etc/modprobe.d/blacklist-nouveau.conf
```
```bash
sudo update-initramfs -u
```

### **2. 重新启动服务器**
```bash
sudo reboot
```

### **3. 下载 NVIDIA 官方驱动**
访问 [NVIDIA 官网](https://www.nvidia.com/download/index.aspx) 下载适用于你的显卡和 Ubuntu 版本的 `.run` 安装包。

例如：
```bash
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/535.154.05/NVIDIA-Linux-x86_64-535.154.05.run
```

### **4. 赋予执行权限**
```bash
chmod +x NVIDIA-Linux-x86_64-535.154.05.run
```

### **5. 进入纯命令行模式**
```bash
sudo systemctl set-default multi-user.target
sudo reboot
```
然后以 root 用户登录。

### **6. 运行 NVIDIA 驱动安装**
```bash
sudo ./NVIDIA-Linux-x86_64-535.154.05.run
```
在安装过程中：
- 选择 **Yes** 以安装 DKMS（可选，但推荐）。
- 如果系统没有现有的 X Server，安装时可能会提示 “X Server is running”，可以选择继续。

### **7. 恢复 GUI 模式（可选）**
如果服务器需要图形界面：
```bash
sudo systemctl set-default graphical.target
sudo reboot
```

### **8. 验证驱动**
```bash
nvidia-smi
```

---

## **方法 3：使用 PPA 方式安装（适用于最新驱动）**
如果想要安装最新的 NVIDIA 驱动，可以使用 `graphics-drivers` PPA：

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa -y
sudo apt update
```
然后使用 `ubuntu-drivers devices` 检查并安装推荐的驱动。

---

## **卸载 NVIDIA 驱动**
如果需要卸载已安装的 NVIDIA 驱动：
```bash
sudo apt remove --purge '^nvidia-.*'
sudo apt autoremove
sudo reboot
```
或者，如果是 `.run` 文件安装的：
```bash
sudo ./NVIDIA-Linux-x86_64-535.154.05.run --uninstall
```

---

## **总结**
| 方法 | 适用场景 |
|------|---------|
| `ubuntu-drivers` | 官方推荐，适用于大多数情况 |
| NVIDIA 官网 `.run` | 适用于较新 GPU 或特殊需求 |
| PPA 源安装 | 获取最新驱动 |

如果是服务器使用，一般推荐 **方法 1**，稳定且容易维护。如果需要最新功能，可以考虑 **方法 3** 或 **方法 2**。