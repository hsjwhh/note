- WSL2 模式下 docker-desktop-data vm 磁盘映像通常位于以下位置：
  ```shell
  C:\Users\用户名\AppData\Local\Docker\wsl\data\ext4.vhdx
  ```
**按照以下说明将其重新定位到其他目录，并保留所有现有的Docker数据**
1. 首先，右键单击 Docker Desktop 图标关闭 Docker 桌面，然后选择退出 Docker 桌面，然后，打开命令提示符：
    ```shell
    wsl --list -v
    ```

2. 您应该能够看到，确保两个状态都已停止
    ```shell
    # 默认情况下，Docker Desktop for Window会创建如下两个发行版（distro)
    C:\Users\用户名\AppData\Local\Docker\wsl

    docker-desktop (对应distro/ext4.vhdx)
    docker-desktop-data （对应data/ext4.vhdx）
    ```

3. 将 docker-desktop-data 导出到文件中(备份 image 及相关文件)，使用如下命令：
    ```shell
    wsl --export docker-desktop-data "D:\\docker-desktop-data.tar"
    ```

4. wsl取消注册docker-desktop-data，请注意
    ```shell
    # C:\Users\用户名\AppData\Local\Docker\wsl\data\ext4.vhdx 文件将被自动删除    
    wsl --unregister docker-desktop-data
    ```
5. 将导出的 docker-desktop-data 再导入回 wsl，并设置路径，即新的镜像及各种 docker 使用的文件的挂载目录，我这里设置到 E:\Docker-wsl\wsl
    ```shell
    wsl --import docker-desktop-data "E:\Docker-wsl\wsl" "D:\\docker-desktop-data.tar" --version 2
    ```
6. 命令执行完毕，就能在目录下看到文件了，这时次启动 Docker Desktop，一切正常