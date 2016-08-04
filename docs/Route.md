# Route 路由
## 传统路由
路由是框架最核心的一个模块,负责将请求转发给不同的处理器。
传统的路由,大多是简单根据定制好的规则进行请求分发;下面这个示例实现了从URL`http://domain/Controller/action`到Controller::action方法
到路由:
```php
class Route
{
    public function parse($path) {
        $path = trim($path);
        $info = explode("/", $path);
        if(count($info) != 2) {
            throw new InvalidArgumentException("Unexpected path format");
        }

        $result = array(
            'controller' => $info[0],
            'action' => $info[1]
        );
        return $result;
    }

    public function execute(array $router) {
        $controller = $router['controller'];
        $action = $router['action'];
        if(!class_exists($controller)) {
            throw new RuntimeException("controller not found");
        }
        $obj = new $controller();
        if(!method_exists($obj, $action)) {
            throw new RuntimeException("action not found");
        }
        return call_user_func(array($obj, $action));
    }
}

$route = new Route();
$router = $route->parse("/Index/Hello");
$route->execute($router);
```

# 规则路由
传统路由在几个方面存在弊端:
+ 无法对路由规则进行灵活到修改,例如对一个URL到入口进行修改,只能修改源代码
+ 路由规则比较难配置化,难以抽离出来存储在配置文件或数据库中
+ 路由与权限到控制比较难结合起来,往往需要设计成强耦合到方式实现
+ 定制化到规则不够灵活,例如根据参数进行路由
类似`fastroute`(laravel使用的路由器)可灵活配置路由到方式,已经被大多数框架所采用,`fastroute`的示例代码如下:
```php
$dispatcher = FastRoute\simpleDispatcher(function(FastRoute\RouteCollector $r) {
    $r->addRoute('GET', '/users', 'get_all_users_handler');
    // {id} must be a number (\d+)
    $r->addRoute('GET', '/user/{id:\d+}', 'get_user_handler');
    // The /{title} suffix is optional
    $r->addRoute('GET', '/articles/{id:\d+}[/{title}]', 'get_article_handler');
});

// Fetch method and URI from somewhere
$httpMethod = $_SERVER['REQUEST_METHOD'];
$uri = $_SERVER['REQUEST_URI'];

// Strip query string (?foo=bar) and decode URI
if (false !== $pos = strpos($uri, '?')) {
    $uri = substr($uri, 0, $pos);
}
$uri = rawurldecode($uri);

$routeInfo = $dispatcher->dispatch($httpMethod, $uri);
switch ($routeInfo[0]) {
    case FastRoute\Dispatcher::NOT_FOUND:
        // ... 404 Not Found
        break;
    case FastRoute\Dispatcher::METHOD_NOT_ALLOWED:
        $allowedMethods = $routeInfo[1];
        // ... 405 Method Not Allowed
        break;
    case FastRoute\Dispatcher::FOUND:
        $handler = $routeInfo[1];
        $vars = $routeInfo[2];
        // ... call $handler with $vars
        break;
}
```

在我们的项目中,决定采用`fastroute`作为路由器。路由在框架中作为配置文件单独存放,这里先在composer.json中引入`fastroute`:
```json
{
  "name": "jenner/simple_framework_based_on_composer",
  "license": "MIT",
  "authors": [
    {
      "name": "Jenner",
      "email": "hypxm@qq.com",
      "homepage" : "http://www.huyanping.cn"
    }
  ],
  "require": {
    "php": ">=5.4.0",
    "monolog/monolog": "1.17.1",
    "fastroute": "v1.0.1"
  },
  "autoload": {
    "psr-4": {
      "Jenner\\SimpleFramework\\": "src/"
    }
  }
}
```
执行`composer update`