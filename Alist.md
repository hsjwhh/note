### 群晖 Alist 安装、配置、webDav 同步

- Docker 安装
    * 搜索 alist，下载并配置；
    ![docker 安装 alist](Resources/alist_docker.png)
    * 配置 alist，配置项大部分默认，网络这边选择 HOST，存储空间这边添加一个映射；
    ![docker 配置](Resources/alist_docker_config.png)
    * 启动 alist，等待启动完成后，请查看容器运行的 log，以便得到管理员密码；
    ![docker init log](Resources/alist_init_log.png)

未完 待续