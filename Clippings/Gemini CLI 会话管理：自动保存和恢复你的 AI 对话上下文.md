---
title: "Gemini CLI 会话管理：自动保存和恢复你的 AI 对话上下文"
source: "https://blog.frognew.com/2025/12/gemini-cli-session-management.html"
author:
  - "[[青蛙小白]]"
published: 2025-12-02
created: 2026-02-06
description: "Gemini CLI 提供了强大的会话管理功能，能够自动保存所有对话历史，并支持随时恢复。这篇文章将详细介绍如何使用这些功能。                                                                                                                                                              Session                                              Session Browser                          ▸ refactor-auth (2 days ago)              feature-api (5 hours ago)        bug-fix-mem (Just now)                                                        Day 1                    Day 2                                Day 3                                              Now                                          project-a                        project-b                        project-c                    $ gemini --resume                                                                                    自动保存：无需手动操作  #Gemini CLI 提供了完全自动的会话保存机制。不需要记住任何保存命令，也不需要担心忘记保存，每次交互都会自动记录。"
tags:
  - "clippings"
---
##### 📅 2025-12-02 | 🖱️2,104

🔖

Gemini CLI 提供了强大的会话管理功能，能够自动保存所有对话历史，并支持随时恢复。这篇文章将详细介绍如何使用这些功能。

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 928 100"><defs><linearGradient id="sessionGradient" x1="0" y1="0" x2="100%" y2="0"><stop offset="0" style="stop-color:#F0F4F8;stop-opacity:1"></stop><stop offset="50%" style="stop-color:#E6EEF5;stop-opacity:1"></stop><stop offset="100%" style="stop-color:#DDE7F0;stop-opacity:1"></stop></linearGradient><linearGradient id="geminiGradient" x1="0" y1="0" x2="100%" y2="100%"><stop offset="0" style="stop-color:#4285F4;stop-opacity:1"></stop><stop offset="100%" style="stop-color:#8AB4F8;stop-opacity:1"></stop></linearGradient><linearGradient id="sessionHighlight" x1="0" y1="0" x2="0" y2="100%"><stop offset="0" style="stop-color:#4285F4;stop-opacity:.2"></stop><stop offset="100%" style="stop-color:#4285F4;stop-opacity:.05"></stop></linearGradient></defs><rect width="928" height="100" fill="url(#sessionGradient)"></rect><g transform="translate(40, 25)"><circle cx="25" cy="25" r="25" fill="url(#geminiGradient)" opacity=".9"></circle><path d="M25 15 35 25 25 35 15 25z" fill="#fff" opacity=".9"></path><circle cx="25" cy="25" r="4" fill="#fff"></circle></g><text x="90" y="60" font-family="Arial" font-size="36" font-weight="bold" fill="#2d3748" filter="url(#glow)">Session</text> <g transform="translate(240, 12)"><rect x="0" y="0" width="260" height="76" rx="5" fill="#fff" stroke="#cbd5e0" stroke-width="1"></rect><rect x="0" y="0" width="260" height="22" rx="5" fill="#f7fafc"></rect><circle cx="12" cy="11" r="3.5" fill="#ff5f56"></circle><circle cx="26" cy="11" r="3.5" fill="#ffbd2e"></circle><circle cx="40" cy="11" r="3.5" fill="#27c93f"></circle><text x="55" y="15" font-family="monospace" font-size="9" fill="#4a5568">Session Browser</text> <g transform="translate(8, 30)" font-family="monospace" font-size="8"><rect x="0" y="0" width="244" height="12" fill="url(#sessionHighlight)" rx="2"></rect><text y="8" fill="#4285f4" font-weight="bold">▸ refactor-auth (2 days ago)</text> <text y="20" fill="#5f6368">feature-api (5 hours ago)</text> <text y="32" fill="#5f6368">bug-fix-mem (Just now)</text></g> <rect x="12" y="38" width="6" height="10" fill="#4285f4" opacity=".8"></rect></g><g transform="translate(540, 20)"><line x1="0" y1="30" x2="180" y2="30" stroke="#cbd5e0" stroke-width="2"></line><g id="session1"><circle cx="0" cy="30" r="8" fill="#4285f4" opacity=".8"></circle><text x="-5" y="50" font-family="Arial" font-size="8" fill="#5f6368">Day 1</text></g> <g id="session2"><circle cx="60" cy="30" r="8" fill="#34a853" opacity=".8"></circle><text x="53" y="50" font-family="Arial" font-size="8" fill="#5f6368">Day 2</text> <path d="M8 30q26-15 52 0" fill="none" stroke="#34a853" stroke-width="2" opacity=".4"></path></g><g id="session3"><circle cx="120" cy="30" r="8" fill="#fbbc04" opacity=".8"></circle><text x="113" y="50" font-family="Arial" font-size="8" fill="#5f6368">Day 3</text> <path d="M68 30q26-15 52 0" fill="none" stroke="#fbbc04" stroke-width="2" opacity=".4"></path></g><g id="sessionCurrent"><circle cx="180" cy="30" r="10" fill="#ea4335" opacity=".9"></circle><text x="168" y="50" font-family="Arial" font-size="8" fill="#5f6368" font-weight="bold">Now</text> <path d="M128 30q26-15 52 0" fill="none" stroke="#ea4335" stroke-width="2" opacity=".5"></path></g></g><g transform="translate(760, 20)"><g opacity=".6"><path d="M0 10V35H40V10H15L12 5H0z" fill="#fbbc04" stroke="#f9ab00" stroke-width="1"></path><text x="5" y="28" font-family="Arial" font-size="7" fill="#5f6368">project-a</text></g> <g transform="translate(50, 0)"><path d="M0 10V35H40V10H15L12 5H0z" fill="#4285f4" stroke="#1967d2" stroke-width="1.5"></path><text x="5" y="28" font-family="Arial" font-size="7" fill="#fff" font-weight="bold">project-b</text></g> <g transform="translate(100, 0)" opacity=".6"><path d="M0 10V35H40V10H15L12 5H0z" fill="#34a853" stroke="#188038" stroke-width="1"></path><text x="5" y="28" font-family="Arial" font-size="7" fill="#5f6368">project-c</text></g></g> <g transform="translate(240, 92)"><text font-family="monospace" font-size="9" fill="#8ab4f8">$ gemini --resume</text></g><g opacity=".3"><circle cx="220" cy="20" r="3" fill="#4285f4"></circle><circle cx="520" cy="75" r="2.5" fill="#34a853"></circle><circle cx="740" cy="65" r="2" fill="#fbbc04"></circle></g><g transform="translate(900, 30)"><circle cx="0" cy="0" r="6" fill="#34a853" opacity=".2"></circle><circle cx="0" cy="0" r="4" fill="#34a853"></circle><path d="M-2-1 0 2 2-2" fill="none" stroke="#fff" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path></g></svg>

