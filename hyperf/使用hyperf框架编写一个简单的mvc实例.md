创建一个hyperf项目
=============

在docker创建一个新的hyperf项目并挂载到window
```
docker run -d --name hyperfmvc -v d/dataproject/project3:/hyperf-skeleton -p 9501:9501 -it --entrypoint /bin/sh hyperf/hyperf:7.4-alpine-v3.11-swoole
```
参数解释：

-d 表示在后台运行

--name 表示给容器取个名字

-v 表示挂载到本机的目录

-p 表示映射到主机的端口，冒号前的是本机，后面是docker容器端口

-it 终端

--entrypoint 通过这个来覆盖dockerfile里定义的entrypoint

镜像容器运行后，然后进入容器中，在容器内安装 Composer


```
#下载composer.phar文件
wget https://github.com/composer/composer/releases/download/1.8.6/composer.phar
#给composer.phar文件加可执行权限
chmod u+x composer.phar
#移动该文件到/usr/local/bin/composer
mv composer.phar /usr/local/bin/composer
```
将 Composer 镜像设置为阿里云镜像，加速国内下载速度
```
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer
```
通过 Composer 安装 hyperf/hyperf-skeleton 项目
```
composer create-project hyperf/hyperf-skeleton
```

进入安装好的 Hyperf 项目目录，linux目录
```
cd hyperf-skeleton
```

启动 Hyperf
```
php bin/hyperf.php start
```

可能报的错误
======
```
[ERROR] RedisException: Connection refused(0) in /hyperf-skeleton/vendor/hyperf/redis/src/RedisConnection.php:246
```

如果报了这个错误是因为没有连接上redis

处理方法
----

使用docker创建一个redis 容器,-p是将docker容器的端口映射到主机端口上
```
docker run -d -p 6379:6379 redis /bin/bash
```
打开powershell，查看ipv4，之所以查看ip，是因为docker容器在运行的时候会自动生成一个ip，而我们的hyperf项目实际上是在docker容器里面跑的ip是docker 容器生成的ip，所以localhost或者127.0.0.1可能无法在hyperf项目上使用，所以我们使用本机ip
```
ipconfig
```
查询到本机的ip是192.168.29.194

使用PhpStrom打开刚刚挂载到window目录的hyperf项目

打开.env文件

修改成刚刚查看的ip
```
 DB_HOST=192.168.29.194
 REDIS_HOST=192.168.29.194
```
连接MySQL
=======

可以再docker环境下创建一个mysql容器，并将该容器挂载在window主机的端口上,因为我的主机有安装mysql3306端口被占用了，所以我将mysql映射至3305端口
```
docker run -d -p 3305:3306 --name mysql mysql /bin/bash
```
修改mysql的端口号
-----------

刚刚我们在docker容器映射的端口是3305所以我们需要修改hyperf项目的.env文件

在Windows下打开phpstorm，我们挂载的hyperf项目

打开.env文件

刚刚创建的数据库名称是users，所以这里顺便修改数据库名称
```
DB_PORT=3305
DB_DATABASE=users
```
重新启动hyperf项目
============

打开docker的可视化界面的CLI或者在终端使用
```
docker exec -it 容器名 /bin/bash
```



进入安装好的 Hyperf 项目目录
```
cd hyperf-skeleton
```
启动 Hyperf
```
php bin/hyperf.php start
```
应该可以看见正常启动了。

然后来进行一个浏览器访问

```
http://localhost:9501
```
进行MVC的一个搭建
==========

表的建立
----

由于刚刚数据库已经配置好了所以我们先在数据库进行一个数据的创建

我们创建一个users用户
```
create database users;
```
之后再users用户下创建user表
```
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
 `id` int(11) NOT NULL AUTO_INCREMENT,
 `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NULL DEFAULT NULL,
 `age` int(11) NULL DEFAULT NULL,
 `qq` int(11) NULL DEFAULT NULL,
 PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 6 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci ROW_FORMAT = Dynamic;
