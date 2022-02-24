在windows使用docker环境搭建hyperf微服务
==============================

创建hyperf项目环境
------------
```
docker run -d --name hyperfrpc -v d/dataproject/hyperfrpc:/hyperf-skeleton -p 9501:9501 -it --entrypoint /bin/sh hyperf/hyperf:7.4-alpine-v3.11-swoole
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

将 Composer 镜像设置为阿里云镜像，加速国内下载速度

composer config -g repo.packagist composer https://mirrors.aliyun.com/composer

通过 Composer 安装 hyperf/hyperf-skeleton 项目

composer create-project hyperf/hyperf-skeleton

进入安装好的 Hyperf 项目目录，linux目录

cd hyperf-skeleton

测试 Hyperf环境是否成功

php bin/hyperf.php start

在powshell打开一个cli

docker exec -it hyperfrpc /bin/bash

进行测试

curl 127.0.0.1:9501

看到输出

{"method":"GET","message":"Hello Hyperf."}

环境成功
```
在hyperf-skeleton目录下创建消费者和提供者
----------------------------

### 删除刚刚创建的hyperf项目

`rm -rf hyperf-skeleton`

### 创建provider

`composer create-project hyperf/hyperf-skeleton hyperf-skeleton/provider`

创建时，选项只选择一个JSON-PRC with Service Governance，其他都选n

### 创建consumer

`cd hyperf-skeleton`
# 复制provider为consumer
`cp -r provider consumer`

在windows使用编辑器编辑provider
=======================

随意使用一个编辑器然后打开项目，目录是你创建docker容器时挂载的地方，我们上面挂载了d/dataproject/hyperfrpc

我们可以在目录看到存在provider、consumer

编写provider
----------

我们在app目录下创建一个文件夹，叫Rpc，在Rpc下创建CalculatorService和CalculatorServiceInterface

### 服务端口

CalculatorServiceInterface是服务接口
```
<?php
​
namespace AppRpc;
​
interface CalculatorServiceInterface
{
 public function add(int $a, int $b);
​
 public function minus(int $a, int $b);
}
```
### 服务实现类

CalculatorService就是我们服务提供者的服务实现类,我们只需要服务实现类上添加注解@RpcService就可以将服务发布

name，给服务起个名字

protocol，使用的协议，这里一般使用jsonrpc-http

server，绑定的server
```
<?php
namespace AppRpc;
​
use HyperfRpcServerAnnotationRpcService;
​
/**
 * Class CalculatorService
 * @package AppRpc
 * @RpcService(name="CalculatorService",protocol="jsonrpc-http",server="jsonrpc-http")
 */
class CalculatorService implements CalculatorServiceInterface
{
​
 public function add(int $a,int $b)
 {
 return $a+$b;
 }
 public function minus(int $a,int $b)
 {
 return $a-$b;
 }
}
```
### 添加对应server

接下来，我们去config/autoload/server.php下，添加刚刚在注解@RpcService上写的server=“jsonrpc-http”的这个server

我们找到servers

然后发现已经存在一个name是http的server了，我们复制一个name是http的server，之后添加在http这个server的下面，然后修改一下

我们把name修改成jsonrpc-http

我们把port修改成9502

我们把callbacks修改一下，把HyperfHttpServerServer::class修改成HyperfJsonRpcHttpServer::class,如下面
```
'servers' => [
 [
 'name' => 'http',
 'type' => Server::SERVER_HTTP,
 'host' => '0.0.0.0',
 'port' => 9501,
 'sock_type' => SWOOLE_SOCK_TCP,
 'callbacks' => [
 Event::ON_REQUEST => [HyperfHttpServerServer::class, 'onRequest'],
 ],
 ],
 [
 'name' => 'jsonrpc-http',
 'type' => Server::SERVER_HTTP,
 'host' => '0.0.0.0',
 'port' => 9502,
 'sock_type' => SWOOLE_SOCK_TCP,
 'callbacks' => [
 Event::ON_REQUEST => [HyperfJsonRpcHttpServer::class, 'onRequest'],
 ],
 ],
```
编写consumer
----------

