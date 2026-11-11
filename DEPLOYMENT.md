# Fast 社区论坛系统 · 部署手册

> 适用版本：fastapi.sh **v4.5**（2026-10-02） · 程序 **Spring Boot 3.5 / JDK 17**
> 本文档为完整部署手册，快速开始请见 [README.md](./README.md)。

---

## 目录

- [一、项目简介](#一项目简介)
- [二、服务器配置要求](#二服务器配置要求)
- [三、前置要求](#三前置要求)
- [四、一键部署](#四一键部署)
- [五、fastapi.sh 命令参考](#五fastapish-命令参考)
- [六、环境变量配置](#六环境变量配置)
- [七、守护配置](#七守护配置)
- [八、更新与回滚](#八更新与回滚)
- [九、目录结构](#九目录结构)
- [十、API 对接须知](#十api-对接须知)
- [十一、更新日志](#十一更新日志)
- [十二、相关链接](#十二相关链接)

---

## 一、项目简介

Fast 社区论坛系统（FastApi）是一套基于 Spring Boot 的社区论坛后端，提供认证、内容、评论、用户、上传、支付、系统管理等 **309 个接口**，覆盖完整社区业务。

- **技术栈**：Spring Boot 3.5、JDK 17（Eclipse Temurin）、MySQL、Redis、FFmpeg（视频封面）
- **管理脚本**：fastapi.sh（v4.5），自动识别 CentOS / RHEL / Rocky / Alma / Ubuntu / Debian
- **支持架构**：x86_64 / aarch64
- **部署方式**：一键安装、systemd / crontab 守护、自动备份回滚

---

## 二、服务器配置要求

| 项目 | 最低配置 | 推荐配置 |
| --- | --- | --- |
| CPU / 内存 | 1 核 CPU / 700M 内存 | 2 核 CPU / 2G 内存 |
| 支持系统 | CentOS 7+、Ubuntu、Debian（建议安装宝塔面板），Windows 也支持 | |
| 架构 | x86_64 | x86_64 / aarch64 |

---

## 三、前置要求

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Typecho | 1.0 / 1.1 / 1.2 | 可选 |
| Redis | 5.0 - 7.0 | 推荐 6.0 |
| MySQL | 5.5 - 5.7 | 推荐 5.6 |
| 域名 A | — | 访问 FastApi（必需） |
| 域名 B | — | 访问后台管理（必需） |

> ⚠️ **重要提醒**
> 请不要在服务器自行安装 Java 和 Tomcat！脚本已内置 JDK 17，自行安装会导致端口冲突。

---

## 四、一键部署

按照 **1 → 2 → 3** 三步完成部署，全程约 **10 分钟**即可完成 API 搭建。

### 步骤 1 / 4 · 前期准备

1. 准备一台满足配置要求的服务器（建议安装宝塔面板）。
2. 安装 MySQL 与 Redis（MySQL 与 Typecho 保持一致）。
3. 准备两个域名：
   - **域名 A**：访问 FastApi
   - **域名 B**：访问后台管理
4. 确保安装端口（默认 8080）未被占用。

### 步骤 2 / 4 · 执行安装脚本

在终端执行下方 FastApi 安装脚本（支持 CentOS / Ubuntu / Debian）：

```bash
cd /opt && (
  wget -O fastapi.sh https://shell.358583.xyz/fastapi.sh 2>/dev/null \
  || curl -o fastapi.sh https://shell.358583.xyz/fastapi.sh
) && chmod +x fastapi.sh && ./fastapi.sh install
```

按提示依次输入配置：

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| 运行端口 | 8080 | 千万别用 80 或 443 |
| 数据库名 | typecho | 与 Typecho 保持一致 |
| 数据库用户名 | root | 与 Typecho 保持一致 |
| 数据库密码 | （空） | 与 Typecho 保持一致 |
| Redis 密码 | （空） | 一般是空 |
| 系统管理 KEY | 自动生成 | 也叫「访问 Key」，留空自动生成 32 位随机值 |

### 步骤 3 / 4 · 外网访问配置

以宝塔面板为例：新建一个纯静态网站，配置 SSL 证书，将下方代码放入伪静态配置：

```nginx
location ^~ / {

    if ( $request_method = 'OPTIONS' ) {
        return 204;
    }

    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header REMOTE-HOST $remote_addr;
}
```

- 将上方 `8080` 替换为第二步设置的 API 端口。
- 访问域名，key 登录，点击「安装及更新」完成安装。
- 如果未安装 Typecho，将弹出提示询问是否需要独立安装。

### 步骤 4 / 4 · 安装完成

🎉 恭喜！FastApi 安装完成。

你使用的域名即为 **API 接口地址**，可直接接入你的应用。

fastapi.sh 常用命令：

```bash
sh /opt/fastapi.sh install    # 安装
sh /opt/fastapi.sh start      # 启动
sh /opt/fastapi.sh stop       # 停止
sh /opt/fastapi.sh restart    # 重启
sh /opt/fastapi.sh update     # 更新
sh /opt/fastapi.sh status     # 状态
sh /opt/fastapi.sh monitor    # 守护
```

---

## 五、fastapi.sh 命令参考

| 命令 | 说明 | 详细 |
| --- | --- | --- |
| `install` | 首次安装 | JDK 17 + FFmpeg + 程序 + 配置 |
| `start` | 启动服务 | 启动 FastApi 进程 |
| `stop` | 停止服务 | 优雅停止进程 |
| `restart` | 重启服务 | 先停止再启动 |
| `status` | 查看状态 | 运行状态与资源占用 |
| `ffmpeg` | FFmpeg | 安装/检测视频封面依赖 |
| `logs` | 查看日志 | `logs -f` 实时 / `logs 200` 指定行数 |
| `monitor` | 守护检测 | 单次检测，crontab 用（带并发锁） |
| `update` | 更新正式版 | 自动备份，失败回滚 |
| `updateBeta` | 更新 Beta | 更新到 Beta 版 |
| `service install` | 安装服务 | systemd 开机自启 |
| `service remove` | 移除服务 | 移除 systemd |
| `env` | 查看配置 | 显示当前环境变量 |
| `migrate` | 迁移布局 | v3.x 旧布局 → /opt/fastapi |
| `dump` | 故障诊断 | 输出完整诊断信息 |
| `uninstall` | 卸载 | 保留备份可选 |
| `version` | 版本 | 显示脚本版本 |
| `help` | 帮助 | 显示帮助信息 |

---

## 六、环境变量配置

优先级：**环境变量 > 配置文件 > 默认值**

| 变量名 | 默认值 | 说明 |
| --- | --- | --- |
| `FASTAPI_HOME` | `/opt/fastapi` | 安装目录 |
| `JVM_OPT` | `-Xms512m -Xmx1024m` | JVM 参数 |
| `START_TIMEOUT` | `180` | 启动就绪等待上限（秒） |
| `STOP_TIMEOUT` | `20` | 停止等待超时（秒） |
| `LOG_MAX_MB` | `50` | app.log 切割阈值（MB） |
| `LOG_KEEP` | `5` | 日志归档保留数量 |
| `LOG_CLEAR_ON_START` | `1` | 启动时清空 app.log |
| `JDK_URL` | （自动探测） | 强制指定 JDK 下载地址 |
| `ASSUME_YES` | `0` | 自动确认，不询问 |
| `USE_SYSTEMD` | `auto` | 是否使用 systemd |

使用示例：

```bash
# 临时覆盖（仅本次生效）
START_TIMEOUT=300 sh /opt/fastapi.sh restart

# 持久化：写入 fastapi.conf
echo "START_TIMEOUT=300" >> /opt/fastapi/fastapi.conf
echo "JVM_OPT=-Xms1g -Xmx2g" >> /opt/fastapi/fastapi.conf
```

---

## 七、守护配置

### 方式一：crontab（推荐）

使用 crontab 定时检测服务状态，自动拉起挂掉的进程：

```bash
# 编辑 crontab
crontab -e

# 添加以下行（每 3 分钟检测一次）
*/3 * * * * /opt/fastapi/fastapi.sh monitor >> /opt/fastapi/logs/monitor.log 2>&1
```

说明：

- `monitor` 命令会检测进程是否存活，挂掉则自动拉起
- 内置 flock 并发锁，防止多次触发重叠执行
- 日志输出到 monitor.log，便于排查
- 也可使用 `sh /opt/fastapi.sh service install` 安装 systemd 服务

### 方式二：systemd

推荐使用 systemd 管理服务，支持开机自启和自动重启：

```bash
# 安装 systemd 服务
sh /opt/fastapi.sh service install

# 常用 systemctl 命令
systemctl start fastapi    # 启动
systemctl stop fastapi     # 停止
systemctl restart fastapi  # 重启
systemctl status fastapi   # 查看状态
journalctl -f -u fastapi   # 实时日志

# 移除服务
sh /opt/fastapi.sh service remove
```

---

## 八、更新与回滚

```bash
sh /opt/fastapi.sh update        # 更新到最新正式版
sh /opt/fastapi.sh updateBeta    # 更新到 Beta 版
```

- 更新前自动备份 JAR 包，更新失败自动回滚到上一版本。
- 新版配置模板不会覆盖线上配置，会放到 `upfile/application.properties` 供人工比对合并。

---

## 九、目录结构

| 路径 | 说明 |
| --- | --- |
| `/opt/fastapi/FastApi.jar` | 程序主包 |
| `/opt/fastapi/application.properties` | 配置文件 |
| `/opt/fastapi/fastapi.env` | 运行环境变量（权限 600） |
| `/opt/fastapi/fastapi.conf` | 脚本配置（JVM 参数等） |
| `/opt/fastapi/logs/app.log` | 运行日志 |
| `/opt/fastapi/backup/` | 更新备份 |
| `/opt/fastapi/jdk-17/` | 内置 JDK 17 |
| `/opt/fastapi/files/static/` | 静态文件目录 |

---

## 十、API 对接须知

> 共 309 个接口，覆盖认证、内容、评论、用户、上传、支付、系统模块。

### 10.1 五条硬规则（违反即接口不工作）

1. **绝对不要发送 JSON 请求体**
   后端 0 个 `@RequestBody`。发 `application/json` 请求体会被静默丢弃，所有参数变成 null。必须用 `application/x-www-form-urlencoded`、`multipart/form-data` 或 URL query string。

2. **复杂参数塞进 params / searchParams**
   后端用 `@RequestParam` 接参，值是字符串，复杂对象要 `JSON.stringify` 后作为 `params=...` 传。写入类接口用 params，列表筛选用 searchParams。

3. **token 必须同时放在请求头和表单里**
   鉴权切面会读 `Authorization: Bearer <token>` 头，但 340 个接口的方法体只读 token 参数来决定「我是谁」。只传头会导致 uid 被解析成 0。

4. **HTTP 状态码永远不要用来判断业务结果**
   401/403/业务失败全都是 HTTP 200。只判断响应体里的 code。

5. **响应 Content-Type 是 text/plain**
   后端方法返回 String，走 StringHttpMessageConverter。response.data 大概率是字符串而不是对象，需要自己 `JSON.parse`。

### 10.2 code 枚举

| code | 含义 | 场景 |
| --- | --- | --- |
| `1` | 成功 | 绝大多数接口 |
| `0` | 失败/无结果 | 业务校验失败、参数缺失、JSON解析失败 |
| `-1` | 硬错误 | 查询失败、支付网关异常 |
| `401` | 未登录/token失效 | 切面拦截（HTTP仍是200） |
| `403` | 无权限 | 权限不足、IP被禁（HTTP仍是200） |
| `200` | 验证通过 | 仅 adminVerify/userVerify |
| `202` | 账号永久封禁 | 仅验证类接口 |
| `203` | 账号暂时封禁 | 仅验证类接口 |

### 10.3 权限等级（purview）

| 值 | 标签 | 说明 |
| --- | --- | --- |
| `-3` | 完全跳过 | 跳过所有校验（含IP黑名单） |
| `-2` | 不做注解级校验 | 连登录都不要求 |
| `-1` | 可选登录 | 若站点开启全局登录，自动降级为0 |
| `0` | 需要登录 | 任意已登录用户 |
| `1` | 编辑/管理员 | group ∈ {administrator, editor} |
| `2` | 仅管理员 | group == administrator |
| `3` | 版主+ | administrator/editor 或版主≥3级 |

### 10.4 请求封装示例（JavaScript / TypeScript）

```javascript
const BASE_URL = 'https://api.358583.xyz';
const TOKEN_KEY = 'userToken';

const getToken = () => localStorage.getItem(TOKEN_KEY) || '';

async function request(path, params = {}) {
  const token = getToken();
  const body = new URLSearchParams();
  for (const [k, v] of Object.entries(params)) {
    if (v === undefined || v === null || v === '') continue;
    body.append(k, typeof v === 'object' ? JSON.stringify(v) : String(v));
  }
  if (token) body.append('token', token); // token 走参数
  const res = await fetch(BASE_URL + path, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      'Accept': 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
    },
    body,
  });
  const text = await res.text();
  return JSON.parse(text); // 手动 parse
}
```

---

## 十一、更新日志

### v4.5（2026-10-02）修复启动超时误报

- `START_TIMEOUT` 默认 60 → 180（实测冷启动 92s，留 2 倍余量）
- 告警文案补全判据：打印已等待秒数、端口状态、日志时间戳
- 注意：它只是等待上限不是 sleep，就绪即返回，调大不拖慢启动

### v4.4（2026-10-02）修复重启后日志不清空

- 恢复 `LOG_CLEAR_ON_START` + 日志准备函数组
- systemd 分支也接上日志处理（原先被绕过）
- systemd unit 加 `ExecStartPre` 覆盖直接 `systemctl restart`
- unit 用 archive 而非 truncate，保护崩溃日志

### v4.3（2026-10-01）集成 FFmpeg 依赖管理

- 新增 `ffmpeg` 子命令：安装/检测 FFmpeg
- 安装策略：包管理器 → 失败自动回落静态编译包
- 静态包按 glibc 选源（CentOS 7 用 johnvansickle）
- `status` / `dump` 显示 FFmpeg 状态

### v4.2（2026-09-29）升级 Spring Boot 3.5 / JDK 17

- `install_jdk` 改为 Eclipse Temurin JDK 17
- 下载源四级兜底：自建 → 清华 TUNA → 固定版本 → GitHub
- 支持 x86_64 / aarch64
- 不再卸载系统已有 JDK

---

## 十二、相关链接

| 资源 | 地址 |
| --- | --- |
| 安装指南 | htt |
| 管理脚本下载 | https://shell |
| 程序包下载 | https://FastApi.jar |
| API 接口地址 | 你的部署域名（如 http） |
