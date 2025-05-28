
# 苹果CMS 影视站搭建教程

## 简介

**苹果CMS** 是一套基于 PHP + MySQL 构建的免费影视站建站程序，具有强大的采集与播放管理功能，适合个人搭建影视资源导航平台。

- 项目地址：[https://github.com/magicblack/maccms10](https://github.com/magicblack/maccms10)

> ⚠️ 本教程仅供技术学习参考，所采集内容均来自公开网络。请合法使用，避免法律风险。

---

## 一、准备工作

### 环境要求

- 操作系统：Linux（如 Debian / Ubuntu）
- Web服务：Apache（或 Nginx）
- PHP版本：推荐 PHP 7.2 ~ 7.4
- 数据库：MySQL 5.6+

### 安装基础组件

```bash
sudo apt update
sudo apt install php unzip apache2 mysql-server
```

---

## 二、下载与部署苹果CMS程序

```bash
cd /var/www
wget https://github.com/magicblack/maccms10/archive/refs/tags/v2023.1000.3052.zip
unzip v2023.1000.3052.zip
rm -rf html/
rm v2023.1000.3052.zip
mv maccms10-2023.1000.3052/ html/
sudo chmod -R 777 html
```

> **访问地址**：在浏览器中访问 `http://你的服务器IP` 进入安装界面。
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/1796422294.png)

> **注意**：安装界面中提示部分目录权限不足，请根据红色提示信息修改权限。
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/120094303.png)
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/670701090.png)

---

## 三、安装 PHP 扩展及配置

### 安装必要的 PHP 扩展

```bash
sudo apt install php-mysql php-xml php-zip php-curl php-mbstring
sudo service apache2 restart
```

### 修改 `php.ini` 配置

1. 编辑配置文件：
   ```bash
   sudo vim /etc/php/7.2/apache2/php.ini
   ```
2. 启用所需扩展（去除注释符`;`），或手动添加：
   ```
   extension=mysqli
   extension=curl
   extension=mbstring
   extension=zip
   extension=xml
   ```
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/1513967122.png)
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/3845578306.png)
---

## 四、安装数据库

你可以使用本地或远程数据库，安装完成后配置连接信息即可。

> 🔗 若需远程访问数据库，请确保 MySQL 已开启远程连接权限，并开放 3306 端口。
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/539128736.png)
---

## 五、后台管理与功能配置

### 进入后台

安装完成后，访问后台地址 `http://你的IP/admin.php`，登录管理系统。
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/4218347839.png)

### 采集设置

以非凡采集为例，采集接口如下：

```
http://cj.ffzyapi.com/api.php/provide/vod/from/ffm3u8/at/xml/
```

将接口地址填入后台采集配置中即可开始数据采集。
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/791380784.png)
### 编辑分类与绑定

点击后台“分类管理”，可根据需求自定义影视分类
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/392450111.png)
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/3766457985.png)

将采集到的资源绑定到指定分类。
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/1565322967.png)
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/718865274.png)
---

## 六、播放器配置

完成采集后，请按模板说明配置播放器解码方式和解析规则，确保能正常在线播放视频资源。
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/2281502209.png)
---

## 七、常见问题解决

### 1. 修改数据无效

```bash
cd /var/www
sudo chmod -R 777 html
```

### 2. 提示“重新安装”

设置以下目录权限后再次访问安装界面：

```bash
chmod -R 777 /var/www/html/application/data/install/
```

### 3. 后台验证码显示异常或错误

```bash
cd /var/www/html/application/extra/
vim maccms.php
```
![docker placeholder](https://blog.hgtrojan.com/usr/uploads/2024/02/3756876953.png)
将配置项 `admin_login_verify` 的值改为 `0` 可暂时关闭验证码：

```php
'admin_login_verify' => 0
```

> ⚠️ **安全警告**：关闭验证码可能被暴力破解，请在配置完成后重新开启，或通过修改 `admin.php` 文件名加强保护。

---

## 八、推荐资源站 & 模板

### CMS系统
- [苹果CMS V10](https://www.maccms.la/)：https://www.maccms.la/
- [海洋CMS](https://www.seacms.net/)：https://www.seacms.net/
- [飞飞CMS 5.0](http://www.feifeicms.vip/portal.php)：http://www.feifeicms.vip/portal.php
- [赞片CMS](https://www.zanpiancms.com/)：https://www.zanpiancms.com/

### 采集接口
- [暴风采集](https://publish.bfzy.tv/)：https://publish.bfzy.tv/
- [非凡采集](http://ffzy5.tv/)：http://ffzy5.tv/
- [快看采集](https://kuaikanzy.net/)：https://kuaikanzy.net/
- [乐视采集](https://www.leshizy1.com/)：https://www.leshizy1.com/

---

## 九、重要提示

> ⚠️ **请设置访问IP白名单，仅限个人学习使用，防止被滥用或侵权。**
>
> ⚠️ 本教程从**技术角度**出发，**不提供任何违法内容的支持或推荐**，使用造成的一切后果由使用者自行承担。
