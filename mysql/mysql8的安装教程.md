# 安装 MySQL

* MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，属于 Oracle 旗下产品。
* MySQL 是最流行的关系型数据库管理系统之一。
* 在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。

## 一、Mysql的下载

**下载方式：**

可以分为==压缩包==和==Msi==，推荐使用压缩包安装，Msi文件安装容易出现错误。

### 压缩包

下载链接：https://dev.mysql.com/downloads/mysql/

![image-20200413160013770](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413160013770.png)

### Msi

下载链接：https://dev.mysql.com/downloads/windows/installer/8.0.html

![image-20200413160101471](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413160101471.png)



## 二、Mysql的安装

### 1. **解压缩**

下载完后，我们将 zip 包解压到相应的目录，这里我将解压后的文件夹放在 **D:\environment\mysql** 下。

**注意：安装路径最好不要有中文，避免出现错误。**

 ![image-20200413160734346](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413160734346.png)

### 2. 配置文件

接下来我们需要配置**MySQL**的配置文件

1. 进入**Mysql**解压缩的文件夹
2. 在文件夹下创建==my.ini==文件

 ![image-20200413161259467](picture\image-20200413161259467.png)

### 3. 编辑my.ini文件

```bash
[client]
# 设置mysql客户端默认字符集
default-character-set=utf8
 
[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=D:\\environment\\mysql
# 设置 mysql数据库的数据的存放目录，MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
# datadir=C:\\web\\sqldata
# 允许最大连接数
max_connections=20
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

注意：==basedir==需要修改为自己的mysql安装目录。

### 4. 以<font color=red>管理员</font>身份打开命令行窗口

 ![image-20200413162629933](picture\image-20200413162211082.png)

此处用cd命令切换到**mysql**的解压缩目录下的==bin==目录下

### 5. 初始化数据库

注意：Mysql8.0之后会自动生成data文件夹。

```bash
#第一种，不设置root密码，进入数据库后可自行设置
mysqld  --initialize-insecure
#第二种，在控制台生成一个随机的root密码，进入数据库后可自行修改
mysqld  --initialize --console
```

将命令复制到命令行窗口运行即可，在这里我使用第二种生成随机密码。

![image-20200413163022533](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413163022533.png)

可以看到文件下自动生成了data文件夹，命令行也显示了初始密码 ==6WA_-zr,3;oe==

### 6. 安装Mysql服务

```bash
#安装mysql服务
mysqld install mysql
 
#卸载mysql服务
sc delete mysql(需要管理员权限)
 
#移除mysql服务（需要停止mysql）
mysqld -remove
```

执行成功之后一般会显示==Service successfully installed.==

 ![image-20200413163451463](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413163451463.png)

### 7. 启动mysql服务

```bash
net start mysql
```

执行结果：

![image-20200413163631022](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413163631022.png) 

这时候mysql服务已经正常启动，可以登录了。

### 8. 登录mysql

```bash
#登录格式,-u:代表username,-p代表password
mysql -u 用户名 -p
#回车之后输入密码即可
```

注意：如果执行初始化mysql命令的时候选择不生成随机密码，则-p可以省略，直接输入`mysql -u 用户名`即可登录。

我这里使用==root==用户和刚才随机生成的密码==6WA_-zr,3;oe==登录

```bash
mysql -u root -p 6WA_-zr,3;oe
```

![image-20200413164616808](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413164616808.png) 

![image-20200413164038244](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413164038244.png)

登录成功之后末行会变成：`mysql>`

### 9. 修改密码

```bash
#进入mysql数据库
use mysql;
 
#修改root用户的密码为123456，根据需要自己设置
alter user 'root'@localhost identified by '123456';
 
#刷新权限,一般修改密码或授权用户的时候需要使用
flush privileges;

#退出mysql,以下两个命令都可以正常退出数据库
quit
exit
#非正常退出：Ctrl+C
```

![image-20200413165344149](picture\image-20200413165344149.png)

进入失败，显示必须要先修改密码。

修改密码后刷新权限。

 ![image-20200413165502102](picture\image-20200413165502102.png)

这时候就可以正常使用数据库了

 ![image-20200413165559461](picture\image-20200413165559461.png)

输入`exit`正常退出

![image-20200413165904067](picture\image-20200413165904067.png) 

使用`mysql -u root -p `再次登录试试。

![image-20200413170028046](picture\image-20200413170028046.png) 

登录成功，修改密码结束。

### 10. 配置环境变量

**为什么要配置环境变量？**

配置了环境变量之后可以在任何目录下使用mysql命令，而不用特意进入到mysql的bin目录下。

1. 右键我的电脑，点击属性
2. 点击高级系统设置
3. 点击右下角的环境变量
4. 找到==Path==变量，双击进行编辑
5. 在最后一行添加，`D:\environment\mysql\bin`，也就是你mysql的bin目录路径

![image-20200413170709793](picture\image-20200413170709793.png) 

6. 点击确定，修改完成。

**环境变量之后可以试试在其他文件夹使用mysql命令是否生效。**

## 三、Navicat连接mysql

### 1. 新建连接

点击连接，选择mysql进行新建连接。

![image-20200413171028957](picture\image-20200413171028957.png) 

### 2. 输入mysql信息

输入mysql的连接信息：

连接名：连接名可以随便起；

主机：默认；端口：默认；

用户名：root；密码：你设置的密码；

![image-20200413171259483.png](https://gitee.com/xuweiquanshi/PictureBed/raw/master/img/image-20200413171259483.png)

输入完后，可以点击测试连接，测试连接成功，就证明信息无误，点击确定保存。

### 3. 打开mysql连接

![image-20200413172514201](picture\image-20200413172514201.png) 

双击打开mysql连接，可以看到mysql的数据库，就证明连接成功。