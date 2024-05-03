## Docker 部署 MySQL

- 拉取官方镜像，按照自己需求确认版本
  ```sh
  docker pull mysql
  ```
- 注意将容器的存储空间映射到本机目录，以便保存数据
  ```sh
  .data:/var/lib/mysql
  .conf:/etc/mysql
  .log:/var/log/mysql
  ```
- 同时在环境变量中设置数据库密码
  ```sh
  MYSQL_ROOT_PASSWORD=你自己的密码
  ```
- 启动
