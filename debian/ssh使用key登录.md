Debian 12 服务器上为普通用户配置 SSH 公钥认证。请按照步骤操作。

### 步骤 1：在本地生成 SSH 密钥对

1. 打开终端，在本地计算机上运行以下命令生成 SSH 密钥对：

   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

2. 按提示输入文件路径（默认是 `~/.ssh/id_rsa`）和密码（如果需要的话，可以留空）。

### 步骤 2：将公钥复制到服务器

1. 使用 `ssh-copy-id` 将公钥复制到你的服务器：

   ```bash
   ssh-copy-id username@your_server_ip
   ```

   将 `username` 替换为你要使用的普通用户的用户名，将 `your_server_ip` 替换为你的服务器 IP 地址。

2. 如果 `ssh-copy-id` 命令不可用，你可以手动复制公钥：

   ```bash
   cat ~/.ssh/id_rsa.pub | ssh username@your_server_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
   ```

### 步骤 3：配置服务器的 SSH 服务

1. 登录到你的服务器：

   ```bash
   ssh username@your_server_ip
   ```

2. 编辑 SSH 配置文件 `/etc/ssh/sshd_config`：

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

3. 确保以下选项配置正确：

   ```plaintext
   PubkeyAuthentication yes
   PasswordAuthentication no
   ```

4. 重启 SSH 服务以使更改生效：

   ```bash
   sudo systemctl restart ssh
   ```

### 步骤 4：测试连接

1. 从本地计算机尝试使用私钥文件连接到服务器：

   ```bash
   ssh username@your_server_ip
   ```

如果一切配置正确，你应该能够无密码登录到你的服务器。

### 附加步骤：禁用 `root` 用户的 SSH 登录（可选）

为了增强安全性，你可以禁用 `root` 用户的 SSH 登录：

1. 编辑 SSH 配置文件 `/etc/ssh/sshd_config`：

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. 确保以下选项配置正确：

   ```plaintext
   PermitRootLogin no
   ```

3. 重启 SSH 服务：

   ```bash
   sudo systemctl restart ssh
   ```

这样可以防止任何人直接以 `root` 用户身份登录你的服务器，只能通过普通用户登录后使用 `sudo` 提升权限。

通过以上步骤，你可以安全地使用证书登录 SSH，并确保服务器的安全性。



在使用 PuTTYNG 登录时遇到“unable to use key file”错误，通常是因为 PuTTY 使用的密钥格式与 OpenSSH 的密钥格式不同。你需要将 OpenSSH 生成的私钥文件转换为 PuTTY 能够识别的格式 (`.ppk`)。

以下是详细步骤：

### 步骤 1：下载并安装 PuTTYgen

如果你尚未安装 PuTTYgen，可以从 PuTTY 官方网站下载并安装：[PuTTY Download Page](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)。

### 步骤 2：使用 PuTTYgen 转换私钥文件

1. 打开 PuTTYgen。
2. 点击 `Load` 按钮，选择 OpenSSH 格式的私钥文件（例如 `id_rsa`）。
   - 你可能需要将文件类型过滤器从 `.ppk` 更改为 `All Files (*.*)`，这样才能看到并选择 `id_rsa` 文件。
3. 加载成功后，点击 `Save private key` 按钮，将私钥文件保存为 PuTTY 格式（`.ppk`）。
   - 你可以选择是否为私钥设置密码。

### 步骤 3：在 PuTTYNG 中配置私钥文件

1. 打开 PuTTYNG。
2. 在左侧的树状结构中选择 `Connection` -> `SSH` -> `Auth`。
3. 在 `Private key file for authentication` 字段中，浏览并选择你刚才转换并保存的 `.ppk` 文件。
4. 返回到 `Session` 菜单，在 `Host Name (or IP address)` 中输入服务器的地址。
5. 确保连接类型选择为 `SSH`，然后点击 `Open` 连接。

### 步骤 4：测试连接

尝试使用 PuTTYNG 连接到服务器，确保使用转换后的 `.ppk` 私钥文件登录。

### 总结

通过将 OpenSSH 格式的私钥文件转换为 PuTTY 格式，并在 PuTTYNG 中正确配置，你应该能够成功使用证书登录 SSH。