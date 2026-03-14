---
name: xhs-cli
description: 使用 xhs CLI 工具与小红书（Xiaohongshu/Red）交互，获取笔记信息、下载图片、搜索内容、管理收藏和互动。当用户提到小红书、笔记下载、搜索、点赞、收藏等场景时触发。支持批量操作和结构化输出（YAML/JSON）。
---

# XHS CLI Skill

通过 [xiaohongshu-cli](https://github.com/jackwener/xiaohongshu-cli) 工具在终端与小红书交互。

## Prerequisites

确保已安装 xiaohongshu-cli 以及已登录：

```bash
xhs status
# 正常情况下会输出已登录用户信息
✅ 已登录：用户名 (用户ID)
```

如果没有安装：

```bash
uv tool install xiaohongshu-cli
```

首次使用需登录：
```bash
xhs login      # 扫码登录
xhs status     # 检查登录状态
```

## Core Workflows

### 1. 获取笔记信息

```bash
# 基本信息
xhs note <笔记ID>

# 带图片下载
xhs note <笔记ID> --images

# AI 摘要
xhs note <笔记ID> --ai

# 热门评论
xhs note <笔记ID> --comments

# 相关推荐笔记
xhs note <笔记ID> --related
```

**结构化输出**（推荐用于后续处理）：
```bash
xhs note <笔记ID> --yaml
xhs note <笔记ID> --json
```

### 2. 搜索与发现

```bash
# 搜索笔记
xhs search "关键词" --type note --max 10

# 搜索用户
xhs search "用户名"

# 热门笔记
xhs hot --page 1 --max 20

# 推荐流
xhs feed
```

### 3. 用户相关

```bash
# 用户资料
xhs user <用户ID>
xhs user "用户名"      # 按名称搜索

# 用户笔记列表
xhs user-notes <用户ID> --max 20

# 当前登录用户信息
xhs whoami
xhs whoami --yaml
```

### 4. 图片下载

```bash
# 下载笔记所有图片
xhs images <笔记ID>

# 指定输出目录
xhs images <笔记ID> -o ~/Downloads/

# 指定图片质量
xhs images <笔记ID> --quality origin
```

### 5. 互动操作

```bash
# 点赞
xhs like <笔记ID>

# 收藏
xhs collect <笔记ID>

# 关注用户
xhs follow <用户ID>

# 取消关注
xhs unfollow <用户ID>
```

### 6. 收藏与列表管理

```bash
# 查看收藏夹列表
xhs collections

# 查看指定收藏夹内容
xhs collections <收藏夹ID> --page 1

# 观看历史
xhs history

# 关注列表
xhs following
```

## Output Formats

### 默认输出
- TTY（终端）：富文本格式，带颜色
- 非 TTY：自动使用 YAML

### 显式指定格式
```bash
# 结构化 YAML（推荐，token效率高）
xhs <command> --yaml

# JSON 格式
xhs <command> --json

# 富文本（强制）
xhs <command> --format rich
```

### 环境变量
```bash
OUTPUT=yaml      # 默认 YAML
OUTPUT=json      # 默认 JSON
OUTPUT=rich      # 默认富文本
OUTPUT=auto      # 自动检测
```

## Structured Data Schema

所有 `--yaml` / `--json` 输出使用统一信封格式：

```yaml
ok: true
schema_version: "1.0"
data:
  note: {...}
  images: [...]
  comments: [...]
  ai_summary: "..."
error: null
```

**数据路径映射：**

| 命令 | 数据路径 |
|------|---------|
| `xhs note` | `data.note`, `data.images`, `data.ai_summary`, `data.comments`, `data.related` |
| `xhs hot` / `feed` | `data.items` |
| `xhs search` | 标准化的用户/笔记列表 |
| 写操作（like/collect等） | 标准化结果 |

## Error Codes

| 错误码 | 含义 | 解决方案 |
|-------|------|---------|
| `not_authenticated` | 需要登录 | 运行 `xhs login` |
| `permission_denied` | 权限不足 | 检查账号权限 |
| `invalid_input` | 输入无效 | 检查笔记ID格式 |
| `network_error` | 网络错误 | 检查网络连接/代理 |
| `upstream_error` | 上游错误 | 小红书服务异常，稍后重试 |
| `not_found` | 资源不存在 | 检查笔记/用户是否存在 |
| `rate_limited` | 请求过于频繁 | 减少 `--max` 参数，等待后重试 |
| `internal_error` | 内部错误 | 重新登录或联系开发者 |

## Best Practices

1. **优先使用结构化输出**：在自动化脚本中使用 `--yaml` 或 `--json` 便于解析
2. **批量操作添加延迟**：避免触发 rate limit
3. **图片下载注意版权**：仅供个人学习使用
4. **关注反爬机制**：合理控制请求频率

## Examples

### 示例 1：获取笔记完整信息
```bash
xhs note <note_id> --images --comments --ai --yaml
```

### 示例 2：批量获取热门笔记
```bash
xhs hot --page 1 --max 50 --yaml
```

### 示例 3：搜索并筛选
```bash
xhs search "穿搭教程" --type note --max 10 --yaml
```

### 示例 4：下载笔记图片
```bash
xhs images <note_id> -o ./images/
```

### 示例 5：获取用户最近笔记
```bash
# 先搜索用户ID
xhs user "用户名"
# 然后用用户ID获取笔记
xhs user-notes <user_id> --max 20 --yaml
```
