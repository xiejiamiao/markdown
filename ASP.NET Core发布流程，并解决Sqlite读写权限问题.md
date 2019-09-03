
### 发布流程
> 参考教程：https://docs.microsoft.com/zh-cn/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-2.2
1. 安装 **.NET Core Runtime**
    > 官网教程：https://dotnet.microsoft.com/download/linux-package-manager/ubuntu16-04/runtime-current
    ```bash
    wget -q https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    sudo apt-get install apt-transport-https
    sudo apt-get update
    sudo apt-get install aspnetcore-runtime-2.2
    ```
2. 安装 **.NET Core SDK** (因为需要在服务器段编译发布代码，所以需要安装SDK)
    > 官网教程：https://dotnet.microsoft.com/download/linux-package-manager/ubuntu16-04/sdk-current
    ```bash
    wget -q https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    sudo apt-get install apt-transport-https
    sudo apt-get update
    sudo apt-get install dotnet-sdk-2.2
    ```
3. 安装`Nginx`(参见nginx官网)
    > 官网教程：https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#official-debian-ubuntu-packages
    ```bash
    sudo apt-get update
    sudo apt-get install nginx
    ```
4. 启动`Nginx`
    ```bash
    sudo service nginx start
    ```
5. 使用sftp上传代码到服务器上(以服务端路径为`/var/www/webapi`为例)
6. 进入项目 (*当有多个项目的时候则进入启动项目*)
    ```bash
    cd /var/www/webapi
    ```
    执行发布命令
    ```bash
    dotnet publish --configuration Release
    ```
    此时在`var/www/webapi/bin/Release/netcoreapp2.2/publish/`目录会生成发布文件
7. 在发布目录里执行`dotnet webapi.dll`启动项目，此时通过ip+端口应该可以访问

8. 配置`Nginx`,修改文件`/etc/nginx/sites-available/default`，添加以下内容
    ```nginx
    server {
        server_name   api.demo.com;  # 域名
        location / {
            proxy_pass         http://localhost:5000;  # 项目启动起来的本地地址
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection keep-alive;
            proxy_set_header   Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }
    }
    ```
    测试`Nginx`配置文件是否正确
    ```bash
    sudo nginx -t
    ```
    重载配置文件
    ```bash
    sudo nginx -s reload
    ```
9. 做域名解析
10. 监控应用(出错自动重启，监视日志等)
    创建文件 `sudo nano /etc/systemd/system/kestrel-webapi.service`，文件内容如下：
    ```ini
    [Unit]
    Description=WebApi running on Ubuntu

    [Service]
    WorkingDirectory=/var/www/webapi
    ExecStart=/usr/bin/dotnet /var/www/webapi/webapi.dll
    Restart=always
    # Restart service after 10 seconds if the dotnet service crashes:
    RestartSec=10
    KillSignal=SIGINT
    SyslogIdentifier=dotnet-example
    User=www-data
    Environment=ASPNETCORE_ENVIRONMENT=Production
    Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

    [Install]
    WantedBy=multi-user.target
    ```
    保存文件并启用服务
    ```bash
    sudo systemctl enable kestrel-webapi.service
    ```
    启用服务
    ```bash
    sudo systemctl start kestrel-webapi.service
    ```
    查询服务状态
    ```bash
    sudo systemctl status kestrel-webapi.service
    ```
    查看服务日志
    ```bash
    sudo journalctl -fu kestrel-webapi.service
    ```
11. 使用`certbot`配置`HTTPS`
    >  参考官网：https://certbot.eff.org

    选择`nginx + ubuntu16.06`的配置，参照官网执行以下命令
    ```bash
    sudo apt-get update
    sudo apt-get install software-properties-common
    sudo add-apt-repository universe
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt-get update
    sudo apt-get install certbot python-certbot-nginx
    sudo certbot --nginx
    ```
    执行过程需要填邮箱，确定是否将http请求默认跳转到https等配置

### 解决权限问题

执行到这里应该就可以通过域名访问部署的项目，项目使用`EFCore + Sqlite`，测试的时候可正常查询，在更新数据的时候一直失败，查看服务日志发现以下错误，推测应该是文件读写权限问题
```
 Microsoft.Data.Sqlite.SqliteException (0x80004005): SQLite Error 8: 'attempt to write a readonly database'.
```
尝试使用：`chmod g+w db.sqlite`和`chown www-data db.sqlite`修改权限和文件所有者均无效

后来发现修改服务文件用户，将`/etc/systemd/system/kestrel-webapi.service`文件里的`User`节点修改文`root`，具体内容如下：
```ini
[Unit]
Description=WebApi running on Ubuntu

[Service]
WorkingDirectory=/var/www/webapi
ExecStart=/usr/bin/dotnet /var/www/webapi/webapi.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=root
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```
保存并重启服务，测试可正常更新，查看日志不再发现错误
