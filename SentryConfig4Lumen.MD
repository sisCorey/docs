# ERP Lumen 接入 Sentry 指引

## 项目部署 Sentry

1. 安装 Sentry SDK

    ```shell
    compose require sentry/sentry-laravel

    ```

2. 添加异常信息上报 

    vim `app/Exception/Handler.php`

    ```php
    public function report(Exception $e)
    {
        if (app()->bound('sentry') && $this->shouldReport($e)) {
            app('sentry')->captureException($e);
        }
        ...
    }

    ```

3. 添加 Provider 
    
    vim `bootstrap/app.php`

    ```php
    $app->register(Sentry\Laravel\ServiceProvider::class);

    ```

4. 添加 sentry 配置

    ```shell
    cp vendor/sentry/sentry-laravel/config/sentry.php config/sentry.php

    ```

5. 配置 sentry dsn

    vim `.env`

    ```php
    SENTRY_LARAVEL_DSN=http://<pub-key>@<host>:9000/1

    ```

    **`dsn` 在 `Sentry Web` 中的 `Settings` -> `Client Keys(DSN)` 中获取**
    

## Preference

[Sentry DockerHub](https://hub.docker.com/_/sentry)

[Sentry Composer Package](https://packagist.org/packages/sentry/sentry)

[Sentry Github](https://github.com/getsentry/sentry-php)
