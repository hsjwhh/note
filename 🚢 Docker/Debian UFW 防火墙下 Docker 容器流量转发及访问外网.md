Debian12 安装 UFW 后安装 docker，在 `/etc/docker/daemon.json`文件内设置`"iptables":false`，以便关闭 docker 创建自己的规则，防止跳过 UFW 的限制。但同时这样操作以后，容器之间就无法互相访问，导致容器 ngixn 无法访问 wordpress 内的 php fpm，网站无法正常运行。同时，容器无法访问外网，wordpress无法访问插件、主题等网站内容，无法更新。可通过以下操作解决此类问题

## 一. 容器间流量转发以便互相访问

```bash
sudo iptables -A FORWARD -s 172.21.1.0/24 -j ACCEPT
```

### 命令分解

- `sudo`：以超级用户（root）权限运行命令。这是因为修改 `iptables` 需要管理员权限。
- `iptables`：用于配置 Linux 内核防火墙的命令行工具。
- `-A FORWARD`：指定将一条新规则添加到 `FORWARD` 链。
- `-s 172.21.1.0/24`：指定源地址是 `172.21.1.0/24`，即所有来自 `172.21.1.0` 到 `172.21.1.255` 的流量。这里 `/24` 表示子网掩码为 255.255.255.0。
- `-j ACCEPT`：指定匹配到该规则的流量应被接受。

### 详细解释

1. **sudo**：
   - `sudo` 命令允许普通用户以超级用户权限执行命令。因为修改 `iptables` 需要系统管理员权限，所以必须以 `sudo` 来运行。

2. **iptables**：
   - `iptables` 是一个用户空间命令行工具，用于设置、维护和检查 Linux 内核的 `netfilter` 防火墙规则。

3. **-A FORWARD**：
   - `-A` 表示添加（Append）一条新规则到指定链。链是 `iptables` 的基础，每条链包含一组规则。
   - `FORWARD` 链用于处理那些转发的网络数据包，这些数据包既不是本地生成的也不是本地接收的，而是从一个网络接口到另一个网络接口。

4. **-s 172.21.1.0/24**：
   - `-s` 参数指定数据包的源地址（Source Address）。
   - `172.21.1.0/24` 是一个子网，表示源地址可以是 `172.21.1.0` 到 `172.21.1.255`。这里 `/24` 是子网掩码，等同于 255.255.255.0，表示前 24 位是网络部分，后 8 位是主机部分。

5. **-j ACCEPT**：
   - `-j` 参数指定匹配到该规则的数据包将跳转（Jump）到指定的目标动作（Target Action）。
   - `ACCEPT` 表示允许数据包通过（Accept the packet）。也就是说，所有匹配该规则的数据包将被允许继续转发到目的地。

### 整体意义

这条 `iptables` 规则的整体作用是：
- **允许所有源地址是 `172.21.1.0/24` 的数据包通过 FORWARD 链**。也就是说，凡是从 `172.21.1.0` 到 `172.21.1.255` 这个子网来的数据包都将被允许继续转发，而不会被防火墙阻止。

这条命令的意义是允许 Docker 容器子网 `172.21.1.0/24` 内的所有容器之间的流量转发。这对于运行多个容器并且它们需要相互通信的环境非常重要。通过这条规则，容器之间的网络通信不会被防火墙阻止，从而保证应用程序的正常运行。

要将 `iptables` 规则添加到 `ufw` 中，可以通过修改 `ufw` 的配置文件 `after.rules` 来实现。以下是具体步骤：

### 修改 `ufw` 的 `after.rules`

1. **打开 `before.rules` 文件**：
   使用你喜欢的文本编辑器来编辑 `before.rules` 文件。这个文件通常位于 `/etc/ufw/` 目录下。

   ```bash
   sudo nano /etc/ufw/before.rules
   ```

2. **在文件中添加 `iptables` 规则**：
   在文件中适当的位置添加以下内容。通常建议将其放在文件末尾的 `COMMIT` 之前。

   ```bash
   # BEGIN CUSTOM RULES FOR DOCKER
   *filter
   # 直接添加
   -A FORWARD -s 172.21.1.0/24 -j ACCEPT
   COMMIT
   # END CUSTOM RULES FOR DOCKER
   ```

   这段代码的意义：
   - `*filter`：定义了一个 `filter` 表。
   - `-A FORWARD -s 172.21.1.0/24 -j ACCEPT`：添加了一条规则，允许 Docker 容器子网 `172.21.1.0/24` 内的所有容器之间的流量转发。
   - `COMMIT`：提交这些规则。

