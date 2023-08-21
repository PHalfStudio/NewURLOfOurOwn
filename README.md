# New URL of Our Own
New URL of Our Own 是面对 [archiveofourown.org ](https://archiveofourown.org)（下称 AO3）网站 的一个提供反向代理配置的项目。

该项目旨在帮助各位内容创作者自由地进行创作，保障在特殊网络情形下对 AO3 网站的正常访问.

本项目修改自原作者 [@ExcitedCodes](https://github.com/ExcitedCodes) 的项目[URLOfOurOwn](https://github.com/ExcitedCodes/URLOfOurOwn)

原作者于2020年公开，随后停止维护，Nginx配置文件已经无法用于现状

本人修改了原作者的代码，使之可用，

## 搭建教程

1. 首先，你需要准备一个在目标地区可以正常解析域名和一个能访问到 AO3 的服务器（即位于海外），并且拥有 `root` 访问权限；

   * _我们建议使用主流 Linux 发行版，如 Ubuntu、CentOS 等。_
   * _我们建议使用注册于海外域名注册商，并且开启了域名隐私保护服务的域名作为反代域名。_
   * _我们建议使用 [Cloudflare](https://cloudflare.com) 。申请账户和添加域名请见（[官方文档](https://support.cloudflare.com/hc/zh-cn/articles/201720164-%E5%88%9B%E5%BB%BA-Cloudflare-%E5%B8%90%E6%88%B7%E5%B9%B6%E6%B7%BB%E5%8A%A0%E7%BD%91%E7%AB%99))_

2. 接下来，在你的服务器上安装必要软件
   * [Nginx](https://nginx.org/) - 请参阅 [安装 Nginx](#%E5%AE%89%E8%A3%85-nginx) 一节
   * 未来操作需要用到的软件 - [git](https://git-scm.com) 和 [vim](https://www.vim.org):
      * Ubuntu: `sudo apt update && apt install -y git vim`
      * CentOS: `sudo yum update && yum install -y git vim`

3. 使用 `git clone` 命令复制所需的文件：在任意路径执行 `git clone https://github.com/PHalfStudio/NewURLOfOurOwn.git`

4. 准备 Nginx 配置文件：在当前执行 `cp NewURLOfOurOwn/proxy/nginx/* /etc/nginx/conf.d`

5. 准备一个放置代理文件的路径，可以直接在同一目录执行 `cp -r NewURLOfOurOwn/proxy/ao3 /var/www/html/ao3`；

6. 若安装方式不同，`Nginx` 的安装目录也可能会不同；若 `Nginx` 是用其他方式安装，你可以使用 `whereis nginx` 命令快速查找 `Nginx` 的位置

7. 修改配置文件使其符合你的域名，请全程使用**英文输入方式**，并注意不要误删任何无关的字符；执行 `vim /etc/nginx/conf.d/site.conf`（也可以使用任何其他你喜爱的编辑器），下文介绍 `vim` 的使用方法：
   * 按一下 i 进入编辑模式，你应该会注意到左下角出现 `-- INSERT --` 字样

   * 在第3行和第10行左右找到 `server_name <Fill_Domain>;` 并将其替换为你的域名，如 `server_name  ao3.wtf;`

   * 在第19行和20行找到`<Pem_Path>`和`<key_Path>`关键字，删除后面的{}，并填入自己域名的SSL证书

     （如没有申请，请参阅下方申请SSL证书环节）

   * 在第27行左右找到 `root <Fill_AO3>;` 并将其替换为第5步中的路径. 如果你直接执行了那行路径，请输入 `/var/www/html/ao3`

   * 我们不提供配置缓存的教程也不建议您配置缓存, 因为这可能引发一系列安全问题

   * 如果你 __不__ 使用 Cloudflare

     （**注意：本修改版仅测试了使用Cloudflare CDN代理后的情况，如操作后网页加载失败，请自行解决**）

     * 找到 `$http_cf_connecting_ip` 并替换为 `$remote_addr`
     * 找到文件末尾的 `include conf.d/cloudflare.inc;` 和 `deny all;` 两行并在最前面加上 `#`

   * 配置完毕后按下 ESC 按键并输入 `:wq`，按回车退出 vim

8. 接下来，启动（或重启）你的 Nginx，执行 `systemctl restart nginx` 或者 `service nginx restart`
   * 如果你希望 Nginx 在未来自动启动，请执行 `systemctl enable nginx`

9. 解析域名到你的服务器 IP；可以使用 `curl ipv4.ip.sb` 命令快速查询服务器 IP

10. 现在，你的代理网站应该可以工作了。~尝试在浏览器中访问它！

### 安装 Nginx

暂时只提供 Linux 下的安装简介

* Ubuntu 18.04:
   * [详细教程（英文）](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)
   * 简化版本:
     ```console
     $ sudo apt update
     $ sudo apt install nginx
     $ sudo ufw allow 'Nginx HTTP'
     ```
 * CentOS 7:
   * [详细教程（英文）](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7)
   * 简化版本:
     ```console
     $ sudo yum install epel-release
     $ sudo yum install nginx
     $ sudo firewall-cmd --permanent --zone=public --add-service=http
     $ sudo firewall-cmd --reload
     ```

### 便捷申请SSL证书

项目前身 [@ExcitedCodes](https://github.com/ExcitedCodes/URLOfOurOwn) 的Nginx配置文件已经失效，经过本人修改后发现缺失了部分代码以及需要HTTPS链接

暂时只提供Linux下申请SSL证书的申请教程

大体分两步，首先安装Snapd，再安装SSL证书申请工具

#### CentOS 7:

- 安装Snapd

  - [详细教程【英文】](https://snapcraft.io/docs/installing-snap-on-centos)

  - 简化版本：

    ```
    $ sudo yum install snapd
    $ sudo systemctl enable --now snapd.socket
    $ sudo ln -s /var/lib/snapd/snap /snap
    ```

- 安装CertBot并申请SSL证书

  - [详细教程【英文】](https://certbot.eff.org/instructions?ws=nginx&os=centosrhel7)

  - 简化版本

    ```
    $ sudo snap install --classic certbot
    $ sudo ln -s /snap/bin/certbot /usr/bin/certbot
    # 自动安装
    $ sudo certbot --nginx
    
    #只是想申请证书
    $ sudo certbot certonly --nginx
    ```

    输入安装命令后请根据命令行提示自行申请SSL证书

    如果选择自动安装后你应该会在`/etc/nginx/conf.d/site.conf`文件中看到这样的字样

    ```
    ssl_certificate /etc/letsencrypt/live/xxx.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/xxx.com/privkey.pem; # managed by Certbot
    ```

​				

#### Ubuntu 20：

- 安装Snapd（如原系统中并没有安装，一般来说Ubuntu会自带Snapd）

  - [详细教程【英文】](https://snapcraft.io/docs/installing-snap-on-ubuntu))

  - 简化版本：

    ```
    $ sudo apt update
    $ sudo apt install snapd
    # 重新启动系统
    $ reboot
    
    ```

- 安装CertBot并申请SSL证书

  - [详细教程【英文】](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal)

  - 简化版本

    ```
    $ sudo snap install --classic certbot
    $ sudo ln -s /snap/bin/certbot /usr/bin/certbot
    # 自动安装
    $ sudo certbot --nginx
    
    #只是想申请证书
    $ sudo certbot certonly --nginx
    ```

    输入安装命令后请根据命令行提示自行申请SSL证书

    如果选择自动安装后你应该会在`/etc/nginx/conf.d/site.conf`文件中看到这样的字样

    ```
    ssl_certificate /etc/letsencrypt/live/xxx.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/xxx.com/privkey.pem; # managed by Certbot
    ```

​				
