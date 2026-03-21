---
name: xhs-cli
description: 使用 xhs CLI 工具与小红书（Xiaohongshu/Red）交互。支持搜索、阅读笔记、评论、点赞、收藏、关注、发布、通知等功能。当用户提到小红书、笔记查看、搜索内容、点赞、收藏、评论、发布等场景时触发。
---

# XHS CLI Skill

通过 [xiaohongshu-cli](https://github.com/jackwener/xiaohongshu-cli) 在终端与小红书交互。

## Prerequisites

```bash
# 检查是否已安装并登录
xhs status
```

如果未安装：
```bash
uv tool install xiaohongshu-cli
# 升级到最新版本
uv tool upgrade xiaohongshu-cli
```

首次使用需登录：
```bash
xhs login           # 自动提取浏览器 Cookie
xhs login --qrcode  # 扫码登录
xhs whoami          # 确认登录状态
```

## Core Commands

### 认证（Auth）

```bash
xhs login                    # 从浏览器自动提取 Cookie
xhs login --qrcode           # 扫码登录（终端显示二维码）
xhs status                   # 检查登录状态
xhs whoami                   # 查看当前用户详细资料
xhs whoami --json            # 结构化 JSON 输出
xhs logout                   # 清除保存的 Cookie
```

### 搜索（Search）

```bash
xhs search "关键词"                    # 搜索笔记
xhs search "旅行" --sort popular       # 排序：general, popular, latest
xhs search "穿搭" --type video         # 类型：all, video, image
xhs search "AI" --page 2              # 翻页
xhs search-user "用户名"               # 搜索用户
xhs topics "美食"                      # 搜索话题/标签
```

### 阅读（Reading）

```bash
# 短索引（在 search/feed/hot/user-posts 之后使用）
xhs read 1                             # 阅读上次列表的第 1 条笔记
xhs comments 1                         # 查看上次列表的第 1 条笔记评论

# 完整命令
xhs read <note_id>                     # 阅读笔记（仅 API）
xhs read "https://www.xiaohongshu.com/explore/xxx?xsec_token=yyy"  # 通过 URL 阅读

xhs comments <url>                     # 查看评论（URL 含 xsec_token）
xhs comments <url> --all               # 获取全部评论（自动翻页）
xhs comments <url> --all --json        # 全部评论，JSON 格式
xhs comments <note_id> --xsec-token T  # 显式指定 token

xhs sub-comments <note_id> <cmt_id>   # 查看评论的回复

xhs user <user_id>                     # 用户主页
xhs user-posts <user_id>              # 用户发布的笔记
xhs user-posts <user_id> --cursor X   # 用 cursor 翻页
```

### 发现（Feed & Discovery）

```bash
xhs feed                              # 推荐 Feed
xhs hot                               # 热门笔记（默认：food）
xhs hot -c fashion                    # 分类：fashion, food, cosmetics,
                                      #       movie, career, love, home,
                                      #       gaming, travel, fitness
```

### 社交（Social）

```bash
xhs favorites                          # 我的收藏夹
xhs favorites <user_id>                # 其他用户的收藏
xhs likes                              # 我的点赞
xhs likes <user_id>                    # 其他用户的点赞
xhs follow <user_id>                   # 关注用户
xhs unfollow <user_id>                 # 取消关注
```

### 互动（Interactions）

```bash
# 短索引形式（在列表命令之后使用）
xhs like 1                             # 给第 1 条笔记点赞
xhs favorite 1                         # 收藏第 1 条笔记
xhs comment 1 -c "好赞！"              # 评论第 1 条笔记
xhs reply 1 --comment-id X -c "回复"   # 回复第 1 条笔记的评论

# 完整形式
xhs like <note_id>                     # 点赞
xhs like <note_id> --undo             # 取消点赞
xhs favorite <note_id>                 # 收藏
xhs unfavorite <note_id>               # 取消收藏
xhs comment <note_id> -c "好赞！"     # 发评论
xhs reply <note_id> --comment-id X -c "回复"  # 回复评论
xhs delete-comment <note_id> <cmt_id> # 删除自己的评论
```

### 创作者（Creator）

```bash
xhs my-notes                           # 我的笔记列表
xhs my-notes --page 1                 # 翻页
xhs post --title "标题" --body "正文" --images img.jpg  # 发布图文笔记
xhs delete <note_id>                   # 删除笔记
xhs delete <note_id> -y               # 跳过确认
```

### 通知（Notifications）

```bash
xhs unread                             # 未读数（点赞、@、新关注）
xhs notifications                      # 评论和 @ 通知
xhs notifications --type likes        # 赞和收藏通知
xhs notifications --type connections   # 新增关注通知
```

## Structured Output

所有命令支持结构化输出：

```bash
xhs <command> --yaml    # YAML 格式（推荐，token 效率高）
xhs <command> --json    # JSON 格式
```

非 TTY 环境（管道/脚本调用）默认输出 YAML。

统一输出信封（Envelope）格式：
```yaml
ok: true
schema_version: "1"
data: { ... }
error: null
```

环境变量控制全局格式：
```bash
OUTPUT=yaml xhs search "美食"   # 强制 YAML
OUTPUT=json xhs whoami          # 强制 JSON
```

## Short-Index Navigation

执行列表命令（search/feed/hot/user-posts/favorites/my-notes）后，结果缓存在 `~/.xiaohongshu-cli/index_cache.json`。

可用短索引的命令：`read`, `comments`, `like`, `favorite`, `unfavorite`, `comment`, `reply`

```bash
xhs search "美食"      # 搜索后缓存结果
xhs read 1            # 阅读第 1 条
xhs comments 1        # 查看第 1 条评论
xhs like 1            # 点赞第 1 条
```

## Error Handling

| 错误 | 解决方案 |
|------|---------|
| `NoCookieError` | 浏览器访问 xiaohongshu.com 登录后，执行 `xhs login` |
| `NeedVerifyError` | 浏览器完成验证码后重试 |
| `IpBlockedError` | 切换网络（热点/VPN） |
| `SessionExpiredError` | 执行 `xhs login` 刷新 Cookie |
| 请求慢 | 正常现象，内置高斯随机延迟避免风控 |

## Best Practices

1. **优先使用短索引**：先 `xhs search`，再 `xhs read 1` 操作，流畅自然
2. **结构化输出用于自动化**：脚本中加 `--yaml` 便于解析
3. **定期升级**：`uv tool upgrade xiaohongshu-cli`，避免 API 变更导致错误
4. **合理频率**：批量操作间增加间隔，避免触发验证码

## Examples

```bash
# 场景 1：搜索并阅读笔记
xhs search "川菜教程" --sort popular
xhs read 1
xhs comments 1

# 场景 2：查看用户内容
xhs search-user "某博主"
xhs user-posts <user_id> --yaml

# 场景 3：批量获取热门笔记
xhs hot -c travel --yaml

# 场景 4：互动
xhs search "咖啡探店"
xhs like 1
xhs comment 1 -c "看起来很好喝！"

# 场景 5：获取所有评论
xhs comments "https://www.xiaohongshu.com/explore/xxx?xsec_token=yyy" --all --yaml
```
