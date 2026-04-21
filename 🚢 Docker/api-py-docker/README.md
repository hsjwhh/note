# **构建 docker image**

## 简介
**将之前测试用项目 python + flask + uwsgi + nginx 打包成 docker images，并启用**

## 目录结构
```
.
├── app
│   ├── static                  # 存储 json 文件
│   │   └── xxxxx.json
│   ├── uwsgi
│   │   └── uwsgi.ini           # uwsgi 配置文件
│   ├── main.py                 # 主程序文件
│   └── requirements.txt        # 程序依赖
├── Dockerfile                  # dockerfile
└── docker-compose.yml          # Docker Compose 配置文件
```

### 构建注意事项

- 记得网络使用 host 否则打包时容器无法联网，也就无法安装依赖包等
  ```bash
  docker build --pull --rm --network=host  -t api-py:0.1 .
  ```

  1. **`docker build`**：
     - 这个命令用于构建Docker镜像。
  
  2. **`--pull`**：
     - 这个标志告诉Docker在构建镜像之前从Docker Hub拉取最新版本的基础镜像。如果不使用这个标志，Docker将使用本地缓存的基础镜像，这可能不是最新的。
  
  3. **`--rm`**：
     - 这个标志告诉Docker在构建过程中删除临时容器。如果不使用这个标志，Docker将在构建失败后保留中间容器，这样可能会浪费磁盘空间。
  
  4. **`--network=host`**:
     - 这个标志告诉Docker在构建过程中使用host网络模式，以便容器可以正常联网。

  5. **`-f "dockerfiles\Dockerfile-for-frps"`**：
     - `-f`参数指定要使用的Dockerfile的路径。默认情况下，Docker会在当前目录中查找名为`Dockerfile`的文件，但你可以通过这个参数指定不同路径和文件名。在这个例子中，Dockerfile位于`dockerfiles`目录下，文件名为`Dockerfile-for-frps`。
  
  6. **`-t api-py:0.1`**：
     - `-t`参数用于标记镜像。这里的镜像名是`api-py`，标签是`0.1`。标签是可选的，但推荐使用标签来标识不同版本的镜像。如果没有指定标签，默认标签是`latest`。
  
  7. **`"。"`**：
     - 最后的参数是上下文路径。Docker将在这个目录中查找构建镜像所需的文件。在这个例子中，上下文路径是`当前`目录，这意味着所有构建所需的文件都应该在这个目录中。

- 安装后，记得清理无用文件， 否则镜像体积会很大