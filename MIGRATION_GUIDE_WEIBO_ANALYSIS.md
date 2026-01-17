# 微博热搜分析 Skill 迁移到 GitHub Actions 完整指南

## 📋 目录

1. [技术可行性分析](#技术可行性分析)
2. [改造方案概述](#改造方案概述)
3. [GitHub Secrets 配置清单](#github-secrets-配置清单)
4. [一步步配置流程](#一步步配置流程)
5. [测试和验证](#测试和验证)
6. [常见问题排查](#常见问题排查)

---

## 技术可行性分析

### ✅ 完全可行

**当前 Skill 的工作流程：**

```
阶段1: 抓取热搜 (天行数据API)
  ↓
阶段2: 背景采集 (WebSearch)
  ↓
阶段3: AI创意分析 (Claude Agent SDK)
  ↓
阶段4: 生成HTML报告 (Python脚本)
```

**GitHub Actions 支持的能力：**

✅ **定时执行** - 通过 `cron` 表达式，支持每天早上6点和晚上6点运行
✅ **Claude Agent SDK** - 官方支持，可直接安装使用
✅ **HTTP API 调用** - 支持 curl/requests 调用天行数据API
✅ **Python 环境** - 内置 Python 3.11，可安装所需依赖
✅ **文件存储** - 支持生成报告并作为 artifacts 上传
✅ **自动提交** - 可以将生成的报告自动提交回仓库

### 📊 对比分析

| 功能 | 本地 Claude Code | GitHub Actions | 迁移难度 |
|------|-----------------|----------------|---------|
| 定时执行 | ❌ 需手动触发 | ✅ cron 自动执行 | 简单 |
| API 调用 | ✅ WebFetch/Bash | ✅ curl/requests | 简单 |
| Claude SDK | ✅ 内置支持 | ✅ 需安装 | 简单 |
| 背景搜索 | ✅ WebSearch | ⚠️ 需改用 API | 中等 |
| 报告生成 | ✅ 本地保存 | ✅ Artifacts | 简单 |
| 通知推送 | ❌ 无 | ✅ 可配置 | 简单 |

---

## 改造方案概述

### 🎯 目标

- 每天北京时间早上6点和晚上6点自动执行分析
- 使用 Claude Agent SDK 进行 AI 创意分析
- 生成 HTML 报告并保存为 Artifacts
- 可选：自动提交报告到仓库

### 🔧 主要改造点

#### 1. **定时触发配置**

```yaml
on:
  schedule:
    # 北京时间早上6点 = UTC 22点（前一天）
    - cron: "0 22 * * *"
    # 北京时间晚上6点 = UTC 10点
    - cron: "0 10 * * *"
  workflow_dispatch:  # 支持手动触发
```

#### 2. **背景信息采集改造**

原方案使用 `WebSearch` 工具，需要改为：
- 使用搜索 API（如 Serper API、Tavily API）
- 或简化为基于热搜标题的 AI 推理生成背景

**推荐方案：使用 Claude Agent SDK 的 `query` 方法**

```python
from claude_agent_sdk import query

async def collect_background(title):
    prompt = f"请分析微博热搜话题 {title} 的背景信息，包括事件脉络、关键人物、公众反应等"
    async for chunk in query(prompt=prompt):
        # 处理返回的背景信息
        pass
```

#### 3. **依赖安装**

```yaml
- name: Install Python dependencies
  run: |
    pip install claude-agent-sdk anthropic anyio httpx requests
```

#### 4. **Secrets 管理**

所有敏感信息通过 GitHub Secrets 配置：
- `ANTHROPIC_API_KEY` - Claude API 密钥
- `TIANAPI_WEIBO_API_KEY` - 天行数据 API 密钥

---

## GitHub Secrets 配置清单

### 必需配置 (2个)

| Secret 名称 | 用途 | 获取方式 | 示例值 |
|------------|------|---------|--------|
| `ANTHROPIC_API_KEY` | Claude API 密钥 | [Anthropic Console](https://console.anthropic.com/) | `sk-ant-api03-...` |
| `TIANAPI_WEIBO_API_KEY` | 天行数据微博热搜API | [天行数据](https://www.tianapi.com/) | `476be10200b170992a1708e68b8a8775` |

### 可选配置 (用于增强功能)

| Secret 名称 | 用途 | 获取方式 |
|------------|------|---------|
| `SERPER_API_KEY` | Google 搜索 API (背景信息增强) | [Serper.dev](https://serper.dev/) |
| `TAVILY_API_KEY` | Tavily 搜索 API (背景信息增强) | [Tavily](https://tavily.com/) |

### 🔍 详细说明

#### 1. ANTHROPIC_API_KEY

**用途：** 调用 Claude API 进行 AI 创意分析和总结

**获取步骤：**
1. 访问 https://console.anthropic.com/
2. 登录或注册账号
3. 进入 Settings > API Keys
4. 点击 "Create Key" 创建新密钥
5. 复制密钥（格式：`sk-ant-api03-...`）

**使用场景：**
- 阶段2: 背景信息推理分析
- 阶段3: AI 创意分析和评分
- 最终总结生成

**费用：**
- API 调用按 token 计费
- 预估每次分析约消耗 50K-100K tokens
- 每天2次执行，月费用约 $10-30

#### 2. TIANAPI_WEIBO_API_KEY

**用途：** 抓取微博实时热搜榜单数据

**获取步骤：**
1. 访问 https://www.tianapi.com/
2. 注册账号并实名认证
3. 在 "接口市场" 搜索 "微博热搜"
4. 申请接口（免费版每天100次）
5. 在 "我的接口" 查看 API Key

**API 信息：**
- 接口地址: `https://apis.tianapi.com/weibohot/index`
- 请求方式: GET
- 认证方式: URL 参数 `key=YOUR_API_KEY`
- 返回格式: JSON (TOP 50 热搜)
- 免费额度: 100次/天

**使用场景：**
- 阶段1: 抓取微博热搜数据

---

## 一步步配置流程

### 步骤 1: 配置 GitHub Secrets

#### 1.1 进入仓库设置

1. 打开你的 GitHub 仓库
2. 点击 **Settings** (设置)
3. 在左侧菜单找到 **Secrets and variables** > **Actions**

#### 1.2 添加 ANTHROPIC_API_KEY

1. 点击 **New repository secret**
2. Name: `ANTHROPIC_API_KEY`
3. Secret: 粘贴你的 Claude API 密钥 (格式: `sk-ant-api03-...`)
4. 点击 **Add secret**

#### 1.3 添加 TIANAPI_WEIBO_API_KEY

1. 点击 **New repository secret**
2. Name: `TIANAPI_WEIBO_API_KEY`
3. Secret: 粘贴你的天行数据 API Key
4. 点击 **Add secret**

**验证截图位置：**
```
Settings > Secrets and variables > Actions
应该看到:
✅ ANTHROPIC_API_KEY
✅ TIANAPI_WEIBO_API_KEY
```

---

### 步骤 2: 推送 Workflow 文件到仓库

#### 2.1 确认文件已创建

确保以下文件存在：
```
.github/workflows/weibo-analysis-scheduled.yml
```

#### 2.2 提交和推送

**方式 1: 使用 Git 命令行**

```bash
# 进入项目目录
cd D:\project\1128

# 检查状态
git status

# 添加 workflow 文件
git add .github/workflows/weibo-analysis-scheduled.yml

# 提交
git commit -m "feat: add GitHub Actions workflow for scheduled weibo analysis"

# 推送到远程仓库
git push origin main
```

**方式 2: 使用 Claude Code**

让我帮你执行 git 操作（需要你确认）

**方式 3: 使用 GitHub Desktop**

1. 打开 GitHub Desktop
2. 选择你的仓库
3. 在 "Changes" 中勾选 workflow 文件
4. 填写 commit 信息: `feat: add GitHub Actions workflow for scheduled weibo analysis`
5. 点击 "Commit to main"
6. 点击 "Push origin"

---

### 步骤 3: 验证 Workflow 已激活

#### 3.1 检查 Actions 页面

1. 在 GitHub 仓库页面，点击 **Actions** 标签
2. 在左侧 "Workflows" 列表中，应该看到:
   - `Weibo Analysis - Scheduled (Beijing Time 6AM & 6PM)`

#### 3.2 手动触发测试

1. 点击 workflow 名称
2. 点击右上角 **Run workflow** 下拉按钮
3. 选择分支（通常是 `main`）
4. 点击绿色的 **Run workflow** 按钮

#### 3.3 查看执行日志

1. 刷新页面，会看到新的 workflow run
2. 点击进入，查看各个步骤的执行日志
3. 确认每个阶段都成功执行 ✅

---

### 步骤 4: 下载和查看报告

#### 4.1 下载 Artifacts

1. 在 workflow run 页面底部，找到 **Artifacts** 部分
2. 点击下载 `weibo-hotspot-report-XXX`
3. 解压 zip 文件

#### 4.2 查看生成的文件

```
weibo-hotspot-report-XXX/
├── 微博热搜分析-2025-01-17.html  # HTML 可视化报告
├── analysis_summary.txt           # Claude AI 生成的总结
└── creative_analysis_results.json # 原始分析数据
```

#### 4.3 打开 HTML 报告

双击 `微博热搜分析-YYYY-MM-DD.html` 文件，在浏览器中查看完整报告。

---

## 测试和验证

### ✅ 功能测试清单

#### 1. 手动触发测试

- [ ] Workflow 可以手动触发
- [ ] 所有步骤成功执行
- [ ] 生成了 HTML 报告
- [ ] Artifacts 可以下载

#### 2. 定时任务测试

- [ ] 等待下一个定时时间点（早上6点或晚上6点北京时间）
- [ ] Workflow 自动触发
- [ ] 执行成功并生成报告

#### 3. 数据质量测试

- [ ] 热搜数据抓取成功（10条）
- [ ] 背景信息完整
- [ ] AI 创意分析有意义
- [ ] HTML 报告样式正常
- [ ] Claude 总结内容准确

#### 4. 错误处理测试

- [ ] API Key 错误时有明确提示
- [ ] 网络失败时能正确报错
- [ ] 部分数据缺失时能继续执行

---

## 常见问题排查

### ❌ 问题 1: Workflow 没有自动触发

**可能原因：**
- 仓库处于私有状态，GitHub Actions 额度用完
- cron 表达式配置错误
- 主分支名称不是 `main`

**解决方案：**
```bash
# 检查仓库设置
Settings > Actions > General
确保 "Actions permissions" 开启

# 检查 cron 表达式
北京时间 6:00 = UTC 22:00 (前一天)
北京时间 18:00 = UTC 10:00 (当天)
```

---

### ❌ 问题 2: ANTHROPIC_API_KEY 无效

**错误信息：**
```
Error: Invalid API key
```

**解决方案：**
1. 检查 Secret 名称是否为 `ANTHROPIC_API_KEY` (大写)
2. 检查 API Key 格式是否为 `sk-ant-api03-...`
3. 在 Anthropic Console 验证 API Key 是否有效
4. 确保 API Key 有足够的额度

---

### ❌ 问题 3: 天行数据 API 调用失败

**错误信息：**
```
{"code": 190, "msg": "APIKEY不可用"}
```

**解决方案：**
1. 检查 API Key 是否正确
2. 确认天行数据账户已实名认证
3. 检查每日调用次数是否超过 100 次
4. 访问天行数据控制台查看接口状态

---

### ❌ 问题 4: Python 依赖安装失败

**错误信息：**
```
ERROR: Could not find a version that satisfies the requirement claude-agent-sdk
```

**解决方案：**
```yaml
# 在 workflow 中添加 pip 缓存
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.11"
    cache: 'pip'

# 升级 pip
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install claude-agent-sdk anthropic anyio httpx
```

---

### ❌ 问题 5: HTML 报告样式错乱

**可能原因：**
- JSON 数据格式不正确
- 文件编码问题

**解决方案：**
1. 检查 `creative_analysis_results.json` 格式
2. 确保所有 Python 脚本使用 `utf-8` 编码
3. 查看 workflow 日志中的错误信息

---

## 高级配置

### 📧 配置通知推送（可选）

#### 方案 1: 钉钉机器人通知

```yaml
- name: Send DingTalk notification
  if: always()
  env:
    DINGTALK_WEBHOOK: ${{ secrets.DINGTALK_WEBHOOK }}
  run: |
    curl -X POST $DINGTALK_WEBHOOK \
      -H 'Content-Type: application/json' \
      -d '{
        "msgtype": "text",
        "text": {
          "content": "微博热搜分析完成！共分析了 10 条热搜，报告已生成。"
        }
      }'
```

#### 方案 2: 企业微信通知

```yaml
- name: Send WeChat Work notification
  env:
    WECHAT_WEBHOOK: ${{ secrets.WECHAT_WEBHOOK }}
  run: |
    curl -X POST $WECHAT_WEBHOOK \
      -H 'Content-Type: application/json' \
      -d '{
        "msgtype": "markdown",
        "markdown": {
          "content": "## 微博热搜分析完成\n\n优秀: 3个\n良好: 5个\n一般: 2个"
        }
      }'
```

---

### 🗂️ 自动归档报告（可选）

将报告按日期自动提交到仓库：

```yaml
- name: Commit and push results
  if: github.event_name == 'schedule'
  run: |
    git config --local user.email "github-actions[bot]@users.noreply.github.com"
    git config --local user.name "GitHub Actions Bot"

    # 创建日期目录
    DATE_DIR="reports/$(date +%Y-%m)"
    mkdir -p $DATE_DIR

    # 移动报告到日期目录
    mv resources/微博热搜分析-*.html $DATE_DIR/

    git add reports/
    git commit -m "chore: add weibo analysis report $(date +%Y-%m-%d)"
    git push
```

---

## 总结

### ✅ 迁移完成后的优势

| 特性 | 说明 |
|------|------|
| 🤖 全自动执行 | 每天早晚6点自动运行，无需人工干预 |
| ☁️ 云端运行 | 不占用本地资源，随时随地查看结果 |
| 📊 历史记录 | 所有执行记录和报告都保存在 GitHub |
| 🔔 通知推送 | 可配置钉钉/企业微信通知 |
| 📦 报告归档 | Artifacts 自动保存 30 天 |
| 🔒 安全可靠 | Secrets 加密存储，不会泄露 |

### 📅 定时执行计划

- **早上 6:00** (北京时间) - 捕捉昨夜到清晨的热搜
- **晚上 18:00** (北京时间) - 捕捉白天的热搜

### 💰 成本估算

| 项目 | 费用 | 说明 |
|------|------|------|
| GitHub Actions | $0 | 公开仓库免费，私有仓库每月 2000 分钟免费 |
| Claude API | $10-30/月 | 每天2次执行，每次约 50K-100K tokens |
| 天行数据 API | $0 | 免费版 100次/天足够使用 |
| **总计** | **$10-30/月** | 主要成本是 Claude API |

---

## 下一步

完成配置后，你可以：

1. ✅ 等待第一次定时执行（早上6点或晚上6点）
2. 📧 配置通知推送，第一时间获取报告
3. 📂 设置报告自动归档到仓库
4. 📊 基于历史数据进行趋势分析
5. 🎨 自定义 HTML 报告样式

如有问题，请查看：
- GitHub Actions 执行日志
- 本文档的 "常见问题排查" 部分
- 提交 Issue 到项目仓库
