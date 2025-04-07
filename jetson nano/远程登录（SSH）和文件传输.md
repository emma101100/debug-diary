#  远程登录（SSH）和文件传输

## 遇到的问题与解决办法

### 远程登录相关问题

#### 问题：什么是远程登录？怎么操作？
**描述**：不清楚“远程登录”概念，如何用 PuTTY 实现。
**解答**：
- 远程登录是通过网络从一台设备（PC）连接到另一台（Jetson Nano），以操作后者。
- 用 PuTTY（Windows 工具）通过 SSH 协议实现，主要用命令行操作。
- 步骤：
  1. Windows 上下载 PuTTY（从 `www.putty.org`）。
  2. Jetson Nano 上确认 IP（用 `ifconfig`）。
  3. Nano 上开启 SSH 服务：
     ```bash
     sudo apt update
     sudo apt install openssh-server
     sudo systemctl enable ssh
     sudo systemctl start ssh
     ```
  4. PuTTY 输入 Nano 的 IP（端口 22，选 SSH），登录时输入 Nano 的用户名和密码。

#### 问题：PuTTY 下载和选择版本
**描述**：PuTTY 官网有多个版本（32-bit、64-bit 等），不知选哪个。
**解答**：
- 根据 Windows 系统架构选：
  - 64-bit x86（常见）：`putty-64bit-0.83-installer.msi`
  - 32-bit x86（老系统）：`putty-0.83-installer.msi`
  - 64-bit Arm（少见）：`putty-arm64-0.83-installer.msi`
- 检查系统类型：Win+R → `msinfo32` → 看“系统类型”。
- 推荐：现代 PC 选 64-bit x86 版。

#### 问题：`ifconfig` 输出中哪个是 IP？
**描述**：运行 `ifconfig`，输出多个接口（`wlan0`、`eth0` 等），不知哪个 IP 用。
**解答**：
- 看 `wlan0`（WiFi）或 `eth0`（以太网）的 `inet` 字段。
- 示例输出：
  ```
  wlan0: inet 192.168.209.247  netmask 255.255.255.0
  ```
  IP 是 `192.168.209.247`。
- 忽略 `lo`（`127.0.0.1`）和没 `inet` 的接口。

#### 问题：`Network error: Connection timed out`
**描述**：用 PuTTY 登录时提示连接超时。
**解答**：
- 可能原因：
  1. IP 或端口错：确认 Nano IP 和端口（默认 22）。
  2. SSH 服务未运行：Nano 上跑 `sudo systemctl status ssh`，确认运行。
  3. 网络不通：PC 和 Nano 不在同一网络。
- 解决：
  - 确保 PC 和 Nano 连同一 WiFi。
  - PC 上 `ping Nano_IP` 测试连通性。
  - Nano 上确认 SSH 服务：
    ```
    sudo systemctl start ssh
    ```

#### 问题：`ping` 不通
**描述**：PC 上 `ping 192.168.209.247` 提示 `Request timed out`。
**解答**：
- 可能原因：
  1. 不在同一网络。
  2. Nano 网络断开。
  3. 防火墙或路由器阻拦。
- 解决：
  - 确认 PC 和 Nano 连同一 WiFi（同一网段，如 `192.168.209.x`）。
  - Nano 上 `ping 8.8.8.8`，确认有网。
  - 临时关 PC 防火墙测试：
    ```
    netsh advfirewall set allprofiles state off
    ```
  - 再试 `ping`。

#### 问题 ：防火墙问题？
**描述**：怀疑 Jetson Nano 防火墙挡了连接。
**解答**：
- 检查 `ufw`（常见防火墙工具）：
  ```
  sudo ufw status
  ```
- 发现 `ufw` 未安装（提示 `command not found`）。
- 解决：
  - 安装 `ufw`：
    ```
    sudo apt update
    sudo apt install ufw
    ```
  - 允许 SSH：
    ```
    sudo ufw allow 22
    sudo ufw enable
    ```
  - 或用 `iptables` 检查：
    ```
    sudo iptables -L -v -n
    ```
    允许 22 端口：
    ```
    sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    ```

#### 问题 ：登录名是什么？
**描述**：PuTTY 提示输入登录名，不清楚用哪个。
**解答**：
- 用 Jetson Nano 系统安装时设置的用户名。
- 检查当前用户名：
  ```
  whoami
  ```
- 示例：若输出 `jetson`，PuTTY 登录时输入 `jetson`，再输入密码。

###  Swap 空间相关问题

#### 问题 ：Swap 空间增加步骤看不懂
**描述**：不理解 Swap 增加的命令作用。
**解答**：
- Swap 是用硬盘当“备用内存”，内存不够时用。
- 步骤：
  1. 创建 3GB Swap 文件：
     ```
     sudo fallocate -l 3G /var/swapfile
     ```
  2. 设置权限：
     ```
     sudo chmod 600 /var/swapfile
     ```
  3. 格式化为 Swap：
     ```
     sudo mkswap /var/swapfile
     ```
  4. 启用：
     ```
     sudo swapon /var/swapfile
     ```
  5. 设置开机自动启用：
     ```
     sudo bash -c 'echo "/var/swapfile swap swap defaults 0 0" >> /etc/fstab'
     ```
- 效果：用 `jtop` 查看，Swap 增加到 6GB（原 3GB + 新增 3GB）。

---

## 重要事项与建议

###  网络配置要点
- **同一网络**：PC 和 Jetson Nano 必须在同一局域网（连同一 WiFi），否则需复杂配置（如端口转发）。
- **IP 动态性**：Jetson Nano 的 IP 可能因 DHCP 变，建议设静态 IP：
  ```
  sudo nano /etc/netplan/01-netcfg.yaml
  ```
  示例：
  ```yaml
  network:
    version: 2
    wifis:
      wlan0:
        dhcp4: no
        addresses: [192.168.209.247/24]
        gateway4: 192.168.209.1
        nameservers:
          addresses: [8.8.8.8, 8.8.4.4]
        access-points:
          "your_wifi_ssid":
            password: "your_wifi_password"
  ```
  应用：
  ```
  sudo netplan apply
  ```
- **防火墙**：若有防火墙，确保 SSH 端口（22）开放。

### SSH 配置
- 确保 SSH 服务一直运行：
  ```
  sudo systemctl enable ssh
  sudo systemctl start ssh
  ```
- 若需从公网访问，需路由器端口转发（不建议初学者直接操作，安全风险高）。

### Swap 空间注意事项
- Swap 放硬盘，速度慢，仅应急用。
- 不要设太大，SD 卡寿命有限。
- 检查 Swap 使用：
  ```
  free -h
  ```
- 若程序仍崩溃，考虑优化代码或加物理内存。

### 文件传输
- PuTTY 不直接传文件，推荐用：
  - **PSCP**（PuTTY 套件）：命令行传文件。
    ```
    pscp 文件路径 用户名@Nano_IP:/目标路径
    ```
  - **FileZilla**：图形化 SFTP 工具，输入 Nano IP、用户名、密码、端口 22。
- 确保 SSH 服务开着。

###  学习 Linux 基础
- 常用命令：
  - `ls`：列目录
  - `cd`：换目录
  - `pwd`：当前路径
  - `nano 文件`：编辑文件
  - `sudo`：提权
- 查帮助：`man 命令` 或 `命令 --help`。

---

## 总结
核心问题是网络配置和远程登录。通过确认 IP、开启 SSH、解决网络连通性问题，用 PuTTY 登录。Swap 空间增加缓解了内存不足问题。
