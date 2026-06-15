# Sub2API Docker Image

Sub2API is an AI API Gateway Platform for distributing and managing AI product subscription API quotas.

## Quick Start

```bash
# 基础用法
docker run -d \
  --name sub2api \
  -p 8080:8080 \
  -e DATABASE_URL="postgres://user:pass@host:5432/sub2api?sslmode=disable&search_path=public" \
  -e REDIS_URL="redis://host:6379?pool_size=256&min_idle_conns=20&idle_timeout_seconds=300" \
  yw79641760/sub2api:latest
```

## Docker Compose

### Basic

```yaml
version: '3.8'

services:
  sub2api:
    image: yw79641760/sub2api:latest
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/sub2api?sslmode=disable
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=sub2api
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Optimized (Recommended for Production)

```yaml
version: '3.8'

services:
  sub2api:
    image: yw79641760/sub2api:latest
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/sub2api?sslmode=disable
      # 优化 Redis 连接池配置（减少连接数）
      - REDIS_URL=redis://redis:6379?pool_size=256&min_idle_conns=20&idle_timeout_seconds=300
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=sub2api
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Environment Variables

### Database & Redis (Required)

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgres://user:pass@host:5432/sub2api?sslmode=disable&search_path=public` |
| `REDIS_URL` | Redis connection string | `redis://host:6379` or `rediss://host:6379` (TLS) |

**DATABASE_URL 查询参数**：

| Query Parameter | Description | Default |
|-----------------|-------------|---------|
| `sslmode` | SSL 模式 | `prefer` |
| `search_path` | PostgreSQL schema | `public` |
| `timezone` | 时区 | `Asia/Shanghai` |

Example:
```bash
# 完整配置
DATABASE_URL=postgres://user:pass@host:5432/sub2api?sslmode=disable&search_path=public&timezone=Asia/Shanghai
```

### Redis Connection Pool (Optional)

Redis 连接池参数可以通过 `REDIS_URL` 查询参数配置：

| Query Parameter | Description | Default |
|-----------------|-------------|---------|
| `pool_size` | 连接池大小（最大并发连接数） | 1024 |
| `min_idle_conns` | 最小空闲连接数 | 20 |
| `idle_timeout_seconds` | 连接池超时（秒） | 300 |
| `dial_timeout_seconds` | 建连超时（秒） | 5 |
| `read_timeout_seconds` | 读取超时（秒） | 3 |
| `write_timeout_seconds` | 写入超时（秒） | 3 |

#### Redis Pool Example

```yaml
# 优化连接池配置（推荐生产环境使用）
REDIS_URL=redis://redis:6379?pool_size=256&min_idle_conns=20&idle_timeout_seconds=300

# 带密码配置
REDIS_URL=redis://:password@redis:6379?pool_size=256&min_idle_conns=20&idle_timeout_seconds=300

# TLS 连接
REDIS_URL=rediss://redis:6379?pool_size=256&min_idle_conns=20
```

**参数说明**：
- `min_idle_conns=20`：保持20个空闲连接，避免冷启动延迟同时防止连接数膨胀
- `idle_timeout_seconds=300`：5分钟超时，定期清理长时间空闲的连接
- 预估连接数：基础 6 + min_idle_conns 20 ≈ **26**（而非之前的 140+）

### Server Options

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | Server port | `8080` |
| `GIN_MODE` | Gin framework mode (`debug`/`release`) | `release` |
| `JWT_SECRET` | JWT signing secret (auto-generated if empty) | - |
| `TOTP_ENCRYPTION_KEY` | TOTP encryption key (auto-generated if empty) | - |

### Optional Features

| Variable | Description | Default |
|----------|-------------|---------|
| `RUN_MODE` | Set to `simple` for simple mode | - |
| `DROP_SCHEMA` | Set to `true` to reset database on first install | - |

## Supported Architectures

- `linux/amd64`
- `linux/arm64`

## Tags

- `latest` - Latest stable release
- `x.y.z` - Specific version
- `x.y` - Latest patch of minor version
- `x` - Latest minor of major version

## Links

- [GitHub Repository](https://github.com/weishaw/sub2api)
- [Documentation](https://github.com/weishaw/sub2api#readme)
