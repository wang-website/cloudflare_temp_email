# Cloudflare workers 后端

## 初始化项目

```bash
cd worker
pnpm install
cp wrangler.toml.template wrangler.toml
```

## 创建 KV 缓存

> [!NOTE]
> 如果你要启用注册用户功能，并需要发送邮件验证，则需要创建 `KV` 缓存, 不需要可跳过此步骤
> 如果需要 Telegram Bot，需要创建 `KV` 缓存，不需要可跳过此步骤

通过命令行创建 KV 缓存，或者在 Cloudflare 控制台创建，然后复制对应配置到 `wrangler.toml` 文件中

```bash
wrangler kv:namespace create DEV
```

## 修改 `wrangler.toml` 配置文件

```toml
name = "cloudflare_temp_email"
main = "src/worker.ts"
compatibility_date = "2023-12-01"
# 如果你想使用自定义域名，你需要添加 routes 配置
# routes = [
#  { pattern = "temp-email-api.xxxxx.xyz", custom_domain = true },
# ]
node_compat = true

# 如果你想要使用定时任务清理邮件，取消下面的注释，并修改 cron 表达式
# [triggers]
# crons = [ "0 0 * * *" ]

# 通过 Cloudflare 发送邮件
# send_email = [
#    { name = "SEND_MAIL" },
# ]

[vars]
# TITLE = "Custom Title" # 自定义网站标题
PREFIX = "tmp" # 要处理的邮箱名称前缀，不需要后缀可配置为空字符串
# (min, max) adderss的长度，如果不设置，默认为(1, 30)
# MIN_ADDRESS_LEN = 1
# MAX_ADDRESS_LEN = 30
# 如果你想要你的网站私有，取消下面的注释，并修改密码
# PASSWORDS = ["123", "456"]
# admin 控制台密码, 不配置则不允许访问控制台
# ADMIN_PASSWORDS = ["123", "456"]
# admin 联系方式，不配置则不显示，可配置任意字符串
# ADMIN_CONTACT = "xx@xx.xxx"
DOMAINS = ["xxx.xxx1" , "xxx.xxx2"] # 你的域名, 支持多个域名
# 对于中文域名，可以使用 DOMAIN_LABELS 显示域名的中文展示名称
# DOMAIN_LABELS = ["中文.xxx", "xxx.xxx2"]
JWT_SECRET = "xxx" # 用于生成 jwt 的密钥, jwt 用于给用户登录以及鉴权
BLACK_LIST = "" # 黑名单，用于过滤发件人，逗号分隔
# 是否允许用户创建邮件, 不配置则不允许
ENABLE_USER_CREATE_EMAIL = true
# 允许用户删除邮件, 不配置则不允许
ENABLE_USER_DELETE_EMAIL = true
# 允许自动回复邮件
ENABLE_AUTO_REPLY = false
# 是否启用 webhook
# ENABLE_WEBHOOK = true
# 前端界面页脚文本
# COPYRIGHT = "Dream Hunter"
# 默认发送邮件余额，如果不设置，将为 0
# DEFAULT_SEND_BALANCE = 1
# Turnstile 人机验证配置
# CF_TURNSTILE_SITE_KEY = ""
# CF_TURNSTILE_SECRET_KEY = ""
# telegram bot 最多绑定邮箱数量
# TG_MAX_ACCOUNTS = 5
# 全局转发地址列表，如果不配置则不启用，启用后所有邮件都会转发到列表中的地址
# FORWARD_ADDRESS_LIST = ["xxx@xxx.com"]

# D1 数据库的名称和 ID 可以在 cloudflare 控制台查看
[[d1_databases]]
binding = "DB"
database_name = "xxx" # D1 数据库名称
database_id = "xxx" # D1 数据库 ID

# kv config 用于用户注册发送邮件验证码，如果不启用用户注册或不启用注册验证，可以不配置
# [[kv_namespaces]]
# binding = "KV"
# id = "xxxx"

# 新建地址限流配置 /api/new_address
# [[unsafe.bindings]]
# name = "RATE_LIMITER"
# type = "ratelimit"
# namespace_id = "1001"
# # 10 requests per minute
# simple = { limit = 10, period = 60 }
```

## Telegram Bot 配置

> [!NOTE]
> 如果不需要 Telegram Bot, 可跳过此步骤

请先创建一个 Telegram Bot，然后获取 `token`，然后执行下面的命令，将 `token` 添加到 secrets 中

```bash
pnpm wrangler secret put TELEGRAM_BOT_TOKEN
```

## 部署

第一次部署会提示创建项目, `production` 分支请填写 `production`

```bash
pnpm run deploy
```

部署成功之后再路由中可以看到 `worker` 的 `url`，控制台也会输出 `worker` 的 `url`

![worker](/readme_assets/worker.png)

> [!NOTE]
> 打开 `worker` 的 `url`，如果显示 `OK` 说明部署成功
>
> 打开 `/health_check`，如果显示 `OK` 说明部署成功