3. **保存并关闭文件**：
   在 `nano` 编辑器中，按 `Ctrl+X` 保存退出编辑器。

### 重新加载 `ufw`

为了使新添加的规则生效，需要重新加载 `ufw`。

```bash
sudo ufw reload
```

### 验证规则

可以通过以下命令检查 `iptables` 规则，确认新规则是否生效：

```bash
sudo iptables -L FORWARD -v -n
```

### 注意事项

- **防火墙安全性**：确保其他的 `ufw` 规则依然生效，防止不必要的端口暴露给外网。
- **规则顺序**：在 `ufw` 中规则的顺序很重要，确保你的自定义规则不会被其他规则覆盖或冲突。
- **备份配置文件**：在修改任何配置文件之前，建议备份原始文件，以便在出现问题时能够恢复。

通过上述步骤，你可以将 `iptables` 规则集成到 `ufw` 中，使其在系统重启后依然有效，确保 Docker 容器之间的流量转发正常工作。


## 二. 容器访问外网

```bash
sudo iptables -t nat -A POSTROUTING -s 172.21.1.0/24 -o ens33 -j MASQUERADE
```

### 命令详细解释

- `sudo`：以超级用户权限执行命令。

- `iptables`：防火墙管理工具。

- `-t nat`：指定要操作的表，这里是 `nat` 表。`nat` 表主要用于网络地址转换（NAT）操作。

- `-A POSTROUTING`：将规则添加到 `POSTROUTING` 链中。`POSTROUTING` 链用于在数据包即将离开主机时进行处理。

- `-s 172.21.1.0/24`：指定源地址范围，这里是 Docker 容器的 IP 地址段 `172.21.1.0/24`。

- `-o eth0`：指定输出网络接口，这里是主机的外部网络接口 `eth0`。意味着只有当数据包从这个接口发送时，规则才会应用。

- `-j MASQUERADE`：指定目标操作为伪装（MASQUERADE）。MASQUERADE 是一种动态的 SNAT，它会将数据包的源 IP 地址更改为主机的外部 IP 地址。

### 执行此命令的作用

1. **源地址伪装**：将来自 Docker 容器网络（172.21.1.0/24）的数据包的源地址更改为主机的外部 IP 地址。这样做的好处是外部网络会认为数据包是来自主机，而不是来自容器网络。

2. **使容器能访问外部网络**：通过伪装，容器内的应用程序可以访问外部网络资源，例如互联网，同时保持对外部网络的透明性。

3. **避免网络冲突**：如果你的容器网络使用了私有 IP 地址段，而这些地址在外部网络中可能会引起冲突，伪装可以帮助避免这种情况。

要将 `iptables` 规则添加到 `ufw` 中，可以通过修改 `ufw` 的配置文件 `after.rules` 来实现。以下是具体步骤：

### 修改 `ufw` 的 `after.rules`

1. **编辑 `/etc/ufw/before.rules` 文件**：

   ```bash
   sudo nano /etc/ufw/before.rules
   ```

2. **在文件中添加规则**：

   在文件的适当位置添加以下内容（通常在 `*nat` 部分），可直接放末尾如果文件内没有*nat：

   ```plaintext
   # rules.before
   #
   # Rules that should be run before the ufw command line added rules. Custom
   # rules should be added to this file to ensure they are maintained.
   #
   # Do not remove the 'COMMIT' line or these rules won't be processed.
   #
   *nat
   :POSTROUTING ACCEPT [0:0]
   # Allow Docker containers to communicate with external networks
   -A POSTROUTING -s 172.21.1.0/24 -o eth0 -j MASQUERADE
   COMMIT
   ```

3. **保存并退出**：

   按 `Ctrl + X`，然后按 `Y` 保存并退出。

4. **重新加载 `ufw`**：

   ```bash
   sudo ufw reload
   ```

### 总结

通过设置源地址伪装规则，你可以确保来自 Docker 容器的流量在通过主机的外部网络接口时被伪装为主机的外部 IP 地址，从而允许容器访问外部网络资源。将该规则添加到 `ufw` 的配置文件中，可以确保规则在系统重启后仍然有效。