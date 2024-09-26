> 转 https://www.orchome.com/17176

### 一. 配置Containerd配置拉取镜像加速

01. 修改config.toml

    ```bash
    sudo nano /etc/containerd/config.toml
    ```

    搜索 `config_path`，值设置为目录 `/etc/containerd/certs.d`:

    ```toml
    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = "/etc/containerd/certs.d"

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""
    ```

02. 创建目录
    ```bash
    sudo mkdir -p /etc/containerd/certs.d
    ```

03. 配置 dockerhub 加速
    
    - 创建目录
      ```bash
      sudo mkdir -p /etc/containerd/certs.d/docker.io
      ```
    
    - 在目录下创建文件 `hosts.toml`
      
      ```toml
      server = "https://docker.io"

      [host."https://dockerproxy.com"]
        capabilities = ["pull", "resolve"]
      ```

04. 配置私有镜像 test
    
    - 新建目录，以实际配置为准。例镜像地址为 `test.exsample.com`，就创建这个域名的目录
      
      ```bash
      sudo mkdir -p /etc/containerd/certs.d/test.exsample.com
      ```

    - 在目录下创建文件 `hosts.toml`
    
      ```toml
      server = "https://test.exsample.com"

      [host."https://test.exsample.com"]
        capabilities = ["pull", "resolve"]
        skip_verify = true  # 跳过证书验证
      ```

05. 重启 containerd
    
    ```bash
    sudo systemctl restart containerd
    ```

### 二. 拉取验证

01. crictl 命令验证
    
    ```bash
    sudo crictl pull docker.io/library/alpine:3.18
    ```

02. ctr 命令验证
    配置文件、目录结构与 crictl 一致，但是 ctr 命令未生效。
    解决方法 ：ctr 命令 pull 镜像添加 --hosts-dir 可以实现到拉取镜像加速。

    ```bash
    sudo ctr --debug image pull docker.io/library/alpine:3.18 --hosts-dir /etc/containerd/certs.d
    ```

03. nerdctl命令配置
    配置文件、目录结构与 crictl 一致。
    注意：配置路径 只能 是 /etc/containerd/certs.d 目录下

### 三. 更多

01. docker.io 示例
    
    ```bash
    sudo mkdir -p /etc/containerd/certs.d/docker.io

    cat <<'EOF' | sudo tee /etc/containerd/certs.d/docker.io/hosts.toml > /dev/null
    server = "https://docker.io"
    [host."https://dockerproxy.com"]
      capabilities = ["pull", "resolve"]

    [host."https://docker.m.daocloud.io"]
      capabilities = ["pull", "resolve"]

    [host."https://hub-mirror.c.163.com"]
      capabilities = ["pull", "resolve"]
    EOF
    ```

02. registry.k8s.io 示例
    
    ```bash
    sudo mkdir -p /etc/containerd/certs.d/registry.k8s.io

    cat <<'EOF' | sudo tee /etc/containerd/certs.d/registry.k8s.io/hosts.toml > /dev/null
    server = "https://registry.k8s.io"
    [host."https://k8s.m.daocloud.io"]
      capabilities = ["pull", "resolve"]
    EOF
    ```

03. k8s.gcr.io 示例
    
    ```bash
    sudo mkdir -p /etc/containerd/certs.d/k8s.gcr.io

    cat <<'EOF' | sudo tee /etc/containerd/certs.d/k8s.gcr.io/hosts.toml > /dev/null
    server = "https://k8s.gcr.io"
    [host."k8s-gcr.m.daocloud.io"]
      capabilities = ["pull", "resolve"]
    EOF
    ```