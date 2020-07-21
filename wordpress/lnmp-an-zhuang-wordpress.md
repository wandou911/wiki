# LNMP安装Wordpress

环境和工具：centos7服务器一台，内存不小于1G；Xshell，root账号和密码，一字不漏地看文章并严格按步骤执行。

## 1 安装LNMP环境

本项目将会用到Nginx（web程序），php（web解析环境）和mysql（数据库）。其中PHP版本必须是7.1，mysql在5.5以上。其他无要求，系统最好是centos7，博主在centos7上测试正常。

首先用xshell连接到你的服务器，root账户登录

* 登录：

`ssh root@12.34.56.78`

* 安装wget：

`yum -y install wget`

* 然后一键安装LNMP环境：

`wget -c http://soft.vpser.net/lnmp/lnmp1.4.tar.gz && tar zxf lnmp1.4.tar.gz && cd lnmp1.4 && ./install.sh lnmp`

上面命令如果安装失败，用下面这个

`wget -c http://soft.vpser.net/lnmp/lnmp1.6.tar.gz && tar zxf lnmp1.6.tar.gz && cd lnmp1.6 && ./install.sh lnmp`

按如图方式选择：mysql5.5 php7.1 并设置数据库的root密码。

![pic4](https://www.xiaoweigod.com/ueditor/php/upload/image/20171106/1509939038433502.png)

让安装跑一会儿，具体时间根据你的服务器配置决定。

安装成功会出现如下图：

![image.png](https://www.xiaoweigod.com/ueditor/php/upload/image/20171106/1509942028391657.png)

## 2 建立一个站点

将你的将域名解析到你的服务器，如没有域名可不解析。切回xshell，ctrl+c取消掉之前lnmp安装完成的提示。然后服务器上执行：

`lnmp vhost add`

![image.png](https://www.xiaoweigod.com/ueditor/php/upload/image/20171106/1509943662814585.png)

如图我使用ss.xiaoweigod.com作为域名。如果没有域名那么直接填写IP地址就行了，将网站路径设置为/www/example.com

## 3 安装WordPress

### 2.1 下载并解压wordpress安装包

```text
cd /www/example.com   #后面的example.com 改成你自己的主域名，如果自定义了网站目录改成你的网站目录路径#

wget https://wordpress.org/latest.tar.gz

tar -xzvf latest.tar.gz #解压缩wordpress安装包

mv wordpress/*  .       #拷贝wordpress程序至网站根目录

rm -rf wordpress latest.tar.gz      #删除wp压缩包#

chown www:www -R /www/example.com    #设置权限
```

### 2.2 创建数据库和用户

浏览器中输入：你的服务器地址/phpmyadmin 打开页面，登录账号是root，密码是安装LNMP时候设置的数据库密码。

1 登录后创建一个名为wordpress的数据库

![image.png](https://i0.wp.com/wordpress.org/support/files/2018/10/phpMyAdmin_create_database_4.4.jpg?ssl=1)

2 创建用户wordpress

![image](https://i2.wp.com/codex.wordpress.org/File:users.jpg?ssl=1)

3 返回权限（Privileges）界面，点击刚刚创建的WordPress用户上的查看权限（Check privileges）图标。在详细数据库权限（Database-specific privileges）界面中，在为以下数据库添加权限下拉式菜单中选择之前创建的WordPress数据库。之后页面会刷新为该WordPress数据库的权限详情。点击选中所有，选择所有权限（Check All），最后点击Go。

4 在结果页面上，记下页面最上方Server:后的主机名hostname（通常为localhost）。

### 2.3 设置wp-config.php文件

返回第一步中解压WordPress压缩包的位置，将wp-config-sample.php重命名为wp-config.php，之后在文本编辑器中打开该文件。

在标有

> //  **MySQL settings - You can get this info from your web host**  //

下输入你的数据库相关信息

```text
DB_NAME 
在第二步中为WordPress创建的数据库名称
DB_USER 
在第二步中创建的WordPress用户名
DB_PASSWORD 
第二步中为WordPress用户名设定的密码
DB_HOST 
第二步中设定的hostname（通常是localhost，但总有例外；参见编辑wp-config.php文件中的“可能的DB_HOST值）。
DB_CHARSET 
数据库字符串，通常不可更改（参见zh-cn:编辑wp-config.php）。
DB_COLLATE 
留为空白的数据库排序（参见zh-cn:编辑wp-config.php）。
```

保存wp-config.php文件

### 2.4 安装wordpress

在浏览器中输入你的域名，如 www.vpser.net

![](https://www.vpser.net/uploads/2019/03/lnmp-wordpress-howto-wp-install1.png)

点击"现在就开始！"，下面需要设置数据库名、数据库用户名、该数据库用户的密码。

![](https://www.vpser.net/uploads/2019/03/lnmp-wordpress-howto-wp-install2.png)

如果按前面"虚拟主机添加教程"创建的数据库，数据库名和用户名相同，密码为你设置的。另外两项数据库主机和表前缀保持默认就可以。

填写好后，点击“提交”。

![](https://www.vpser.net/uploads/2019/03/lnmp-wordpress-howto-wp-install3.png)

点击"现在安装"。

![](https://www.vpser.net/uploads/2019/03/lnmp-wordpress-howto-wp-install4.png)

填写你的邮箱就可以了，最后确认信息没错后点击"安装wordpress"。

稍等片刻就会提示安装成功，如下图：

![](https://www.vpser.net/uploads/2019/03/lnmp-wordpress-howto-wp-install5.png)

输入你的用户名和密码后就可以登录到wordpress后台。

![](https://www.vpser.net/uploads/2019/03/lnmp-wordpress-howto-wp-install6.png)

wordpress后台界面如下图：

![](https://www.vpser.net/uploads/2019/03/lnmp-wordpress-howto-wp-admin.png)

wordpress的安装教程到此结束

#### 参考链接:

[网站运行环境LNMP安装+wordpress安装+https访问](https://www.vpser.net/build/lnmp-wordpress-howto-3.html)

