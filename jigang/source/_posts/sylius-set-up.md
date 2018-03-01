---
title: 基于Sylius搭建电商平台
date: 2018-01-03 12:53:57
tags:
---

![sylius](http://sylius.org/files/2017-06/1496869953_integrates.png)

[Sylius](http://sylius.org)是一种电子商务技术，它能让你完全自由地创造一种独特的客户体验。它与您现有的系统集成在一起，并保证快速上市。

Sylius使用PHP编程语言和[Symfony框架](http://symfony.com/)。像Drupal或eZ这样的大型的开源项目也是如此。
它使用doctrine作为ORM、数据库抽象和TWIG(PHP的安全模板引擎)。

Sylius不是一个单一的解决方案——您可以随时更改或开发它。它可以作为你的自定义电子商务平台的基础，也可以作为你自己系统的核心。

这篇文章将介绍如何搭建Sylius框架和添加ShopAPI插件，以适应移动APP的电商客户端API。

<!-- more -->

## 系统需求

首先，看一下运行Symfony的需求。

`推荐`操作系统是Unix系统 —— Linux，MacOS

### Web服务器和配置
- Apache web server ≥ 2.2
- 开发过程中，推荐方法是使用PHP内置的web服务器

### PHP需要模块和配置

#### PHP版本:

PHP	^7.1

#### PHP扩展:
- [gd](http://php.net/manual/en/book.fileinfo.php)	没有具体的配置
- [exif](http://php.net/manual/en/book.exif.php)	没有具体的配置
- [fileinfo](http://php.net/manual/en/book.fileinfo.php)	没有具体的配置
- [intl](http://php.net/manual/en/book.intl.php)	没有具体的配置

#### PHP配置设置:
- memory_limit	≥1024M
- date.timezone	Europe/Warsaw

### 数据库

- MySQL	5.x

> 当然，您可以使用任何其他RDBMS，例如PostgreSQL。

### 访问权限

大多数应用程序文件夹和文件只需要读取访问权限，但是一些文件夹也需要对Apache/Nginx用户的写访问权限:

- var/cache
- ar/logs
- web/media

## 环境搭建

为了方便，环境在macOS上选择使用MAMP Pro。

![MAMP Pro](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/mamp_pro.png)

开发过程中的Web服务器使用PHP内置的web服务器，所以主要的配置是PHP和数据库的环境。MAMP Pro上自带MySQL和phpMyAdmin.

![MAMP Pro MySQL](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/mamp_pro_mysql.png)

双击phpMyAdmin图标：

![phpMyAdmin](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/mamp_pro_mysql_phpmyadmin.png)

### PHP配置

使能相关扩展：

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/mamp_pro_phpini.png)

内存限制：

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/mamp_pro_phpini_1.png)

### 创建数据库用户和库

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/mamp_pro_mysql_phpmyadmin_user_1.png)

## 安装

本文假设您熟悉PHP的依赖管理器[Composer](http://packagist.org/)。

### 创建Sylius项目

要使用Sylius标准版创建一个新项目，请运行以下命令:

```bash
composer create-project sylius/sylius-standard sylius
```

composer开始安装：

```bash
Installing sylius/sylius-standard (v1.0.7)
  - Installing sylius/sylius-standard (v1.0.7): Downloading (100%)         
Created project in sylius
Loading composer repositories with package information
Installing dependencies (including require-dev) from lock file
Warning: The lock file is not up to date with the latest changes in composer.json. You may be getting outdated dependencies. Run update to update them.
Package operations: 151 installs, 0 updates, 0 removals
  - Installing ocramius/package-versions (1.2.0): Downloading (100%)         
  - Installing behat/transliterator (v1.2.0): Loading from cache
  - Installing doctrine/lexer (v1.0.1): Loading from cache
... ...

```

> 如果是第一次安装可以下载需要一段时间，依赖了很多的库。

依赖下载完成之后，安装过程会构建参数，需要填写信息：

```bash
> Incenteev\ParameterHandler\ScriptHandler::buildParameters
Creating the "app/config/parameters.yml" file
Some parameters are missing. Please provide them.
database_driver (pdo_mysql):
database_host (127.0.0.1):
database_port (null): 8889
database_name (sylius): sylius
database_user (root): sylius
database_password (null): sylius
mailer_transport (smtp):
mailer_host (127.0.0.1):
mailer_user (null): sylius
mailer_password (null): sylius
secret (EDITME):

```

以上信息可以看出，创建了一个`app/config/parameters.yml`文件。设置了一些配置参数，这些参数也可以在文件中更改。

最后会清除缓存和安装资源：

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/1514954759643.jpg)



Sylius项目新建成功！！

新建完成Sylius项目后，会出现一个sylius文件夹，里面是整个项目工程。

```bash
$ tree -L 1
.
├── Gulpfile.js
├── LICENSE
├── README.md
├── app
├── behat.yml.dist
├── bin
├── composer.json
├── composer.lock
├── etc
├── features
├── package.json
├── src
├── var
├── vendor
├── web
└── yarn.lock
```

标准的Symfony项目目录结构。Symfony和Sylius的源码都在vendor目录级下。

### 安装Sylius

```bash
$ cd sylius # Move to the newly created directory
$ php bin/console sylius:install
```

安装过程主要是:
- 检查系统需求
- 建立数据库。创建Sylius相关的数据表和一些测试的资源数据(比如产品/顾客用户等)。
> 所以安装过程请确保数据库能正常连接。

- 创建媒体文件夹 'web/media','web/media/image'

- 选择货币
- 添加语言环境en_US
- 创建您的管理员帐户

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/sylius_install.png)

### 安装assets

为了看到一个功能齐全的前端，您需要安装它的assets.

Yarn安装到您的项目目录并运行:

```bash
$ yarn install
```

现在您可以使用gulp来安装views，只需运行一个简单的命令:

```bash
$ yarn run gulp
```

### 访问商店

建议使用Symfony内置web服务器运行`php bin/console server:start 127.0.0.1:8000`命令，
然后在web浏览器中访问http://127.0.0.1:8000，以查看该商店。

![shop](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/sylius_shop.png)

### 访问管理面板

查看/admin url，您将在其中找到管理面板。

> 请记住，您必须使用安装Sylius时提供的凭证作为管理员登录。

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/sylius_Dashboard.png)

#### 修改管理面板为中文

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/sylius_Dashboard_acctout.png)

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/sylius_Dashboard_acctout_zh.png)

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/sylius_Dashboard_acctout_zh_ok.png)

