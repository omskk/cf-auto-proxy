# CF 动态代理服务

基于 [jonssonyan/cf-workers-proxy](https://github.com/jonssonyan/cf-workers-proxy) 二次开发的动态代理服务，支持通过 API 动态更新代理目标域名，适用于开发测试环境。

## 功能特性
- 🚀 动态代理主机名更新（通过 `/update-hostname` 接口）
- 🔄 自动替换响应内容中的域名
- 🛡️ 支持多种过滤规则（UA/IP/地区 黑白名单）
- ⚡ 自动处理 HTTPS 重定向
- 🔒 支持 KV 存储，支持持久化配置

## 快速开始
### 通过 Cloudflare 控制台部署
1. **创建 Worker 服务**
   - 登录 [Cloudflare Dashboard](https://dash.cloudflare.com)
   - 进入 Workers 服务 → 创建服务 → 输入服务名称 "cf-auto-proxy"
   - 选择 "HTTP handler" 模板

2. **绑定 KV 命名空间**
   - 点击「新建命名空间」创建KV存储空间
   - 在 Worker 设置页面 → 设置 → KV 命名空间绑定

3. **配置环境变量**
   - 在 Worker 设置页面 → 设置 → 变量 → 环境变量

    | 变量名                    | 必填  | 默认值   | 示例                                             | 备注                  |
    |------------------------|-----|-------|------------------------------------------------|---------------------|
    | ORG_PROXY_HOSTNAME     | √   |       | github.com                                     | 代理地址 hostname       |
    | AUTH_KEY               | √   |       | your-secret-string                             | API调用认证密钥，需包含在请求头 |
    | PROXY_PROTOCOL         | ×   | https | https                                          | 代理地址协议              |
    | PATHNAME_REGEX         | ×   |       | ^/jonssonyan/                                  | 代理地址路径正则表达式         |
    | UA_WHITELIST_REGEX     | ×   |       | (curl)                                         | User-Agent 白名单正则表达式 |
    | UA_BLACKLIST_REGEX     | ×   |       | (curl)                                         | User-Agent 黑名单正则表达式 |
    | IP_WHITELIST_REGEX     | ×   |       | (192.168.0.1)                                  | IP 白名单正则表达式         |
    | IP_BLACKLIST_REGEX     | ×   |       | (192.168.0.1)                                  | IP 黑名单正则表达式         |
    | REGION_WHITELIST_REGEX | ×   |       | (JP)                                           | 地区白名单正则表达式          |
    | REGION_BLACKLIST_REGEX | ×   |       | (JP)                                           | 地区黑名单正则表达式          |
    | URL302                 | ×   |       | https://github.com/jonssonyan/cf-workers-proxy | 302 跳转地址            |
    | DEBUG                  | ×   | false | false                                          | 开启调试                |

## 接口说明
### 1. 更新代理目标 `/update-hostname`

#### 请求参数
| 参数名        | 必填 | 类型     | 示例值                     | 说明             |
|-------------|----|--------|--------------------------|----------------|
| newHostname | 是  | string | `api-stage.example.com` | 新的代理目标域名（可以带端口） |

#### 请求示例
```bash
# 基础请求
curl -X GET "https://cf-auto-proxy.[your-account].workers.dev/update-hostname?newHostname=api.new-backend.com"

# 带调试信息的请求（显示详细输出）
curl -ivX GET \
-H "CF-Connecting-IP: 1.1.1.1" \
"https://cf-auto-proxy.example.workers.dev/update-hostname?newHostname=backend.company.com"
```
## 安全配置
在环境变量中新增：

| 变量名     | 必填 | 示例值               | 说明                 |
|----------|----|--------------------|--------------------|
| AUTH_KEY | 是  | your-secret-string | API调用认证密钥，需包含在请求头 |

## 调用示例
```bash
# 带认证的请求
curl -X GET \
-H "Authorization: Bearer your-secret-string" \
"https://[Your-Workers-URL]/update-hostname?newHostname=api.new-backend.com"
```

## 致谢
本项目基于 [jonssonyan/cf-workers-proxy](https://github.com/jonssonyan/cf-workers-proxy) 进行功能扩展开发，主要新增以下能力：
- 动态代理目标更新接口 `/update-hostname`
- KV 存储持久化配置
- 自动域名替换增强逻辑
- 调试模式支持
