## Docker 部署 Graylog + Opensearch + MongoDB

- 首先拉取官方镜像，然后查看确认镜像是否下载完成
    ```shell
    [root@localhost graylog]# docker images
    REPOSITORY                     TAG       IMAGE ID       CREATED         SIZE
    graylog/graylog                5.2       7f62f560e26c   4 days ago      538MB
    mongo                          latest    2e123a0ccb4b   2 weeks ago     757MB
    opensearchproject/opensearch   latest    c88bc2cd4d18   5 weeks ago     1.22GB
    ```
- 创建 `docker-compose.yml` 文件，以便一步简单启动，内容如下：
    ```yml
    version: '3'
    services:
    mongodb:
        image: mongo:latest
        container_name: mongodb
        ports:
          - "27017:27017"
        volumes:
          # 在主机创建目录，以便永久保存数据。
          - /home/project/mongo/data:/data/db
        networks:
          - graylog-net

    opensearch:
        image: opensearchproject/opensearch:latest
        container_name: opensearch
        ports:
          - "9200:9200"
        volumes:
          # 设置存储目录，以便永久存储
          # 请注意赋予 所有者/组 1000:1000 755 权限，默认是 root:root
          - /home/project/opensearch/data:/usr/share/opensearch/data
        environment:
          - discovery.type=single-node
          # `Xms` specifies the initial memory allocation pool. `Xmx` specifies the maximum allocation pool for the Java Virtual Machine (JVM). "m" and "g" obviously stand for MB and GB.
          - ES_JAVA_OPTS=-Xms512m -Xmx1g
          # 需要关闭以下安全选项，否则无法连接
          - plugins.security.ssl.http.enabled=false
          - plugins.security.disabled=true
        networks:
          - graylog-net

    graylog:
        image: graylog/graylog:5.2
        container_name: graylog
        environment:
          # 配置项需以GRAYLOG_开头，并且是全大写的
          # 密码加密值，不能小于16个字符
          # 运行以下命令获取：< /dev/urandom tr -dc A-Z-a-z-0-9 | head -c${1:-96};echo;
          - GRAYLOG_PASSWORD_SECRET=输入你的
          - GRAYLOG_HTTP_EXTERNAL_URI=http://0.0.0.0:9000/
          - GRAYLOG_WEB_ENDPOINT_URI=http://0.0.0.0:9000/api
          # 运行以下命令获取：echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
          - GRAYLOG_ROOT_PASSWORD_SHA2=输入你的
          - GRAYLOG_REST_LISTEN_URI=http://0.0.0.0:12900/
          # 如果设置了特殊字符的密码，请使用 URL 编码，比如 @ 换成 %40
          - GRAYLOG_MONGODB_URI=mongodb://mongodb:27017/graylog
          # 虽然使用 opensearch，但设置项名称还是 elasticsearch 的
          - GRAYLOG_ELASTICSEARCH_HOSTS=http://opensearch:9200
          #- GRAYLOG_ELASTICSEARCH_USERNAME=admin
          #- GRAYLOG_ELASTICSEARCH_PASSWORD=admin
          # 配置时区
          - GRAYLOG_ROOT_TIMEZONE=Asia/Shanghai
        ports:
          - "9000:9000"
          - "5044:5044"
          - "12900:12900"
          - "12201:12201/udp"
        networks:
          - graylog-net
        depends_on:
          - mongodb
          - opensearch

    networks:
      graylog-net:
        driver: bridge
    ```
- Graylog 启动：转到保存 `docker-compose.yml` 文件的目录内，运行 `docker-compose up -d`，就可以正常启动，然后登录 web 页面进行设置操作；