我们就在consumer项目的controller调用一下刚刚写的CalculatorService把。

### 对应接口

在调用之前我们需要把服务接口从provider哪里复制一份过来

我们在consumer的app目录下创建Rpc目录，在Rpc目录下创建刚刚复制的CalculatorServiceInterface
```
<?php
​
namespace AppRpc;
​
interface CalculatorServiceInterface
{
 public function add(int $a, int $b);
​
 public function minus(int $a, int $b);
}
```
### 使用controller调用

我们就在IndexController里面的index方法调用服务,但是现在还无法消费成功，我们还需要一些配置。
```
<?php
​
declare(strict_types=1);
/**
 * This file is part of Hyperf.
 *
 * @link     https://www.hyperf.io
 * @document https://hyperf.wiki
 * @contact  group@hyperf.io
 * @license  https://github.com/hyperf/hyperf/blob/master/LICENSE
 */
namespace AppController;
​
​
​
use HyperfDiAnnotationInject;
​
class IndexController extends AbstractController
{
 /**
 * @Inject
 * @var AppRpcCalculatorServiceInterface
 */
 private $calculatorService;
​
 public function index()
 {
 return $this->calculatorService->add(1,2);
 }
}
​
```
### 创建services的配置文件

我们需要去到/config/autoload目录下创建services.php文件，文件配置如下
```
<?php
​
​
use AppRpcCalculatorServiceInterface;
​
return [
 'consumers'=>[
 [
 'name'=>'CalculatorService',
 'service'=>CalculatorServiceInterface::class,
 'nodes'=>[
 ['host'=>'127.0.0.1','port'=>9502],
 ],
 ],
 ],
];
```
这个服务的name和我们编写注解的name是一致的

服务的service对应consumer的CalculatorServiceInterface，

由于我们没有使用注册中心，所以我们直接使用节点地址，对应上面代码中的nodes

### 修改consumer的启用端口

因为consumer复制过来的，所以我们需要修改一下端口

在/config/autoload下的server.php文件下

我们找到server，定位到name是http的server，将端口修改成9503
```
'servers' => [
 [
 'name' => 'http',
 'type' => Server::SERVER_HTTP,
 'host' => '0.0.0.0',
 'port' => 9503,
 'sock_type' => SWOOLE_SOCK_TCP,
 'callbacks' => [
 Event::ON_REQUEST => [HyperfHttpServerServer::class, 'onRequest'],
 ],
 ],
 ],
```
consumer编写完成

启动provider
----------

进入终端
```
cd hyperf-skeleton/
cd provider/
php bin/hyperf.php start
```
启动consumer
----------
```
cd hyperf-skeleton/
cd consumer/
php bin/hyperf.php start
```
发起请求
----

在打开一个终端
```
curl 127.0.0.1:9503
```
我们应该可以看到输出了
```
3
```
使用consul
========

我们这里使用的是consul

我们在终端provider目录试下安装hyperf的服务治理

`composer hyperf/service-governance`

安装consul组件

`composer require hyperf/consul`

发布一下对应的配置文件

`php bin/hyperf.php vendor:publish hyperf/consul`

我们可以看到consul的配置文件就已经生成成功了

可以到cogfig/autoload目录下的consul.php文件查看
```
return [
 'uri' => 'http://127.0.0.1:8500',
 'token' => '',
];
```
默认是到本地的8500端口

下载并启动consul服务
-------------
```
#下载并解压consul
cd /opt/
​
mkdir consul
​
chmod 777 consul
​
cd consul
​
wget https://releases.hashicorp.com/consul/1.3.0/consul_1.3.0_linux_amd64.zip
​
unzip consul_1.3.0_linux_amd64.zip
​
cp consul /usr/local/bin/
​
#检查是否安装成功
consul
​
consul version
​
#启动consul服务
consul agent -dev -ui -node=consul-dev -client=127.0.0.1
​
​
#3.再开一个终端，输入查看8500端口
curl 127.0.0.1:8500
#看见<a href="/ui/">Moved Permanently</a>.就成功了
```
### 将provider的实现类注册进consul

