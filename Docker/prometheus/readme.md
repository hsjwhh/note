### 项目结构

你可以参考以下文件结构：

```
.
├── prometheus-config
│   ├── prometheus.yml        # 配置主文件
│   └── web-config.yml        # 网页验证配置文件
├── prometheus-data           # prometheus 数据永久化存储
├── grafana-data              # grafana    数据永久化存储
└── docker-compose.yml        # Docker Compose 配置文件
```

### 注意事项

- `data` 文件夹请更新用户及组，否则启动容器会报错，提示权限不足，无法创建文件，导致无法启动。
  ```shell
  sudo chown -R nobody:nogroup data
  ```
  **报错信息：**
  ```log
  caller=query_logger.go:114 level=error component=activeQueryTracker msg="Error opening query log file" file=/prometheus/queries.active err="open /prometheus/queries.active: permission denied"
  panic: Unable to create mmap-ed active query log
  ```
- `web-config.yml` 存储加密了的用户名和密码，以便提供一个简单的登录验证。格式见示例文件，具体说明请见官方说明文档[^1]及生成方法[^2]，可在线生成[^3]。
  ```yml
  # TLS and basic authentication configuration example.
  #
  # Additionally, a certificate and a key file are needed.
  tls_server_config:
    cert_file: server.crt
    key_file: server.key

  # Usernames and passwords required to connect to Prometheus.
  # Passwords are hashed with bcrypt: https://github.com/prometheus/exporter-toolkit/blob/master/docs/web-configuration.md#about-bcrypt
  basic_auth_users:
    # 用户1 alice
    alice: $2y$10$mDwo.lAisC94iLAyP81MCesa29IzH37oigHC/42V2pdJlUprsJPze
    # 用户2 bob
    bob: $2y$10$hLqFl9jSjoAAy95Z/zw8Ye8wkdMBM8c5Bn1ptYqP/AXyV0.oy0S8m
  ```
- 添加 `node-exporter` 收集数据，建议开启只读等模式，确保安全。
- 添加 `grafana` 查看数据，请注意修改挂在存储文件的用户和组
  ```shell
  # 在 grafana 容器内，改文件夹属于 grafana:root
  # 通常在容器内 grafana 用户的 UID 通常为 472。
  sudo chown -R 472:root grafana-data
  ```


[^1]:https://prometheus.io/docs/prometheus/latest/configuration/https/
[^2]:https://prometheus.io/docs/guides/basic-auth/#securing-prometheus-api-and-ui-endpoints-using-basic-auth
[^3]:https://bcrypt-generator.com