## 项目结构简介

在项目的根目录中，您将找到这些重要的子目录:

- `app/config/` - 添加yaml配置文件，包括路由、安全、状态机配置等
- `var/logs/` - 应用程序的日志
- `var/cache/` - 项目的缓存
- `src/` - `AppBundle`中添加所有自定义逻辑的地方
- `web/` - 放置你的项目的资产

## bin/console 命令列表

```bash
$ php bin/console list
Symfony 3.4.2 (kernel: app, env: dev, debug: true)

Usage:
  command [options] [arguments]

Options:
  -h, --help            显示帮助信息
  -q, --quiet           不要输出任何消息
  -V, --version         显示这个应用程序版本
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  不要问任何互动的问题
  -e, --env=ENV         环境名称[默认:“dev”]
      --no-debug        关闭调试模式
  -v|vv|vvv, --verbose  增加消息的冗余:1用于正常输出，2用于更详细的输出，3用于调试。

Available commands:
  about                                   显示当前项目的信息
  help                                    显示命令的帮助
  list                                    命令列表
 assets
  assets:install                          在公共目录下安装bundle web资产
 cache
  cache:clear                             清空缓存
  cache:pool:clear                        清除缓存池
  cache:pool:prune                        修剪缓存池
  cache:warmup                            预热空缓存
 config
  config:dump-reference                   为扩展转储默认配置
 debug
  debug:autowiring                        列出可以用于自动装配的类/接口
  debug:config                            转储当前配置
  debug:container                         显示应用程序的当前服务
  debug:event-dispatcher                  为应用程序显示已配置的侦听器
  debug:form                              显示表单类型信息
  debug:payum:gateway                     [payum:gateway:debug]
  debug:router                            显示应用程序的当前路由
  debug:swiftmailer                       显示应用程序的当前邮件
  debug:translation                       显示消息翻译信息
  debug:twig                              显示一组分支函数、筛选器、全局变量和测试
  debug:winzou:state-machine              
 doctrine
  doctrine:cache:clear-collection-region  清除二级缓存收集区域.
  doctrine:cache:clear-entity-region      清除二级缓存实体区域.
  doctrine:cache:clear-metadata           为一个实体管理器清除所有的元数据缓存
  doctrine:cache:clear-query              为一个实体管理器清除所有的查询缓存
  doctrine:cache:clear-query-region       清除二级缓存查询区域.
  doctrine:cache:clear-result             为实体管理器清除结果缓存
  doctrine:cache:contains                 检查是否存在一个高速缓存条目
  doctrine:cache:delete                   删除缓存条目
  doctrine:cache:flush                    [doctrine:cache:clear] 刷新一个给定的缓存
  doctrine:cache:stats                    获取给定缓存提供者的统计信息
  doctrine:database:create                创建配置的数据库
  doctrine:database:drop                  Drops the configured database
  doctrine:database:import                将SQL文件直接导入数据库.
  doctrine:ensure-production-settings     验证该原则是否为生产环境配置了适当的配置.
  doctrine:fixtures:load                  将数据固定到数据库中.
  doctrine:generate:entities              [generate:doctrine:entities] 从映射信息中生成实体类和方法存根
  doctrine:mapping:convert                [orm:convert:mapping] 转换支持格式之间的映射信息.
  doctrine:mapping:import                 从现有数据库导入映射信息
  doctrine:mapping:info                   
  doctrine:migrations:diff                通过将当前数据库与映射信息进行比较，生成迁移.
  doctrine:migrations:execute             手动执行单个迁移版本.
  doctrine:migrations:generate            生成一个空白的迁移类.
  doctrine:migrations:latest              输出最新版本号
  doctrine:migrations:migrate             执行迁移到指定的版本或最新的可用版本.
  doctrine:migrations:status              查看一组迁移的状态.
  doctrine:migrations:version             手动添加和删除版本表中的迁移版本.
  doctrine:query:dql                      从命令行直接执行任意DQL.
  doctrine:schema:create                  执行(或转储)生成数据库模式所需的SQL
  doctrine:schema:drop                    执行(或转储)删除当前数据库模式所需的SQL
  doctrine:schema:update                  执行(或转储)更新数据库模式所需的SQL，以匹配当前的映射元数据.
  doctrine:schema:validate                验证映射文件.
 fos
  fos:oauth-server:clean                  清除到期的令牌
 gaufrette
  gaufrette:filesystem:keys               列出文件系统的所有文件键
 liip
  liip:imagine:cache:remove               删除用于传递的资产路径和过滤器名称的资产缓存
  liip:imagine:cache:resolve              为已传递的资产路径和过滤集名称解析资产缓存
 lint
  lint:twig                               Lints a template and outputs encountered errors
  lint:xliff                              Lints a XLIFF file and outputs encountered errors
  lint:yaml                               Lints a file and outputs encountered errors
 payum
  payum:security:create-capture-token     
  payum:security:create-notify-token      
  payum:status                            允许获得付款状态.
 router
  router:match                            通过模拟路径信息匹配来帮助调试路由
 security
  security:check                          检查项目依赖项中的安全性问题
  security:encode-password                编码一个密码.
 server
  server:run                              运行一个本地web服务器
  server:start                            在后台启动一个本地web服务器
  server:status                           为给定地址输出本地web服务器的状态
  server:stop                             停止使用`server:start` 命令启动的本地web服务器
 sonata
  sonata:block:debug                      调试所有可用的块，显示每个块的默认设置
  sonata:core:dump-doctrine-metadata      获取当前教条模式的信息
 swiftmailer
  swiftmailer:email:send                  发送简单的电子邮件消息
  swiftmailer:spool:send                  从线轴发送电子邮件
 sylius
  sylius:cancel-unpaid-orders             删除在配置期间未支付的订单。配置参数——sylius_order.order_expiration_period.
  sylius:debug:resource                   调试资源元数据.
  sylius:fixtures:list                    列出可用的fixtures
  sylius:fixtures:load                    从给定的套件加载fixtures
  sylius:install                          在您的首选环境中安装Sylius.
  sylius:install:assets                   安装所有Sylius资产.
  sylius:install:check-requirements       检查所有的Sylius需求是否满足.
  sylius:install:database                 安装Sylius数据库.
  sylius:install:sample-data              将样本数据安装到Sylius.
  sylius:install:setup                    Sylius配置设置.
  sylius:oauth-server:create-client       创建一个新客户
  sylius:remove-expired-carts             删除在配置期间空闲的购物车。配置参数——sylius_order.cart_expires_after.
  sylius:theme:assets:install             在公共web目录下安装主题web资产
  sylius:theme:list                       显示被检测到的主题的列表.
  sylius:user:demote                      通过删除角色来演示用户.
  sylius:user:promote                     通过添加角色来提升用户.
 translation
  translation:update                      更新的翻译文件
```

