## Docker 部署 ossec-server 及输出 Graylog 记录查看日志

>Linux 上使用 Docker 部署 ossec-server，Windows server 上安装 ossec-agent，然后配合之前部署的 Graylog 存储查看日志
>ossec 官网下载需要的软件 https://www.ossec.net/download-ossec/

### 下载并启动 ossec-server
- 先下载最新 ossec 镜像；
  ```shell
  docker pull atomicorp/ossec-docker
  ```
- 创建 ossec-server 数据存储目录，以便永久存储 ossec-server 配置；
  ```shell
  mkdir /home/project/ossec/data
  ```
- 运行 ossec-server
  ```shell
  docker run -d -p 1514:1514/udp -p 1515:1515/tcp -v /home/project/ossec/data:/var/ossec/data --name ossec-server atomicorp/ossec-docker
  ```
- 可以通过 logs 命令查看容器运行情况
  ```shell
  [root@localhost ~]# docker logs ossec-server
  Starting ossec-authd...
  Starting OSSEC HIDS 3.6.0...
  Started ossec-maild...
  Started ossec-execd...
  Started ossec-analysisd...
  2024/01/16 14:14:26 ossec-logcollector(1905): INFO: No file configured to monitor.
  Started ossec-logcollector...
  Started ossec-remoted...
  Started ossec-syscheckd...
  Started ossec-monitord...
  Completed.
  2024/01/16 14:14:26 ossec-analysisd: INFO: Ignoring file: '/var/ossec/active-response/ossec-hids-responses.log'
  2024/01/16 14:14:26 ossec-analysisd: INFO: Started (pid: 56).
  2024/01/16 14:14:26 ossec-analysisd: logstat: Unable to create stat queue: /stats/weekly-average
  2024/01/16 14:14:27 ossec-monitord: INFO: Started (pid: 72).
  2024/01/16 14:14:27 ossec-remoted(4111): INFO: Maximum number of agents allowed: '16384'.
  2024/01/16 14:14:27 ossec-remoted(1410): INFO: Reading authentication keys file.
  2024/01/16 14:14:27 ossec-remoted: INFO: No previous counter available for 'DEFAULT_LOCAL_AGENT'.
  2024/01/16 14:14:27 ossec-remoted: INFO: Assigning counter for agent DEFAULT_LOCAL_AGENT: '0:0'.
  2024/01/16 14:14:27 ossec-remoted: INFO: Assigning counter for agent WindowsServer2008: '0:2888'.
  2024/01/16 14:14:27 ossec-remoted: INFO: Assigning sender counter: 0:1288
  2024/01/16 14:14:31 ossec-syscheckd: INFO: Started (pid: 69).
  2024/01/16 14:14:31 ossec-rootcheck: INFO: Started (pid: 69).
  ```

### 进入容器配置 agent 参数
- 正常运行 ossec-server 后，使用以下命令进入容器内；
  ```shell
  docker exec -it ossec-server bash
  ```
- 运行以下命令配置 agent；
  ```shell
  [root@140cd0adce99 /]# /var/ossec/bin/manage_agents


  ****************************************
  * OSSEC HIDS v3.6.0 Agent manager.     *
  * The following options are available: *
  ****************************************
    (A)dd an agent (A).
    (E)xtract key for an agent (E).
    (L)ist already added agents (L).
    (R)emove an agent (R).
    (Q)uit.
  Choose your action: A,E,L,R or Q: a # 增加一个 agent

  - Adding a new agent (use '\q' to return to the main menu).
    Please provide the following:
    * A name for the new agent: # 为你的 agent 命名
    * The IP Address of the new agent: # 输入你安装 agent 的主机 ip
    * An ID for the new agent[002]: # agent id，默认就好
  Agent information:
    ID:002
    Name:XXXXXXXXXXXXXX
    IP Address:XXX.XXX.XXX.XXX

  Confirm adding it?(y/n): y # y 确认以上信息
  Agent added with ID 002.


  ****************************************
  * OSSEC HIDS v3.6.0 Agent manager.     *
  * The following options are available: *
  ****************************************
     (A)dd an agent (A).
     (E)xtract key for an agent (E).
     (L)ist already added agents (L).
     (R)emove an agent (R).
     (Q)uit.
  Choose your action: A,E,L,R or Q: e # 创建 agent 需要的 key

  Available agents: 
     ID: 001, Name: DEFAULT_LOCAL_AGENT, IP: 127.0.0.1
     ID: 002, Name: XXXXXXXXXXXXXX, IP: XXX.XXX.XXX.XXX
  Provide the ID of the agent to extract the key (or '\q' to quit): 002 # 输入 agent id

  Agent key information for '002' is: # 复制下面的 key，稍后 Windows 主机上的 agent 需要用到
  MDAzIFdpbmRvd3NTZXJ2ZXJSMiAx9TIuMTY4LjEuMTEgMTM1NzFkMzg0YzlkNDcwMjNmODFhMjFhYzlkNzJhZGM0MjJmOGFkZjliNWNmNGRiOWU1ZDE2NjZhNWMcYzZmOA==

  ** Press ENTER to return to the main menu.
  ```

### 安装 ossec-agent-win32
- Windows 系统中安装之前官网下载的 ossec-agent-win32 程序，输入安装了 ossec-server 的服务器 ip，以及之前配置的 agent key，选择【Save】，菜单【Manage】选择 start 或者 restart，刷新下状态，正常应该如下图显示,agent 正常运行；
  ![ossec-agent](https://myimg.imdouba.com/?/com/uploads/2024/01/ossec-agent-win32.png){width="500px"}

### 配置 ossec-server 输出至 Graylog
>使用 Graylog 提供的配置方法，https://github.com/Graylog2/graylog-guide-ossec

- 进入 Graylog web控制界面，选择菜单`System -> Inputs`，创建一个新的 **CEF UDP** input，端口自取，名称自取。建议设置时区，以保证日志显示时间是本地时间。
- 进入 ossec-server 容器内，编辑 `/var/ossec/erc/ossec.conf`，添加一下配置；
  ```yml
          <syslog_output>
            <server>Graylog-server-ip</server>
            <port>端口自取</port>
            <format>cef</format>
          </syslog_output>
  ```
- 开启 OSSEC syslog
  ```shell
  /var/ossec/bin/ossec-control enable client-syslog
  ```
- 重启 ossec-server
  ```shell
  /var/ossec/bin/ossec-control enable client-syslog
  ```
- 退出容器，查看 log，正常应该看到 提示已开启 syslog 并且连接到了你设置的 graylog-server-ip
- 现在就可以在 Graylog 内查看到 ossec-server 输出的日志了。