在注解上添加publishTo="consul"
```
<?php
​
namespace AppRpc;
​
use HyperfRpcServerAnnotationRpcService;
​
/**
 * Class CalculatorService
 * @package AppRpc
 * @RpcService(name="CalculatorService",protocol="jsonrpc-http",server="jsonrpc-http",publishTo="consul")
 */
class CalculatorService implements CalculatorServiceInterface
{
​
 public function add(int $a,int $b)
 {
 return $a+$b;
 }
 public function minus(int $a,int $b)
 {
 return $a-$b;
 }
}
```
将consumer的services修改
--------------------

这里的services是config/autoload下的
```
<?php
​
​
use AppRpcCalculatorServiceInterface;
​
return [
 'consumers'=>[
 [
 'name'=>'CalculatorService',
 'service'=>CalculatorServiceInterface::class,
 'registry'=>[
 'protocol'=>'consul',
 'address'=>'http://127.0.0.1:8500',
 ],
//            'nodes'=>[
//                ['host'=>'127.0.0.1','port'=>9502],
//            ],
 ],
 ],
];
```
registry将去consul里面取服务，我们需要先将nodes注释

### 重启provider
```
CTRL+C
php bin/hyperf.php start
```
### 重启consumer
```
CTRL+C
php bin/hyperf.php start
```
### 终端测试
```
curl 127.0.0.1:9503
#可以看到还是输出，我们已经把nodes注释了
3
```
服务熔断
====

为什么要熔断
------

分布式系统中经常会出现由于某个基础服务不可用造成整个系统不可用的情况，这种现象被称为服务雪崩效应。为了应对服务雪崩，一种常见的做法是服务降级。

比如我们需要到另外服务中查询用户列表，用户列表需要关联很多的表，查询效率较低，但平常并发量不高的时候，响应速度还说得过去。一旦并发量激增，就会导致响应速度变慢，并会使对方服务出现慢查。

在比如我们进行一些Db查询的时候，需要查询多个表，在这种情况下，平常并发量不高的时候，相应速度还可以说得过去。一旦并发量激增得时候，就可能出现服务不可用的情况，最后导致整个系统都不可用，这种现象被称为服务雪崩效应。为了应对服务雪崩，一种常见的做法是服务降级。

降级是指自己的待遇下降了，从RPC调用环节来讲，就是去访问一个本地的伪装者而不是真实的服务。

使用熔断器
-----

熔断器的使用十分简单，只需要加入 `HyperfCircuitBreakerAnnotationCircuitBreaker` 注解，就可以根据规定策略，进行熔断。

这个时候，我们只需要配置一下熔断超时时间 `timeout` 为 0.05 秒，失败计数 `failCounter` 超过 1 次后熔断，相应 `fallback` 为 `AppServiceUserService` 类的 `searchFallback` 方法。这样当响应超时并触发熔断后，就不会再请求对端的服务了，而是直接将服务降级从当前项目中返回数据，即根据 `fallback` 指定的方法来进行返回。

### 创建使用熔断器的Service

我们在服务提供者provider里的appService下创建一个UserService来进行使用,

首先，我们先在APP目录下创建目录Service

然后再Service目录下创建UserService，具体如下：
```
<?php
​
namespace AppService;
use HyperfCircuitBreakerAnnotationCircuitBreaker;
​
class UserService
{
 /**
 * @return string[]
 * @CircuitBreaker(timeout="0.05",failCounter=1,successCounter=1,fallback="AppSerViceUserService::searchFallback")
 */
 public function search()
 {
 for ($i=1;$i<=3;$i++){
 sleep(1);
 $a=$i;
 print $a."n";
 }
 return [
 'message'=>"正常",
 ];
}
 public static function searchFallback()
 {
 return [
 'message'=>"流量过大，熔断",
 ];
 }
}
```
在我们需要的服务上加上@CircuitBreaker这个注解就可以进行服务熔断了，接下来我说明一下几个参数

