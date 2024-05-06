## Docker 部署 MySQL

- 拉取官方镜像，按照自己需求确认版本（这边拉取最新版，8.0.36）
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
- 将 my.cnf 文件放到 conf 目录下的 conf.d 目录下
- 启动
- 进入容器更改 root 密码
  ```sh
  sh-4.4# mysql -u root -p
  Enter password: 
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 9
  Server version: 8.4.0 MySQL Community Server - GPL
  
  Copyright (c) 2000, 2024, Oracle and/or its affiliates.
  
  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.
  
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  
  mysql> 
  ```
  切换数据库`use mysql`
  更改密码
  ```sh
  ALTER USER 'root'@'%' IDENTIFIED BY '你自己的密码' PASSWORD EXPIRE NEVER;
  ALTER USER 'root'@'localhost' IDENTIFIED BY '你自己的密码';
  ```