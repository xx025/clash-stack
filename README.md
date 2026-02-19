# Mihomo + MetaCubeXd + Sub-Store 一键栈

通过 Docker Compose 一键部署：**Mihomo（Clash Meta）代理** + **MetaCubeXd 官方 Web 面板** + **Sub-Store 多订阅组合与管理**。

## 服务与端口

| 服务 | 端口 | 说明 |
|------|------|------|
| Mihomo | 7890 (混合) / 7891 (API) | 代理核心（Clash Meta 兼容） |
| MetaCubeXd | 7892 | Mihomo 官方 Web 控制面板 |
| Sub-Store | 7893 | 多订阅组合与管理（合并、筛选、重命名、导出 Clash/Mihomo 等） |

宿主机端口连续占用 **7890–7893**，便于防火墙/安全组一次放行。

## 快速开始

```bash
docker compose up -d
docker compose ps
```

## Sub-Store：多订阅组合

Sub-Store 用于**管理多个订阅并组合成一条订阅**，而不是做格式转换。你可以：

- **添加多个订阅/机场**：在 Sub-Store 里添加多条订阅链接或个人节点  
- **合并为一条订阅**：创建「订阅合集」，选择要包含的订阅，导出为一条链接  
- **筛选与重命名**：按关键词筛选节点、正则重命名、加国旗、排序等  
- **导出多种格式**：导出为 Clash、Clash Meta、Surge、Quantumult X、Sing-box 等  

### 访问 Sub-Store

1. 浏览器打开：`http://<你的服务器IP>:7893`
2. 首次进入需配置 **Backend（后端）**：点击右上角设置，在「Backend 设置」中填写：
   - 本机：`http://127.0.0.1:7893/api`
   - 远程：`http://<你的服务器IP>:7893/api`
   - 若你修改了 `SUB_STORE_FRONTEND_BACKEND_PATH`，把上面的 `/api` 换成你设的路径
3. 建议将 `SUB_STORE_FRONTEND_BACKEND_PATH` 改为随机字符串（如 20 位），避免 API 被随意访问。在 `docker-compose.yml` 里修改后执行：`docker compose up -d sub-store`

### 在 Mihomo 中使用 Sub-Store 导出的订阅

两种常用方式：

**方式一：proxy-providers（推荐）**  
在 `config/mihomo/config.yaml` 里用 Sub-Store 的「下载链接」作为 `proxy-providers` 的 `url`，例如：

```yaml
proxy-providers:
  my-sub:
    type: http
    url: http://<服务器IP>:7893/download/你的合集名称?target=ClashMeta
    interval: 3600
    path: ./proxies/my-sub.yaml
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 300

proxy-groups:
  - name: 🚀 节点选择
    type: select
    use: [my-sub]
  # ... 其他组与 rules
```

**方式二：直接订阅链接**  
在 Sub-Store 中复制「订阅链接」（即合并后的订阅 URL），在 Mihomo 或 MetaCubeXd 中通过「远程配置/订阅」填入该链接（若客户端支持）。

## 使用流程简述

1. **Sub-Store**：`http://<IP>:7893` → 添加多个订阅/节点 → 创建合集 → 复制「下载链接」或「订阅链接」  
2. **Mihomo 配置**：在 `config/mihomo/config.yaml` 里用上述链接配置 `proxy-providers`（或按客户端要求填订阅），然后 `docker compose restart mihomo`  
3. **MetaCubeXd**：`http://<IP>:7892` → 设置 Backend URL 为 `http://<IP>:7891` → 在面板中选节点、看规则与流量  
4. **代理**：系统/浏览器代理设为 `<IP>:7890`（HTTP 与 SOCKS5 均走该混合端口）  

## 目录结构

```
.
├── docker-compose.yml
├── config/
│   └── mihomo/
│       └── config.yaml   # Mihomo 配置（proxy-providers 可指向 Sub-Store 下载链接）
└── README.md
```

Sub-Store 数据持久化在 Docker 命名卷 `sub_store_data` 中，重启或重建容器不会丢失订阅与合集配置。

## 常用命令

```bash
docker compose up -d
docker compose down
docker compose logs -f sub-store
docker compose restart mihomo
```

## 安全与生产环境建议

- 将 `SUB_STORE_FRONTEND_BACKEND_PATH` 改为随机长字符串，并仅在内网或反向代理后暴露 7893。  
- 生产环境建议用 Nginx/Caddy 对 7891、7892、7893 做反向代理并配置 HTTPS。
- 防火墙/安全组放行 **7890–7893** 即可（连续端口）。
