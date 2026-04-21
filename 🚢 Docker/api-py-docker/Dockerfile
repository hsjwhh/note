# 使用官方 Python 镜像
FROM python:slim

# 设置工作目录
WORKDIR /app

# 复制项目文件
COPY app/. /app/

# 更新包列表并安装 C 编译器和构建工具, 结束后清理无用文件节约空间
RUN apt-get update && \
    apt-get install -y build-essential && \
    pip install --no-cache-dir -r requirements.txt && \
    apt-get remove -y build-essential && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 暴露 uWSGI 端口
EXPOSE 5000

# 启动 uWSGI
CMD ["uwsgi", "--ini", "./uwsgi/uwsgi.ini"]