```
​

然后我们插入一些数据，进行测试
```
INSERT INTO `user` VALUES (1, '张三', 12, 1);
INSERT INTO `user` VALUES (2, '李四', 13, 2);
INSERT INTO `user` VALUES (3, '王五', 14, 3);
INSERT INTO `user` VALUES (4, '李世民', 15, 4);
INSERT INTO `user` VALUES (5, '朱迪', 16, 5);
```
表已经建好了，接下来我们进行MVC的搭建

创建controller
------------

### 使用 @AutoController

在我们的hyperf项目的app目录下有一个controller目录

进入到这个目录创建UserController类

hyperf框架是支持注解和依赖注入的，所以这里我们使用注解的方式进行控制层的一个使用

我们先在UserController这个类上加上注解@AutoController

@AutoController这个注解的作用是自动帮助我们生成访问地址

比如

我们的类名是UserController，我们的方法名是Index

那么我们访问的地址就是 ip地址:端口/user/index，它会自动的把类名的Controller部分去除，并将User转化为小写。方法上就是把index转化为小写。

好了现在我们先来尝试一下使用UsreController在浏览器上打印Hello！

代码示例如下
```
namespace AppController;
​
​
use HyperfHttpServerAnnotationAutoController;
​
/**
 * Class UserController
 * @AutoController
 */
class UserController
{
public function index(){
 return "hello";
}
}
```
好的我们重新启动hyperf项目

在CLI窗口按下
```
CTRL+C
```
然后重新输入
```
php bin/hyperf.php start
```

启动完成之后我们在浏览器查看一下,输入
```
http://localhost:9501/user/index
```
可以看见 hello出现在浏览器上

### 使用@Controller +@RequestMapping

同样的，我们创建一个新的类，在Controller目录下，就叫MappingController吧

同样是在类名上面添加注解

@Controller，里面有三个属性

分别是prefix前缀名，server默认http，option是一个空数组用来保存传递信息

一般我们使用prefix这个前缀就醒了

这个前缀的意思和上面那个@AutoController转化的访问地址里

[http://localhost:9501/user/index](http://localhost:9501/user/index)的user是一个意思

接下来我们编写一个方法index方法，在方法名上添加注解@RequestMapping

一般在这个注解传递你需要的访问地址名就可以了，我这里使用index

代码示例如下
```
namespace AppController;
​
use HyperfHttpServerAnnotationController;
use HyperfHttpServerAnnotationRequestMapping;
​
/**
 * Class MappingController
 * @package AppController
 * @Controller(prefix="mapping")
 */
class MappingController
{
 /**
 * @return string
 * @RequestMapping("index")
 */
public function index(){
 return "this is mappingcontroller";
}
}
```
接下来我们访问
```
http://localhost:9501/mapping/index
```
使用AST生成模型
---------

hyperf提供了从数据库生成model的脚本，可以生成对应字段的实体类

1.1.0 + 版本：
```
php bin/hyperf.php gen:model table_name
```
1.0.* 版本：
```
php bin/hyperf.php db:model table_name
```
这里我们使用1.1.0+版本的

先进入目录hyperf/skeleton

然后使用
```
php bin/hyperf.php gen:model user
```
打开我们的hyperf项目

可以看见在app/model下生成了User类

创建Service层
----------

在app下创建service目录

这里我们创建一个UserService进行一个查询，并将结果返回
```
namespace AppService;
​
use AppModelUser;
use HyperfDbConnectionDb;
​
/**
 * Class UserService
 * @package AppService
 */
class UserService
{
 
public function findUser1($id)
{
 return User::query()->where('id',$id)->first();
}
​
 public function findUser2($id): array
 {
 return Db::select('select * from user where id =?',[$id]);
 }
​
}
```
在UserController里面编写控制器
```
namespace AppController;
use AppServiceUserService;
use HyperfDiAnnotationInject;
use HyperfHttpServerAnnotationAutoController;
/**
 * Class UserController
 * @AutoController
 */
class UserController
{
 /**
 * @return string
 */
public function index(){
 return "hello";
}
​
/**
 * @var UserService
 * @Inject
 */
private $UserService;
public function findUser1()
{
 return $this->UserService->findUser1(1);
}
​
public function findUser2(): array
{
 return $this->UserService->findUser2(1);
 }
​
}
```
访问
```
http://localhost:9501/user/findUser1
http://localhost:9501/user/findUser2
```
这样一个MVC就创建完成