timeout="0.05"熔断超时时间

failCounter=1失败记数，超过次数后熔断

successCounter=1成功记数

fallback="AppSerViceUserService::searchFallback"发送熔断后调用的方法

### 在控制器中使用

我们编写一个控制器，就叫做UserController吧。

我们在UserController中注入刚刚编写的UserService，我为了方便就没有抽象成接口了

然后再UserController里面编写一个方法来调用UserService的方法

代码如下：
```
<?php
namespace AppController;
use HyperfCircuitBreakerAnnotationCircuitBreaker;
use HyperfDiAnnotationInject;
use HyperfHttpServerAnnotationAutoController;
use AppServiceUserService;
​
/**
 * Class UserController
 * @package AppController
 * @AutoController()
 */
class UserController extends AbstractController
{
 /**
 * @Inject
 * @var UserService
 */
 private $UserService;
​
 public function search()
 {
 return  $this->UserService->search();
 }
}
```
### 测试

由于我们之前在创建docker绑定了9501，所以我们在windows上可以用浏览器访问9501端口

我们打开浏览器

访问,多刷新几次就可以看见服务熔断这个返回，或者你打开2个访问窗口
`
127.0.0.1:9501/user/search
`
或者我们可以在终端上访问，不过需要多次访问
```
curl curl 127.0.0.1:9501/user/search
curl curl 127.0.0.1:9501/user/search
curl curl 127.0.0.1:9501/user/search
```
服务限流
====

限流的目的是通过对并发访问/请求进行限速或者一个时间窗口内的请求进行限速保护系统，一旦达到限制速率就可以拒绝服务（定向到错误页或告知资源没有了），排队等候（比如秒杀、评论和下单）、降级（返回兜底数据和默认数据，如商品详情页默认有货）。 下面我们将使用到令牌桶限流器。

安装
--

我们先进入provider目录
`
composer require hyperf/rate-limit
`
发布配置

`php bin/hyperf.php vendor:publish hyperf/rate-limit`

使用
--
```
<?php
​
namespace AppController;
​
use HyperfDiAopProceedingJoinPoint;
use HyperfHttpServerAnnotationController;
use HyperfHttpServerAnnotationRequestMapping;
use HyperfRateLimitAnnotationRateLimit;
​
/**
 * @Controller(prefix="rate-limit")
 * @RateLimit(limitCallback={RateLimitController::class, "limitCallback"})
 */
class RateLimitController
{
 /**
 * @RequestMapping(path="test")
 * @RateLimit(create=1, capacity=3)
 */
 public function test()
 {
 return ["QPS 1, 峰值3"];
 }
 
 public static function limitCallback(float $seconds, ProceedingJoinPoint $proceedingJoinPoint)
 {
 // $seconds 下次生成Token 的间隔, 单位为秒
 // $proceedingJoinPoint 此次请求执行的切入点
 // 可以通过调用 `$proceedingJoinPoint->process()` 继续执行或者自行处理
 echo "限流中";
 return $proceedingJoinPoint->process();
 }
}
```
create

1

每秒生成令牌数

consume

1

每次请求消耗令牌数

capacity

2

令牌桶最大容量

limitCallback

`[]`

触发限流时回调方法

waitTimeout

1

排队超时时间

参考文档：

[https://hyperf.wiki/2.1/#/zh-cn/microservice](https://hyperf.wiki/2.1/#/zh-cn/microservice)

[https://blog.csdn.net/weixin_40670060/article/details/109582821](https://blog.csdn.net/weixin_40670060/article/details/109582821)