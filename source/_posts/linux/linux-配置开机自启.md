---
title: linux-配置开机自启
date: 2025-02-26 20:29:53
categories: linux
tag: 开机自启
---
### 方法一： 使用 `crontab` 设置开机自动启动

1. **编辑当前用户的 `crontab` 文件**  
   运行以下命令编辑 `crontab`：

   ```bash
   crontab -e
   ```

2. **添加开机启动任务**  
   在 `crontab` 文件的最后一行添加以下内容，将启动日志输出到log.txt中：

   ```bash
   @reboot /usr/bin/java -jar /opt/mp/mp-web-1.0.0.jar >> /opt/mp/log.txt 2>&1 
   ```

3. **验证是否设置成功**  
   你可以使用以下命令查看已设置的 `crontab` 任务：

   ```bash
   crontab -l
   ```


### 方法二：使用 `systemd`

`systemd` 是现代 Linux 系统用来启动和管理服务的工具，通常在大多数发行版中都默认安装。

1. **创建一个 systemd 服务文件**  
   在 `/etc/systemd/system/` 目录下创建一个新的 `.service` 文件，例如 `openvpn-client.service`。

   ```bash
   sudo vi /etc/systemd/system/openvpn-client.service
   ```

2. **编辑服务文件**  
   在文件中加入以下内容：

   ```ini
   [Unit]
   Description=OpenVPN client service
   After=network.target
   
   [Service]
   Type=simple
   ExecStart= /usr/sbin/openvpn --config /opt/openvpn/xxx.ovpn
   Restart=always
   
   [Install]
   WantedBy=multi-user.target
   ```
3. **重新加载 `systemd` 配置**  

   ```bash
   sudo systemctl daemon-reload
   ```

4. **启动服务**  
   启动服务并验证是否正常运行：

   ```bash
   sudo systemctl start openvpn-client.service
   ```

5. **设置开机自启**  
   启动后，设置服务开机自动启动：

   ```bash
   sudo systemctl enable openvpn-client.service
   ```

6. **查看服务状态**  
   查看服务是否正常运行：

   ```bash
   sudo systemctl status openvpn-client.service
   ```

### 方法三：使用 `rc.local`（适用于较老的 Linux 版本）

`rc.local` 是一个比较传统的方式，在一些较老的 Linux 系统中使用。

1. **编辑 `rc.local` 文件**  
   打开 `/etc/rc.local` 文件，如果文件不存在，可能需要手动创建：

   ```bash
   sudo vi /etc/rc.local
   ```

2. **在文件末尾添加启动命令**  
   在文件末尾添加启动 Java 应用的命令，确保添加在 `exit 0` 之前：

   ```bash
   /usr/bin/java -jar /path/to/your/project/your-app.jar &
   ```

   使用 `&` 将应用程序放入后台运行。

3. **确保 `rc.local` 可执行**  
   如果 `rc.local` 文件没有可执行权限，需要修改：

   ```bash
   sudo chmod +x /etc/rc.local
   ```

4. **重启系统**  
   重启系统后，Java 应用将会自动启动。
