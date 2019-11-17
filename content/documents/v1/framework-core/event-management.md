+++
title = "事件管理"
toc = true
type = "docs"
draft = false
date = "2018-09-19"
lastmod = "2018-09-20"
weight = 206

[menu.v1]
  parent = "framework-core"
  weight = 6
+++
在swoft我们将事件分为三大类：

- swoole server的回调事件
- swoft server的事件，基于swoole的回调处理，扩展了一些可用事件以增强自定义性
- 应用内的自定义事件管理和使用。也是我们通常了解和使用的事件管理了。

> swoole和server级别的事件监听器，应当放置在boot阶段。(即通常应放置于 `App\Boot` 空间下)

## Swoole Server 事件

- TAG: `@SwooleListener("event name")`

用注解tag `@SwooleListener("event name")` 来注册swoole的回调事件监听, 支持所有swoole官网列出来的事件回调名。 
具体请查看 `SwooleEvent::class` 以及 swoole官网。

> 请谨慎使用 `@SwooleListener`。 它是直接注册到 swoole server上的(**监听相同事件将会被覆盖**)，操作不当可能导致出现问题。

## Swoft Server 事件

- TAG: `@ServerListener("event name")`

用注解tag `@ServerListener("event name")` 来注册服务器级别的事件监听。

它是对 `@SwooleListener` 的补充扩展，除了支持swoole的事件以外，还增加了一些额外的可用事件监听。

两者的区别是：

 - SwooleListener 中一个事件的监听器只允许一个，并且是直接注册到 swoole server上的(**监听相同事件将会被覆盖**)
 - ServerListener 允许对swoole事件添加多个监听器，会逐个通知
 - ServerListener 不影响基础swoole事件的监听

### Examples:

```php
<?php
namespace App\Boot\Listener;

use Swoft\Bean\Annotation\ServerListener;
use Swoft\Bootstrap\Listeners\Interfaces\StartInterface;
use Swoft\Bootstrap\SwooleEvent;
use Swoole\Server;

/**
 * Class TestStartListener
 * @package App\Boot\Listener
 * @ServerListener(event=SwooleEvent::ON_START)
 */
class MyServerListener implements StartInterface
{
    /**
     * @var Server $server
     * */
    public function onStart(Server $server)
    {
        \output()->writeln('TestStartListener');
        var_dump('TestStartListener');
    }
}
```

## 自定义事件

基本的事件注册与触发管理

- implement the [Psr 14](https://github.com/php-fig/fig-standards/blob/master/proposed/event-dispatcher.md) - Event dispatcher
- 支持设置事件优先级
- 支持快速的事件组注册
- 支持通配符事件的监听

> 作为核心服务组件，事件管理会自动启用

```php
'eventManager'    => [
    'class'     => \Swoft\Event\EventManager::class,
],		     
```

### 注册事件监听

- 用注解tag `@Listener("event name")` 来注册用户自定义的事件监听

```php
/**
 * 应用加载事件
 *
 * @Listener(AppEvent::APPLICATION_LOADER)
 */
class ApplicationLoaderListener implements EventHandlerInterface
{
    /**
     * @param EventInterface $event      事件对象
     */
    public function handle(EventInterface $event)
    {
        // do something ....
    }
}
```

> 事件名称管理推荐放置在一个单独类的常量里面，方便管理和维护

- 触发事件

```php
\Swoft::trigger('event name', null, $arg0, $arg1);
// OR use \Swoft\App::trigger();
```

## 拓展介绍

一些关于自定义事件的拓展介绍说明

### 事件分组

除了一些特殊的事件外，在一个应用中，大多数事件是有关联的，此时我们就可以对事件进行分组，方便识别和管理使用。

- **事件分组**  推荐将相关的事件，在名称设计上进行分组

例如：

```text
// 模型相关：
model.insert
model.update
model.delete

// DB相关：
db.connect
db.disconnect
db.query

// 应用相关：
app.start
app.run
app.stop
```

### 事件通配符 `*`

支持使用事件通配符 `*` 对一组相关的事件进行监听, 分两种。

1. `*` 全局的事件通配符。直接对 `*` 添加监听器(`$em->attach('*', 'global_listener')`), 此时所有触发的事件都会被此监听器接收到。
2. `{prefix}.*` 指定分组事件的监听。eg `$em->attach('db.*', 'db_listener')`, 此时所有触发的以 `db.` 为前缀的事件(eg `db.query` `db.connect`)都会被此监听器接收到。

> 当然，你在事件到达监听器前停止了本次事件的传播`$event->stopPropagation(true);`，就不会被后面的监听器接收到了。

### 更多介绍

更多关于自定义事件的理解参考 https://github.com/inhere/php-event-manager/blob/master/README.md
