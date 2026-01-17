# 🚀 快速开始：微博热搜分析迁移到 GitHub Actions

## 3 分钟完成配置

### 第一步：配置 Secrets (2个必需)

访问你的 GitHub 仓库：**Settings** > **Secrets and variables** > **Actions**

点击 **New repository secret**，添加以下两个密钥：

#### 1️⃣ ANTHROPIC_API_KEY
- **获取地址**: https://console.anthropic.com/
- **格式**: `sk-ant-api03-xxxxx...`
- **用途**: Claude AI 分析引擎

#### 2️⃣ TIANAPI_WEIBO_API_KEY
- **获取地址**: https://www.tianapi.com/
- **格式**: `476be10200b170992a1708e68b8a8775` (示例)
- **用途**: 抓取微博热搜数据
- **免费额度**: 100次/天

---

### 第二步：推送 Workflow 文件

```bash
# 进入项目目录
cd D:\project\1128

# 添加文件
git add .github/workflows/weibo-analysis-scheduled.yml

# 提交
git commit -m "feat: add GitHub Actions workflow for scheduled weibo analysis"

# 推送
git push origin main
```

---

### 第三步：验证和测试

1. 打开 GitHub 仓库页面，点击 **Actions** 标签
2. 找到 `Weibo Analysis - Scheduled (Beijing Time 6AM & 6PM)`
3. 点击 **Run workflow** > 选择分支 `main` > 点击 **Run workflow**
4. 等待执行完成（约 2-5 分钟）
5. 在页面底部 **Artifacts** 下载报告

---

## ⏰ 定时执行时间

- **早上 6:00** (北京时间) = UTC 22:00 前一天
- **晚上 18:00** (北京时间) = UTC 10:00 当天

---

## 📦 生成的文件

每次执行会生成 3 个文件：

```
weibo-hotspot-report-XXX/
├── 微博热搜分析-2025-01-17.html      # 📊 可视化报告（双击打开）
├── analysis_summary.txt               # 📝 AI 总结
└── creative_analysis_results.json     # 📄 原始数据
```

---

## ✅ 验证成功的标志

在 Actions 执行日志中看到：

```
✅ 热搜数据抓取成功
✅ 解析和格式化数据
✅ AI创意分析完成
✅ HTML报告生成成功
✅ 分析总结已保存
```

---

## ❓ 常见问题

### Q1: Secrets 配置后看不到具体值？
**A**: 这是正常的，GitHub 加密存储，配置后只显示名称。

### Q2: Workflow 不自动触发？
**A**: 检查 Settings > Actions > General，确保 Actions 已启用。

### Q3: API Key 无效？
**A**:
- ANTHROPIC_API_KEY 格式：`sk-ant-api03-...`
- TIANAPI_WEIBO_API_KEY 需要天行数据账户实名认证

### Q4: 想立即测试，不等定时？
**A**: 使用 **Run workflow** 按钮手动触发。

---

## 📚 详细文档

查看完整迁移指南：[MIGRATION_GUIDE_WEIBO_ANALYSIS.md](./MIGRATION_GUIDE_WEIBO_ANALYSIS.md)

包含：
- 技术可行性分析
- 改造方案详解
- 完整配置流程
- 故障排查指南
- 高级功能配置

---

## 💰 成本估算

| 项目 | 费用 |
|------|------|
| GitHub Actions | 免费（公开仓库） |
| Claude API | $10-30/月 |
| 天行数据 API | 免费（100次/天） |
| **总计** | **$10-30/月** |

---

## 🎯 下一步

- [ ] 配置 Secrets
- [ ] 推送 Workflow
- [ ] 手动测试执行
- [ ] 等待定时触发
- [ ] 查看生成报告

**祝你配置顺利！🎉**