## 安装ShopAPI插件

![shop-api](http://sylius.org/files/2017-07/shop-api.png)

[ShopAPI插件](https://github.com/Sylius/SyliusShopApiPlugin)提供了Sylius电子商务平台上的ShopApi实现。

### 使用

#### 安装插件包

修改`composer.json`文件,在`require`节点中添加一行：

`"sylius/shop-api-plugin": "^1.0.0@beta"`

然后，执行：

```bash
$ composer.phar update
```

输出结果为：

![](http://oxwfu3w0v.bkt.clouddn.com/sylius/setup/shop-api/update.png)

从输出信息中可以看出：
- 安装了`sylius/shop-api-plugin (v1.0.0-beta.20)`
- 安装了依赖库:
  - 'league/tactician'
  - 'league/tactician-doctrine'
  - 'league/tactician-bundle'
- 更新了一些依赖包

#### 扩展配置文件

##### 添加`SyliusShopApi`到`AppKernel`

  ```PHP
  // app/AppKernel.php

    /**
     * {@inheritdoc}
     */
    public function registerBundles()
    {
        $bundles = [
            // ...

            new \Sylius\ShopApiPlugin\ShopApiPlugin(),
            new \League\Tactician\Bundle\TacticianBundle(),
        ];

        return array_merge(parent::registerBundles(), $bundles);
    }
  ```
##### `app/config/config.yml`

- 添加 - `{ path: '^/shop-api', priorities: ['json'], fallback_format: json, prefer_extension: true }` 到 `fos_rest.format_listener.rules`节
- 导入插件配置

```yml
# app/config/config.yml

imports:
    # ...
    - { resource: "@ShopApiPlugin/Resources/config/app/config.yml" }

# ...

fos_rest:
    # ...

    format_listener:
        rules:
            - { path: '^/shop-api', priorities: ['json'], fallback_format: json, prefer_extension: true } # <-- Add this
            - { path: '^/api', priorities: ['json', 'xml'], fallback_format: json, prefer_extension: true }
            - { path: '^/', stop: true }
```

##### 添加路由到 `app/config/routing.yml`

```yml
# app/config/routing.yml

# ...

sylius_shop_api:
    resource: "@ShopApiPlugin/Resources/config/routing.yml"
```


##### 配置防火墙

- 改变`sylius.security.shop_regex`参数也包含shop-api前缀
- 添加ShopAPI正则表达式参数`shop_api.security.regex: "^/shop-api”`
- 添加ShopAPI防火墙配置:

```yml
parameters:
    # ...

    sylius.security.shop_regex: "^/(?!admin|api|shop-api)[^/]++" # shop-api has been added inside the brackets
    shop_api.security.regex: "^/shop-api"

# ...

security:
    firewalls:
        // ...

        shop_api:
            pattern: "%shop_api.security.regex%"
            stateless:  true
            anonymous:  true
```

##### 调整签出配置

避免与Sylius shop API发生冲突。例如(假设，您正在使用常规的Sylius安全定义):

```yml
# app/config/config.yml

# ...

sylius_shop:
    checkout_resolver:
        pattern: "%sylius.security.shop_regex%/checkout/"
```

##### `(可选)`跨源Ajax请求

如果您已经安装了`nelmio/NelmioCorsBundle`以支持跨源Ajax请求

- 将`NelmioCorsBundle`添加到`AppKernel`中

```PHP
// app/AppKernel.php

/**
 * {@inheritdoc}
 */
public function registerBundles()
{
    $bundles = array(
        // ...
        new Nelmio\CorsBundle\NelmioCorsBundle(),
        // ...
    );
    // ...
}
```

- 将配置添加到`config.yml`中

```yml
# app/config/config.yml

# ...

nelmio_cors:
    defaults:
        allow_credentials: false
        allow_origin: []
        allow_headers: []
        allow_methods: []
        expose_headers: []
        max_age: 0
        hosts: []
        origin_regex: false
        forced_allow_origin_value: ~
    paths:
        '^/shop-api/':
            allow_origin: ['*']
            allow_headers: ['Content-Type', 'authorization']
            allow_methods: ['POST', 'PUT', 'GET', 'DELETE', 'OPTIONS']
            max_age: 3600
```

### 附加功能

#### 属性

如果您想要接收序列化的属性，您需要在`shop_api.included_attributes`键下定义一个他们的代码数组。如:

```yml
shop_api:
    included_attributes:
        - "MUG_MATERIAL_CODE"
```

#### 授权

默认情况下，不提供此bundle的授权。
但是为了样例配置检查，它被测试与[LexikJWTAuthenticationBundle](https://github.com/lexik/LexikJWTAuthenticationBundle)一起工作

- [security.yml](https://github.com/Sylius/SyliusShopApiPlugin/blob/master/tests/Application/app/config/security.yml)

```yml
# app/config/security.yml

parameters:
    ...
    shop_api.security.regex: "^/shop-api"

security:

    ...

    firewalls:

        ...

        shop_api_login:
            pattern:  "%shop_api.security.regex%/login"
            stateless: true
            anonymous: true
            form_login:
                provider: sylius_shop_user_provider
                login_path: shop_api_login_check
                check_path: shop_api_login_check
                success_handler: lexik_jwt_authentication.handler.authentication_success
                failure_handler: lexik_jwt_authentication.handler.authentication_failure
                require_previous_session: false

        shop_api:
            pattern: "%shop_api.security.regex%"
            stateless:  true
            anonymous:  true
            guard:
                authenticators:
                    - lexik_jwt_authentication.jwt_token_authenticator

        ... ...

    access_control:
        ...
        - { path: "%shop_api.security.regex%/login", role: IS_AUTHENTICATED_ANONYMOUSLY }

        - { path: "%shop_api.security.regex%/register", role: IS_AUTHENTICATED_ANONYMOUSLY }


```

- 在config.yml中的[jwt parameters](https://github.com/Sylius/SyliusShopApiPlugin/blob/master/tests/Application/app/config/config.yml#L4-L7)和[jwt配置](https://github.com/Sylius/SyliusShopApiPlugin/blob/master/tests/Application/app/config/config.yml#L55-L59)

```yml
parameters:
    secret: "Heron is the best animal in the world!"
    jwt_private_key_path: '%kernel.root_dir%/config/jwt/private-test.pem'
    jwt_public_key_path: '%kernel.root_dir%/config/jwt/public-test.pem'
    jwt_key_pass_phrase: 'heron'
    jwt_token_ttl: 3600

lexik_jwt_authentication:
    private_key_path: '%jwt_private_key_path%'
    public_key_path:  '%jwt_public_key_path%'
    pass_phrase:      '%jwt_key_pass_phrase%'
    token_ttl:        '%jwt_token_ttl%'
```

- [rsa keys例子](https://github.com/Sylius/SyliusShopApiPlugin/tree/master/tests/Application/app/config/jwt)


### 测试

应用程序可以通过API测试用例进行测试。为了运行测试套件，执行以下命令:

```bash
$ bin/phpunit
```

### Shop APIs

```bash
$ bin/console debug:router | grep shop-api
  shop_api_pickup_cart                                   POST            ANY      ANY    /shop-api/carts/{token}                                                   
  shop_api_cart_summary                                  GET             ANY      ANY    /shop-api/carts/{token}                                                   
  shop_api_add_to_cart                                   POST            ANY      ANY    /shop-api/carts/{token}/items                                             
  shop_api_add_multiple_items_to_cart                    POST            ANY      ANY    /shop-api/carts/{token}/multiple-items                                    
  shop_api_drop_cart                                     DELETE          ANY      ANY    /shop-api/carts/{token}                                                   
  shop_api_change_item_quantity                          PUT             ANY      ANY    /shop-api/carts/{token}/items/{id}                                        
  shop_api_remove_item_from_cart                         DELETE          ANY      ANY    /shop-api/carts/{token}/items/{id}                                        
  shop_api_estimated_shipping_cost                       GET             ANY      ANY    /shop-api/carts/{token}/estimated-shipping-cost                           
  shop_api_add_coupon_to_cart                            PUT             ANY      ANY    /shop-api/carts/{token}/coupon                                            
  shop_api_remove_coupon_to_cart                         DELETE          ANY      ANY    /shop-api/carts/{token}/coupon                                            
  shop_api_product_show_details_by_slug                  GET             ANY      ANY    /shop-api/products-by-slug/{slug}                                         
  shop_api_add_product_review_by_slug                    POST            ANY      ANY    /shop-api/product-reviews-by-slug/{slug}                                  
  shop_api_product_show_catalog_by_slug                  GET             ANY      ANY    /shop-api/taxon-products-by-slug/{taxonSlug}                              
  shop_api_product_show_reviews_by_slug                  GET             ANY      ANY    /shop-api/product-reviews-by-slug/{slug}                                  
  shop_api_product_show_details_by_code                  GET             ANY      ANY    /shop-api/products/{code}                                                 
  shop_api_product_show_catalog_by_code                  GET             ANY      ANY    /shop-api/taxon-products/{code}                                           
  shop_api_product_show_reviews_by_code                  GET             ANY      ANY    /shop-api/products/{code}/reviews                                         
  shop_api_add_product_review_by_code                    POST            ANY      ANY    /shop-api/products/{code}/reviews                                         
  shop_api_taxon_show_details                            GET             ANY      ANY    /shop-api/taxons/{code}                                                   
  shop_api_taxon_show_tree                               GET             ANY      ANY    /shop-api/taxons/                                                         
  sylius_shop_api_register                               POST            ANY      ANY    /shop-api/register                                                        
  sylius_shop_api_resend_verification_token              POST            ANY      ANY    /shop-api/resend-verification-link                                        
  sylius_shop_api_user_verification                      PUT             ANY      ANY    /shop-api/verify-account                                                  
  sylius_shop_api_reset_password                         PUT             ANY      ANY    /shop-api/request-password-reset                                          
  sylius_shop_password_reset                             PUT             ANY      ANY    /shop-api/password-reset/{token}                                          
  shop_api_address_checkout                              PUT             ANY      ANY    /shop-api/checkout/{token}/address                                        
  shop_api_summarize_checkout                            GET             ANY      ANY    /shop-api/checkout/{token}                                                
  shop_api_available_shipping_checkout                   GET             ANY      ANY    /shop-api/checkout/{token}/shipping                                       
  shop_api_choose_shipping_method                        PUT             ANY      ANY    /shop-api/checkout/{token}/shipping/{shippingId}                          
  shop_api_available_payment_methods_checkout            GET             ANY      ANY    /shop-api/checkout/{token}/payment                                        
  shop_api_choose_payment_method_checkout                PUT             ANY      ANY    /shop-api/checkout/{token}/payment/{paymentId}                            
  shop_api_complete_checkout                             PUT             ANY      ANY    /shop-api/checkout/{token}/complete                                       
  shop_api_me                                            GET             ANY      ANY    /shop-api/me                                                              
  shop_api_login_check                                   POST            ANY      ANY    /shop-api/login_check                                                      
```
