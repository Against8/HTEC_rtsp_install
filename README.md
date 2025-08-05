### **步骤 1: 安装依赖项**

首先，更新软件包列表并安装 `nano` 和 `ffmpeg` 。

```bash
sudo apt update
sudo apt install nano ffmpeg
```

### **步骤 2: 下载 MediaMTX**

从 MediaMTX 的 GitHub Releases 页面下载适用于 ARM64 架构的最新版本。也可以使用 `wget` 直接在设备上下载。


> 建议先访问 [MediaMTX Releases](https://github.com/bluenviron/mediamtx/releases) 页面，确认最新的版本号，然后替换下面命令中的版本号 (`v1.13.1`)。

```bash
wget https://github.com/bluenviron/mediamtx/releases/download/v1.13.1/mediamtx_v1.13.1_linux_arm64.tar.gz
```

### **步骤 3: 解压归档文件**

使用 `tar` 命令解压刚刚下载的压缩包。

```bash
tar -xvf mediamtx_v1.13.1_linux_arm64.tar.gz
```

### **步骤 4: 部署二进制文件和配置文件**

将可执行文件移动到 `/usr/local/bin` (使其成为系统范围内的命令)，并将默认配置文件移动到 `/usr/local/etc`。

```bash
sudo mv mediamtx /usr/local/bin/
sudo mv mediamtx.yml /usr/local/etc/
```

### **步骤 5: 创建 `systemd` 服务**

为了让 MediaMTX 作为后台服务运行并实现开机自启，需要创建一个 `systemd` 服务单元文件。

1.  使用 `nano` 创建服务文件：
    ```bash
    sudo nano /etc/systemd/system/mediamtx.service
    ```

2.  将以下内容复制并粘贴到编辑器中：

    ```ini
    [Unit]
    Description=MediaMTX RTSP/RTMP/HLS Server
    After=network.target

    [Service]
    User=jetson
    ExecStart=/usr/local/bin/mediamtx /usr/local/etc/mediamtx.yml
    Restart=always
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    ```

3.  按 `Ctrl+X`，然后按 `Y`，最后按 `Enter` 保存并退出 `nano`。

### **步骤 6: 管理 `systemd` 服务**

创建服务文件后，需要重载 `systemd` 配置，然后启用并启动我们的新服务。

```bash
# 重新加载 systemd 管理器配置
sudo systemctl daemon-reload

# 设置服务开机自启
sudo systemctl enable mediamtx.service

# 立即启动服务
sudo systemctl start mediamtx.service
```

### **步骤 7: 验证服务状态**

最后，检查服务是否已成功运行。

```bash
sudo systemctl status mediamtx.service
```

如果一切正常，你应该会看到绿色的 `active (running)` 状态，如下所示：

```
● mediamtx.service - MediaMTX RTSP/RTMP/HLS Server
     Loaded: loaded (/etc/systemd/system/mediamtx.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-10-24 14:30:00 UTC; 10s ago
   Main PID: 12345 (mediamtx)
      Tasks: 8 (limit: 4135)
     Memory: 10.5M
     CGroup: /system.slice/mediamtx.service
             └─12345 /usr/local/bin/mediamtx /usr/local/etc/mediamtx.yml
```

