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

- Open Integration Engine 4.7.0
- Java 17+

仍在运行 OIE 4.6.0？请使用 [oie-cache-manager 1.2.0](https://github.com/diridium-com/oie-cache-manager/releases/tag/v1.2.0)。

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

此脚本将本插件构建所依赖的五个 OIE 引擎 jar 安装到本地 Maven 仓库中，jar 取自与 POM 的 `mc.version` 匹配的已发布 OIE 版本，并通过 POM 中固定的 SHA-256 校验值（`oie.dist.sha256`）进行校验。CI 也运行同一脚本。每个引擎版本运行一次即可。

可安装的插件 ZIP 包生成于：

```
package/target/oie-cache-manager-<version>.zip
```

如需启用 JAR 签名构建（需要 PKCS#11 令牌；参见 [RELEASING.md](RELEASING.md) 第 3 步）：

```bash
read -rs PIN
mvn clean package -Psigning -Dsigning.storepass="$PIN"
unset PIN
```

### 端到端测试

`e2e/run.py` 通过引擎自带的安装器将构建好的 zip 安装到已发布的 OIE 引擎中，并在真实数据库上检验插件：全新安装与重启（表只创建一次，缺失的 `max_connections` 列只补加一次，1.3.0 及之前版本在 MySQL、PostgreSQL 和 SQL Server 上遗留的 `name` 重复索引只删除一次）、从 `e2e/upgrade-sources.txt` 中每个版本的升级（为旧引擎构建的版本先在该旧引擎上运行——旧引擎取自 `e2e/engines.txt`——随后在保持插件安装的情况下升级引擎），以及各项功能（连接/查询测试、保存/编辑/删除定义、通道的 `$g('name').lookup(key)` 调用、统计、检查器、刷新和重启）。每个缓存都通过对应数据库的驱动读取同一数据库服务器上的查找表。CI 在每次 push 和 pull request 时针对 Derby、PostgreSQL、MySQL、SQL Server（jTDS 和 Microsoft 驱动）和 Oracle 运行这些测试，引擎分别运行在 Java 17 和 Java 21 上。当某个发行版面向尚未发布的引擎时，CI 会使用 `--label-as` 在 POM 的 `mc.version` 引擎上运行：安装 zip 的副本，并将其 `plugin.xml` 中的引擎名称改为该引擎。

需要 Java 17 或更高版本、除 Derby 外每种数据库都需要 Docker，以及 zip 所面向引擎的发行 tarball（即其 `plugin.xml` 中的 `mirthVersion`）：

```bash
python3 e2e/run.py --db postgres --zip package/target/oie-cache-manager-<version>.zip \
    --tarball target/oie-dist/oie_unix_<x_y_z>.tar.gz --work /path/that/does/not/exist/yet
```

tarball 以引擎版本命名，例如 4.7.0 对应 `oie_unix_4_7_0.tar.gz`。当该引擎就是 POM 的 `mc.version` 时，`./scripts/install-engine-jars.sh` 会把其 tarball 留在 `target/oie-dist/` 中，而完整的 `mvn clean` 会将其删除。对于为尚未发布引擎构建的 zip，请使用 `mc.version` 的 tarball 并加上 `--label-as <mc.version>`，与 CI 的做法一致。

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
