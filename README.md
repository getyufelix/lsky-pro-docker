# Lsky-Pro Docker镜像

~~每天自动拉取最新代码构建Docker镜像，~~ 现已上传amd64和arm64两种硬件架构。
> 由于开源版长时间未维护，目前暂停了自动拉取代码上传。待开源版有更新之后会恢复自动构建

## 使用方法

```docker
docker run -d \
    --name lsky-pro \
    --restart unless-stopped \
    -p 8089:8089 \
    -v $PWD/lsky:/var/www/html \
    -e WEB_PORT=8089 \
    halcyonazure/lsky-pro-docker:latest
```

## 环境变量

目前该容器只有一个环境变量：`WEB_PORT`，用于指定容器内的`Apache`监听的端口，默认为`8089`，如果需要修改的话可以在启动容器时添加`-e WEB_PORT=8089`来指定端口

## Docker-Compose部署参考

```yaml
version: '3'
services:
  lskypro:
    image: getyufelix/lsky-pro-docker:latest
    restart: unless-stopped
    hostname: lskypro
    container_name: lskypro
    environment:
      - WEB_PORT=8089
    volumes:
      - $PWD/web:/var/www/html/
    ports:
      - "8089:8089"
    networks:
      - lsky-net

  # 注：arm64的无法使用该镜像，请选择sqlite或自建数据库
  mysql-lsky:
    image: mysql:5.7.22
    restart: unless-stopped
    # 主机名，可作为"数据库连接地址"
    hostname: mysql-lsky
    # 容器名称
    container_name: mysql-lsky
    # 修改加密规则
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - $PWD/mysql/data:/var/lib/mysql
      - $PWD/mysql/conf:/etc/mysql
      - $PWD/mysql/log:/var/log/mysql
    environment:
      MYSQL_ROOT_PASSWORD: password # 数据库root用户密码，自行修改
      MYSQL_DATABASE: lsky-data # 可作为"数据库名称/路径"
    networks:
      - lsky-net

networks:
  lsky-net: {}
```

原项目：[☁️兰空图床(Lsky Pro) - Your photo album on the cloud.](https://github.com/lsky-org/lsky-pro)

## 构建您自己的镜像

现在，您可以通过提供的Dockerfile直接构建自己的Lsky-Pro镜像。Dockerfile已经配置为多段构建，不再需要手动拉取源码。下面的命令展示了如何构建镜像：

```bash
docker build -t lsky-pro-docker .
```

如果您想为不同的硬件架构构建镜像（例如，arm64或amd64），您可以使用以下命令：

```bash
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t lsky-pro-docker .
```

## 手动备份/升级

如果需要迁移数据库/手动升级`Lsky-Pro`，可以参考官方文档：[升级｜Lsky Pro](https://docs.lsky.pro/docs/free/v2/quick-start/upgrade.html)，来备份主要文件以进行恢复/升级
