# Loki-Nginx 日志监控系统

## 项目介绍

这是一个基于 Grafana Loki 的 Nginx 日志监控系统，用于收集、存储和可视化 Nginx 的访问日志和错误日志。

该系统使用 Docker Compose 编排，包含以下组件：

- **Nginx**：Web 服务器，配置为输出 JSON 格式的访问日志
- **Loki**：日志聚合系统，负责存储和索引日志
- **Promtail**：日志收集代理，将 Nginx 日志发送到 Loki
- **Grafana**：可视化平台，用于查询和展示日志数据

## 系统架构

```
+--------+    日志文件    +-----------+    推送日志    +--------+    查询日志    +-----------+
| Nginx  | ------------> | Promtail  | ------------> |  Loki  | <------------ | Grafana   |
+--------+               +-----------+               +--------+               +-----------+
```

## 目录结构

```
├── docker-compose.yaml   # Docker Compose 配置文件
├── grafana/              # Grafana 配置目录
│   └── provisioning/     # Grafana 自动配置目录
│       └── etc/          # 数据源和仪表板配置
├── loki/                 # Loki 配置目录
│   └── config.yaml       # Loki 配置文件
├── nginx/                # Nginx 配置目录
│   ├── default.conf      # Nginx 默认站点配置
│   ├── nginx-stub-status.conf  # Nginx 状态监控配置
│   └── nginx.conf        # Nginx 主配置文件
└── promtail/             # Promtail 配置目录
    └── promtail.yaml     # Promtail 配置文件
```

## 配置说明

### Nginx 配置

Nginx 配置了 JSON 格式的访问日志，包含丰富的字段信息，便于后续分析和可视化：

- `nginx.conf`：主配置文件，定义了 JSON 格式的日志
- `default.conf`：默认站点配置，监听 8080 端口
- `nginx-stub-status.conf`：Nginx 状态监控配置，监听 8081 端口（默认未启用）

### Loki 配置

Loki 使用文件系统作为存储后端，配置简单：

- 禁用身份验证
- HTTP 监听端口为 3100
- 使用内存存储一致性哈希环
- 使用 TSDB 存储格式

### Promtail 配置

Promtail 配置为收集 Nginx 日志并发送到 Loki：

- 监听端口为 9080
- 目标 Loki 服务为 `http://loki-service:3100/loki/api/v1/push`
- 配置了 JSON 解析和标签提取

### Grafana 配置

Grafana 配置为匿名访问，无需登录：

- 端口映射为 9000:3000
- 启用匿名访问并赋予管理员权限
- 禁用注册功能

## 使用指南

### 启动服务

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps
```

容器卸载

```shell
docker-compose down
```

### 访问服务

- **Nginx**：http://localhost:8080
- **Grafana**：http://localhost:9000
- **Loki API**：http://localhost:9001

### 查看日志

1. 访问 Grafana (http://localhost:9000)
2. 导航到 Explore 页面
3. 选择 Loki 数据源
4. 使用 LogQL 查询日志，例如：`{job="loki-nginx-access"}`

## 常见问题

### 日志不显示

- 检查 Promtail 服务是否正常运行
- 检查 Nginx 日志目录是否正确挂载
- 检查 Promtail 配置中的路径是否正确

### Grafana 无法连接 Loki

- 检查 Loki 服务是否正常运行
- 检查 Grafana 数据源配置是否正确

## 注意事项

- 默认配置中，日志数据存储在容器内部，重启容器后数据会丢失。如需持久化存储，请修改 Loki 配置
- 生产环境中应启用身份验证和 TLS 加密
- 默认未启用 Nginx 状态监控，如需启用，请取消 docker-compose.yaml 中相关行的注释