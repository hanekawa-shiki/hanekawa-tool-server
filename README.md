# holiday-cn-worker

基于 Cloudflare Worker + KV 的中国节假日 API，每周从 [NateScarlet/holiday-cn](https://github.com/NateScarlet/holiday-cn) 自动同步数据。

## 架构

```
GitHub (holiday-cn)  →  Cloudflare Worker (Cron, 每周)  →  KV Storage
                                                              ↓
                                                    HTTP API (前端调用)
```

## 快速开始

```bash
npm install
npm run kv:create        # 获取 KV namespace id 填入 wrangler.toml
npm run deploy
# 部署后设置 CORS 白名单域名（加密存储，不会暴露在代码中）
echo "https://your-app.example.com" | npx wrangler secret put ALLOWED_ORIGINS
```

如果忘记了 KV namespace id，运行 `npm run kv:list` 查看已有的命名空间列表。

修改 ALLOWED_ORIGINS（部署后随时可改，无需重新部署代码）：
```bash
echo "https://new-domain.example.com,https://another.com" | npx wrangler secret put ALLOWED_ORIGINS
```


## API 接口

### 获取指定年份完整数据

```
POST /api/holidays/year
Content-Type: application/json

{ "year": 2025 }
```

返回原始 holiday-cn JSON，结构与仓库一致（包含 `year`、`days`、`papers` 字段）。

## 项目结构

```
src/
├── index.ts    # Worker 入口（POST API + Cron）
├── sync.ts     # GitHub → KV 全量同步
└── types.ts    # 类型定义
```

## KV Key 设计

| Key | 说明 |
|-----|------|
| `holiday:years` | 年份索引 `[2007, 2008, ..., 2027]` |
| `holiday:year:2025` | 某年完整 JSON |

## 定时任务

每周日 UTC 00:00（北京时间 08:00）自动全量同步。