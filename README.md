# OIE 缓存管理器

一个 [Open Integration Engine](https://github.com/nextgenhealthcare/connect) 插件，使用 Google Guava 的 LoadingCache 和 HikariCP 连接池，提供对外部数据库的快速内存键值查找。

<p align="center">
  <img src="images/cache-inspector.png" width="700" alt="OIE 管理界面与缓存检查器">
</p>

## 功能特性

- **按需懒加载** — 数据在首次查找时从数据库获取，之后从内存中提供
- **自动淘汰** — 可配置的最大大小和基于时间的过期策略
- **HikariCP 连接池** — 每个缓存限制并发连接数，防止连接风暴
- **管理界面** — 在 OIE 管理界面中创建、编辑、复制、启用/禁用和删除缓存定义
- **缓存检查器** — 查看命中率、内存使用量、条目数量，并浏览缓存数据
- **REST API** — 完整的 CRUD 和统计端点，带有 OpenAPI 注解
- **密码加密** — 使用 OIE 内置加密器对数据库凭据进行静态加密
- **事件日志** — 缓存操作记录到 OIE 事件系统

## 系统要求

- Open Integration Engine 4.6.0
- Java 17+

## 安装

1. 从 [Releases](https://github.com/diridium-com/oie-cache-manager/releases) 页面下载最新的 `oie-cache-manager-<version>.zip`
2. 在 OIE 管理界面中，进入 **设置 > 扩展**，点击 **安装扩展**
3. 选择 ZIP 文件并重启服务器
4. 确认 **缓存管理器** 选项卡出现在 **设置** 下

## 使用方法

在管理界面中定义一个指向外部数据库的缓存，然后从任意通道中查找：

```javascript
var state = $g('zip2').lookup(zip);
```

首次查找某个键时会查询数据库。后续调用直接从内存返回。

## 权限

此插件注册了两个权限：**查看缓存**（定义、统计和缓存检查器）和 **管理缓存**（创建、编辑、删除、刷新以及连接/查询测试），涵盖其所有 REST 操作和设置选项卡。

在默认安装的 OIE 中，这不会造成任何变化：默认授权控制器允许每个已认证用户执行所有操作。在运行基于角色的授权控制器的安装中，必须将权限授予角色后，其用户才能查看或管理缓存。

> **从 1.1.0 或更早版本升级到基于角色的服务器：** 缓存操作之前使用核心服务器设置权限名称，扩展操作实际上无法匹配这些权限，因此访问遵循控制器的未知操作策略。升级后，管理员必须为每个需要此权限的角色授予查看缓存 / 管理缓存权限。

## 从源码构建

```bash
./scripts/install-engine-jars.sh
mvn clean package
```

此脚本将本插件构建所依赖的五个 OIE 引擎 jar 安装到本地 Maven 仓库中，jar 取自与 POM 的 `mc.version` 匹配的已发布 OIE 版本，并通过该版本的 `sha256sums` 校验。每个引擎版本运行一次即可。

可安装的插件 ZIP 包生成于：

```
package/target/oie-cache-manager-<version>.zip
```

如需启用 JAR 签名构建（需要 PKCS#11 令牌）：

```bash
mvn clean package -Psigning
```

## 文档

完整文档请参见 [Wiki](https://github.com/diridium-com/oie-cache-manager/wiki)：

- [Getting Started](https://github.com/diridium-com/oie-cache-manager/wiki/Getting-Started)
- [Managing Caches](https://github.com/diridium-com/oie-cache-manager/wiki/Managing-Caches)
- [Cache Inspector](https://github.com/diridium-com/oie-cache-manager/wiki/Cache-Inspector)
- [Using Caches in Channels](https://github.com/diridium-com/oie-cache-manager/wiki/Using-Caches-in-Channels)
- [REST API](https://github.com/diridium-com/oie-cache-manager/wiki/REST-API)
- [Troubleshooting](https://github.com/diridium-com/oie-cache-manager/wiki/Troubleshooting)
- [Security Considerations](https://github.com/diridium-com/oie-cache-manager/wiki/Security-Considerations)

## 许可证

[Mozilla Public License 2.0](LICENSE)
