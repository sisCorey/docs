# Sentry Web 部署

## Requirements

1. Docker 19.03.6+
1. Compose 1.24.1+
1. 4 CPU Cores
1. 8 GB RAM
1. 20 GB Free Disk Space


## Setup

1. 启动基础数据服务 `redis & postgres`

    ```shell
    $ docker run -d --name sentry-redis redis 

    $ docker run -d --name sentry-db -e POSTGRES_PASSWORD="<pswd>" -e POSTGRES_USER=sentry postgres

    ```

    **通过环境变量 `POSTGRES_USER`, `POSTGRES_PASSWORD` 设置DB访问账号。**

    **DB也可用 `mysql` 替换**


2. 生成服务私钥
    
    ```shell
    $ docker run --rm -it sentry config generate-secret-key
    # or
    $ sentryPriKey=(docker run --rm -it sentry config generate-secret-key)

    ```

    **可把私钥保存到环境变量中，方便后续步骤使用**

3. 服务数据初始化

    #### 1. 官方最简易用法

    ```shell
    # 数据库初始化
    $ docker run -it --rm -e SENTRY_SECRET_KEY="${sentryPriKey}" --link sentry-db:postgres --link sentry-redis:redis sentry upgrade

    # 创建 web 缺省用户
    $ docker run -it --rm -e SENTRY_SECRET_KEY="${sentryPriKey}" --link sentry-db:postgres --link sentry-redis:redis sentry createuser
    ```
    
    **实际中 --link 效果在各 docker 版本中差异较大，通过变量设置 redis 与 db 的的 host、port、passwd 比较稳妥**

    #### 2. 复杂稳妥用法

    ```shell
    # 数据库初始化
    $　docker run -it --rm -e SENTRY_SECRET_KEY="${sentryPriKey}" -e SENTRY_REDIS_HOST="${SENTRY_REDIS_HOST}" -e SENTRY_POSTGRES_HOST="${SENTRY_POSTGRES_HOST}" -e SENTRY_DB_USER="${SENTRY_DB_USER}" -e SENTRY_DB_PASSWORD="${SENTRY_DB_PASSWORD}" --network=sentry_default sentry upgrade

    # 创建 web 缺省用户
    $ docker run -it --rm -e SENTRY_SECRET_KEY="${sentryPriKey}" -e SENTRY_REDIS_HOST="${SENTRY_REDIS_HOST}" -e SENTRY_POSTGRES_HOST="${SENTRY_POSTGRES_HOST}" -e SENTRY_DB_USER="${SENTRY_DB_USER}" -e SENTRY_DB_PASSWORD="${SENTRY_DB_PASSWORD}" --network=sentry_default sentry createuser

    ```

    **不使用 `--link` 时，需要设置 `--network` 参数**


4. 启动 Sentry Web

    ```shell
    # simple
    $ docker run -d --name sentry-web -e SENTRY_SECRET_KEY="${sentryPriKey}" --link sentry-db:postgres -p 9000 --link sentry-redis:redis sentry

    # with environment setting
    $ docker run -d --name sentry-web -e SENTRY_SECRET_KEY="${sentryPriKey}" -e SENTRY_REDIS_HOST="${SENTRY_REDIS_HOST}" -e SENTRY_POSTGRES_HOST="${SENTRY_POSTGRES_HOST}" -e SENTRY_DB_USER="${SENTRY_DB_USER}" -e SENTRY_DB_PASSWORD="${SENTRY_DB_PASSWORD}" -p 9000 --network=sentry_default sentry 

    ```


5. 启动 worker & cron

    ```shell
    # simple
    $ docker run -d --name sentry-cron -e SENTRY_SECRET_KEY="${sentryPriKey}" --link sentry-db:postgres --link sentry-redis:redis sentry run cron

    $ docker run -d --name sentry-worker-1 -e SENTRY_SECRET_KEY="${sentryPriKey}" --link sentry-db:postgres --link sentry-redis:redis sentry run worker
    

    # with environment setting
    $ docker run -d --name sentry-cron -e SENTRY_SECRET_KEY="${sentryPriKey}" -e SENTRY_REDIS_HOST="${SENTRY_REDIS_HOST}" -e SENTRY_POSTGRES_HOST="${SENTRY_POSTGRES_HOST}" -e SENTRY_DB_USER="${SENTRY_DB_USER}" -e SENTRY_DB_PASSWORD="${SENTRY_DB_PASSWORD}" --network=sentry_default sentry 

    $ docker run -d --name sentry-worker-1 -e SENTRY_SECRET_KEY="${sentryPriKey}" -e SENTRY_REDIS_HOST="${SENTRY_REDIS_HOST}" -e SENTRY_POSTGRES_HOST="${SENTRY_POSTGRES_HOST}" -e SENTRY_DB_USER="${SENTRY_DB_USER}" -e SENTRY_DB_PASSWORD="${SENTRY_DB_PASSWORD}" --network=sentry_default sentry 

    ```

    **按官方文档说，worker 实例可尽可能的创建多个**


6. 建议使用 `docker-compose.yml` 来部署。


7. 至此部署完毕，通过浏览器访问 `http://localhost:9000` ，用第 `3` 步创建的账号登录，并创建 `project`。创建后可在菜单 `Settings` -> `Client Keys(DSN)` 中获取 `dsn`。