## 自动保存：无需手动操作

Gemini CLI 提供了 **完全自动的会话保存机制** 。不需要记住任何保存命令，也不需要担心忘记保存，每次交互都会自动记录。

### 保存的内容

每个会话中会自动保存以下内容：

- **完整的对话历史** ：你的所有提示词和模型的响应
- **工具执行记录** ：所有工具调用的输入和输出
- **Token 使用统计** ：输入、输出、缓存的 Token 数量
- **推理过程摘要** ：Assistant 的思考过程（如果可用）

### 存储位置和范围

会话数据存储在本地目录：

```bash
1~/.gemini/tmp/<project_hash>/chats/
```

这里有一个重要特性： **会话是项目特定的** 。 `<project_hash>` 是根据当前工作目录生成的哈希值，这意味着：

- 在 `/home/user/project-a` 运行 Gemini CLI 的会话历史
- 和在 `/home/user/project-b` 运行的会话历史是完全独立的
- 切换目录会自动切换到对应项目的会话历史

这个设计避免了不同项目的上下文混淆，非常实用。

## 恢复会话

Gemini CLI 提供了灵活的会话恢复方式，既可以从命令行启动时恢复，也可以在交互界面中选择。

### 方式 1：命令行恢复最新会话

最简单的方式是使用 `--resume` （或 `-r` ）标志：

```bash
1gemini --resume
```

这会立即加载最近的一次会话，恢复所有对话历史。

### 方式 2：按索引恢复特定会话

如果你想恢复某个特定的历史会话，首先需要列出所有可用的会话：

```bash
1gemini --list-sessions
```

输出示例：

```text
1Available sessions for this project (3):
2
3  1. Fix bug in auth (2 days ago) [a1b2c3d4]
4  2. Refactor database schema (5 hours ago) [e5f67890]
5  3. Update documentation (Just now) [abcd1234]
```

然后使用索引号恢复：

```bash
1gemini --resume 1
```

这会恢复第一个会话（Fix bug in auth）。

### 方式 3：按会话 ID 恢复

如果你记录了会话的完整 UUID，也可以直接使用：

