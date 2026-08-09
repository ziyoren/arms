# ziyoren/arms

Webman ARMS —— 阿里云应用监控链路追踪组件，基于 [webman/arms](https://github.com/webman/arms) 二次开发。

将 Zipkin 链路追踪接入阿里云应用监控（ARMS），上报应用的接口调用、SQL 执行、入参/响应等链路信息，实现跨服务调用链追踪。

## 与 webman/arms 的关系

本项目是 [webman/arms](https://github.com/webman/arms) 的独立维护分支，在其基础上针对实际生产场景进行了二次开发，主要包括：

- **B3 跨服务链路传播**：解析网关注入的 `X-B3-*` 请求头，以 `newChild()` 续接上游链路，实现多服务间的完整调用链串联（网关注入 B3 头 + 服务端续接）
- **请求头记录与脱敏**：新增 `Request:Headers` span 记录请求头，支持配置敏感字段（如 `authorization`、`cookie`）脱敏后入库
- **配置项防御性读取**：记录开关改为 `?? false` 防御性读取，兼容缺键的旧版配置
- **SQL 链路绑定实际参数**：SQL span 记录最终执行的绑定参数，便于问题定位

## 鸣谢

感谢 [webman/arms](https://github.com/webman/arms) 原作者 [@walkor](https://github.com/walkor) 及所有贡献者（[@Tinywan](https://github.com/Tinywan) [@adebug](https://github.com/adebug)等），本项目基于其代码二次开发，并保持 MIT 协议开源。

## 安装

```bash
composer require ziyoren/arms
```

安装后配置自动复制到 `config/plugin/ziyoren/arms/` 目录（中间件已默认注册，全站生效）。

## 配置

修改 `config/plugin/ziyoren/arms/app.php`：

```php
return [
    'enable' => true,
    'app_name' => '你的应用名称', // 应用名称，将显示在 ARMS 链路中
    'endpoint_url' => '接入点url', // 进入后台 https://tracing.console.aliyun.com/ 获取
    'time_interval' => 30, // 30秒上报一次，尽量将上报对业务的影响减少到最低
    'enable_request_params' => true, // 是否记录入参，json格式呈现
    'enable_response_body' => true, // 是否记录响应内容，如果存在响应数据太大或二进制，不建议开启
    'enable_request_headers' => true, // 是否记录请求头header，json格式呈现
    'mask_headers' => ['authorization', 'cookie'], // 记录请求头时脱敏的header名（小写）
];
```

中间件类：`Ziyoren\Arms\Arms`。

## 对接 Zipkin（本地调试）

本组件基于 Zipkin v2 协议上报，`endpoint_url` 可直接指向任意 Zipkin 服务端，便于本地联调。使用 Docker 启动 Zipkin：

```bash
docker run -d -p 9411:9411 openzipkin/zipkin
```

服务启动后，将配置中的 `endpoint_url` 指向 Zipkin 上报端点：

```php
'endpoint_url' => 'http://your_host:9411/api/v2/spans',
```

打开 `http://your_host:9411/zipkin/` 即可在 Zipkin UI 中查询并查看链路详情。

> 提示：容器内访问宿主机 Zipkin 时，将 `your_host` 替换为 `host.docker.internal`。

## 跨服务链路

网关需向下游注入 B3 请求头（`X-B3-TraceId`、`X-B3-SpanId`、`X-B3-Sampled`），本组件会自动识别并续接上游链路；无 B3 头时自动创建新链路，业务不受影响。

## 协议

[MIT](LICENSE)

## 链接
- [阿里云应用监控](https://help.aliyun.com/document_detail/101872.html)
- [Zipkin](https://zipkin.io/)
- [webman/arms](https://github.com/webman/arms)
- [webman arms 文档](https://www.workerman.net/plugin/3)
- [webman](https://github.com/webman-php/webman)