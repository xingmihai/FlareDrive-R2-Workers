# FlareDrive-R2 Worker 部署指南

## 前置要求

- Cloudflare 账号
- 安装 Node.js 和 npm
- 安装 Wrangler CLI: `npm install -g wrangler`
- 已开通 Cloudflare R2 服务

## 部署步骤

### 1. 配置 R2 存储桶

1. 前往 Cloudflare R2 控制台
2. 创建新的存储桶（建议名称全小写，如 `my-drive-bucket`）
3. 记录存储桶名称

### 2. 配置公开访问（用于 /raw/ 路径）

1. 在 R2 控制台中，找到你的存储桶
2. 进入 **Settings** 标签页
3. 配置 **Public URLs**，启用公开访问
4. 记录公开访问 URL（如 `https://pub-xxx.r2.dev`）

### 3. 配置 R2 S3 API 凭证

1. 前往 Cloudflare 仪表盘 -> R2 -> Manage R2 API Tokens
2. 创建新的 API Token
3. 记录 **Access Key ID** 和 **Secret Access Key**
4. 记录 **Account ID**（在 R2 控制台页面右侧）

### 4. 配置 wrangler.toml

编辑 `wrangler.toml` 文件，填入你的配置信息：

```toml
name = "flaredrive-r2"                    # Worker 名称
main = "src/index.js"
compatibility_date = "2024-01-01"

[[r2_buckets]]
binding = "BUCKET"
bucket_name = "your-bucket-name"          # 替换为你的 R2 桶名

[assets]
directory = "./public"
binding = "ASSETS"

[vars]
PUBURL = "https://your-puburl.r2.dev"              # 替换为你的 R2 公开访问 URL
CF_ACCOUNT_ID = "your-account-id"                  # 替换为你的 Cloudflare 账号 ID
AWS_ACCESS_KEY_ID = "your-access-key-id"           # 替换为你的 S3 Access Key
AWS_SECRET_ACCESS_KEY = "your-secret-access-key"   # 替换为你的 S3 Secret Key
GUEST = "*"                                        # 访客权限，* 表示允许访问所有路径
```

### 5. 设置 Secrets（可选）

如果你不想在配置文件中明文存储密钥，可以使用 Wrangler Secrets：

```bash
wrangler secret put AWS_ACCESS_KEY_ID
wrangler secret put AWS_SECRET_ACCESS_KEY
```

然后在 `wrangler.toml` 中删除对应的 `[vars]` 配置项。

### 6. 安装依赖并部署

```bash
# 安装依赖
npm install

# 登录 Cloudflare（首次使用需要）
wrangler login

# 本地开发测试
npm run dev

# 部署到 Cloudflare
npm run deploy
```

### 7. 访问你的网盘

部署完成后，Wrangler 会输出你的 Worker URL（如 `https://flaredrive-r2.your-account.workers.dev`），打开即可使用。

## 权限配置

### 访客访问 (GUEST)

在 `wrangler.toml` 的 `[vars]` 中设置 `GUEST`：

- `GUEST = "*"` - 允许访客访问所有路径
- `GUEST = "public,shared"` - 仅允许访问 `public` 和 `shared` 目录
- 不设置 `GUEST` - 需要 Basic 认证才能访问

### Basic 认证账户

可以通过 Wrangler 设置用户账户和密码：

```bash
# 设置用户名和密码（格式：用户名=密码）
wrangler secret put user1    # 值为允许访问的路径，如 "public,private"
```

然后在访问时，浏览器会弹出 Basic 认证框，输入用户名和密码即可。

## 自定义域名（可选）

1. 在 Cloudflare 控制台中，进入你的 Worker
2. 点击 **Triggers** -> **Custom Domains**
3. 添加你的自定义域名

## 注意事项

1. 首次部署后，网盘是空的，需要通过上传功能添加文件
2. 大文件上传使用分片上传，支持断点续传
3. `/raw/` 路径用于文件直链访问，需要配置 `PUBURL`

## 项目结构

```
flaredrive-worker/
├── src/
│   └── index.js          # Worker 入口（API路由 + 静态文件服务）
├── public/               # 静态文件目录
│   ├── index.html        # 前端入口
│   ├── favicon.ico
│   └── assets/           # Vue组件、CSS、JS等前端资源
│       ├── App.vue
│       ├── *.vue
│       ├── main.css
│       ├── main.mjs
│       └── ...
├── wrangler.toml         # Cloudflare Worker 配置
├── package.json
└── DEPLOY.md             # 本文件
```