```bash
1gemini --resume a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

这种方式在脚本或自动化场景中特别有用。

## 交互式会话浏览器

Gemini CLI 还提供了一个非常直观的交互式界面来管理会话。在运行中的 CLI 会话里，输入：

```text
1/resume
```

这会打开 **会话浏览器（Session Browser）** ，提供以下功能：

### 浏览和预览

在浏览器中，你可以：

- **滚动查看** 所有历史会话列表
- **预览详情** ：查看会话日期、消息数量、第一个用户提示等信息
- **快速识别** ：通过会话 ID 和简短描述找到目标会话

### 搜索功能

支持输入关键词来过滤会话。这在有很多历史会话时特别有用。

### 选择和恢复

使用方向键导航到想要的会话，按 `Enter` 即可立即恢复该会话的完整上下文。

## 删除不需要的会话

随着时间推移，你可能会积累很多会话记录。Gemini CLI 提供了删除功能来清理不需要的历史。

### 从命令行删除

使用 `--delete-session` 标志加上索引号或会话 ID：

```bash
1gemini --delete-session 2
```

这会删除第二个会话。

### 从会话浏览器删除

1. 使用 `/resume` 打开会话浏览器
2. 导航到要删除的会话
3. 按 `x` 键

## 配置会话管理

你可以在 `settings.json` 中配置会话管理的行为，包括自动清理策略和会话长度限制。

### 自动清理旧会话

为了防止会话历史无限增长，可以启用自动清理策略：

```json
1{
2  "general": {
3    "sessionRetention": {
4      "enabled": true,
5      "maxAge": "30d",
6      "maxCount": 50
7    }
8  }
9}
```

配置说明：

- **`enabled`** ：是否启用自动清理（默认 `false` ）
- **`maxAge`** ：会话保留时长，支持格式如 “24h”、“7d”、“4w”
- **`maxCount`** ：最多保留的会话数量，超过此数量会删除最老的会话
- **`minRetention`** ：最小保留期（安全限制），默认 “1d”，新于此期限的会话永远不会被自动删除

例如，上面的配置表示：

- 保留最近 30 天的会话
- 或最多保留 50 个会话（以先触发的为准）
- 但至少保留最近 1 天的会话（即使超过 50 个）

### 限制单个会话长度

还可以限制单个会话的轮次（turn）数，防止上下文窗口过长导致成本过高：

```json
1{
2  "model": {
3    "maxSessionTurns": 100
4  }
5}
```
- **`maxSessionTurns`** ：单个会话允许的最大轮次数（一轮 = 一次用户输入 + 一次模型响应）
- 设置为 `-1` 表示不限制（默认）

**达到限制时的行为** ：

- **交互模式** ：CLI 显示提示信息，停止发送请求，你需要手动开始新会话
- **非交互模式** ：CLI 退出并返回错误码

## 最佳实践

### 1\. 善用项目目录隔离

为不同的任务或项目创建独立的目录，充分利用项目特定会话：

```bash
1~/projects/feature-auth/     # 认证功能相关讨论
2~/projects/bug-memory-leak/  # 内存泄漏调试
3~/projects/refactor-db/      # 数据库重构
```

### 2\. 定期清理历史

即使启用了自动清理，我也建议定期手动检查：

```bash
1gemini --list-sessions
```

删除明显不需要的会话，保持列表简洁。

### 3\. 合理设置保留策略

根据工作习惯配置清理策略：

- **短期项目** ： `"maxAge": "7d"` + `"maxCount": 20`
- **长期项目** ： `"maxAge": "60d"` + `"maxCount": 100`
- **实验性工作** ： `"maxAge": "3d"` + `"maxCount": 10`

### 4\. 控制会话长度

对于复杂的长期讨论，主动管理会话长度：

- 达到 30-50 轮对话时，考虑总结当前进展
- 开始新会话时，简要说明上下文和目标
- 避免单个会话变成"万能对话"

### 5\. 使用描述性的初始提示

会话列表会显示第一个用户提示，所以让它有意义：

**好的示例** ：

```text
1"重构 auth.js 模块，解决性能问题"
2"调查 /api/users 端点的内存泄漏"
```

**不好的示例** ：

```text
1"hi"
2"帮我看看这个"
```

## 数据隐私和安全

### 本地存储

所有会话数据都存储在本地 `~/.gemini/` 目录：

- 不会自动上传到云端
- 文件权限默认仅当前用户可读写

### 清理敏感信息

如果会话中包含敏感信息（密钥、密码等），项目结束后记得清理：

### 团队协作注意事项

如果需要在团队中共享会话（例如用于培训或问题复现），注意：

1. 会话文件是 JSON 格式，可以手动复制
2. 检查内容是否包含敏感信息
3. 考虑编辑会话文件，移除敏感部分

## 总结

Gemini CLI 的会话管理系统通过自动保存、灵活恢复和智能清理，解决了 AI 辅助编程中的上下文持久化问题。关键特性如下：

1. **自动保存** ：无需手动操作，所有对话自动记录
2. **项目隔离** ：不同目录的会话完全独立，避免上下文混淆
3. **灵活恢复** ：支持命令行快速恢复和交互式浏览器选择
4. **智能清理** ：可配置的自动清理策略，平衡历史保留和存储空间
5. **会话控制** ：可限制单个会话长度，控制成本和复杂度

合理使用这些功能，能让多日、多项目的 AI 辅助开发工作流变得更加流畅和高效。

## 参考资料

- [Gemini CLI Session Management 官方文档](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/session-management.md)
- [Gemini CLI 命令参考](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/commands.md)