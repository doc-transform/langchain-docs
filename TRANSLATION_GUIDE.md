
# LangChain 文档中文翻译指南

本项目 fork 自 [langchain-ai/langchain-docs](https://github.com/langchain-ai/langchain-docs)，目标是将英文文档翻译为中文。

---

## 📌 仓库远程配置

| 远程名 | 地址 | 用途 |
|--------|------|------|
| `origin` | `https://github.com/doc-transform/langchain-docs.git` | 你的 fork 仓库 |
| `upstream` | `https://github.com/langchain-ai/langchain-docs.git` | 官方原始仓库 |

```bash
# 如果尚未配置 upstream，执行：
git remote add upstream https://github.com/langchain-ai/langchain-docs.git
git fetch upstream
```

---

## 🌿 分支策略

采用**三层分支结构**，保证翻译工作与上游同步互不干扰：

```
upstream/main (官方最新)
  │
  ├── main              ← 保持与 upstream 同步，永远不在此翻译
  │
  └── transform         ← 翻译主分支，所有翻译最终合入这里
       ├── transform/langsmith    ← (可选) 按目录拆分的子任务分支
       ├── transform/oss          ← (可选) 按目录拆分的子任务分支
       └── ...
```

| 分支 | 用途 | 说明 |
|------|------|------|
| `main` | 保持与 upstream 同步 | **永远不要在这里翻译**，只用于 sync upstream |
| `transform` | 翻译主分支 | 所有翻译最终合入这里 |
| `transform/<目录名>` | 子任务分支（可选） | 按目录拆分翻译任务，完成后合入 `transform` |

### 创建翻译分支

```bash
git checkout main
git checkout -b transform
git push -u origin transform
```

---

## 🔄 同步 upstream 流程

当官方文档更新后，需要把更新合入翻译分支。推荐使用 **rebase** 保持提交历史干净。

### 步骤 1：同步 main

```bash
git checkout main
git fetch upstream
git rebase upstream/main
git push origin main --force-with-lease
```

### 步骤 2：把 main 的更新合入 transform

```bash
git checkout transform
git rebase main
# ⚠️ 这里可能有冲突（官方改了你已翻译的文件），需要手动解决
git push origin transform --force-with-lease
```

### 冲突解决原则

- **以你的翻译为主**
- 检查官方是否**新增了内容**，新增内容需要补翻
- 检查官方是否**删除了内容**，对应删除翻译
- 检查官方是否**修改了代码示例**，同步更新代码块

---

## 📝 翻译规范

### 需要翻译的内容

- 标题（h1 ~ h6）
- 段落正文
- 列表文案
- 表格中的文字内容
- 图片的 alt 文本
- frontmatter 中的 `title`、`description`（按需决定）

### 不翻译的内容

- **代码块**：` ```python ... ``` ` 中的代码保持原样
- **专有名词**：`LangChain`、`LangSmith`、`LangGraph`、`Agent`、`RAG`、`Chain`、`Retriever`、`Prompt`、`Token` 等
- **URL 链接**
- **MDX 组件名**：如 `<CodeBlock>`、`<Tabs>`、`<TabItem>` 等
- **frontmatter 中的路由字段**：`slug`、`sidebar_label`、`sidebar_position` 等
- **文件名和路径**

### 格式注意事项

- 中英文之间加一个空格，例如：`使用 LangChain 构建应用`
- 中文与数字之间加一个空格，例如：`共 3 个步骤`
- 全角标点和半角标点不要混用，中文语境下使用全角标点
- 保持原文的 Markdown / MDX 格式结构不变
- 保留原文的换行结构

---

## 📊 翻译进度追踪

在项目根目录维护 `translation-progress.json` 文件来管理翻译状态：

```json
{
  "upstream_commit": "基于的 upstream commit hash",
  "last_sync_date": "2026-03-13",
  "progress": {
    "src/langsmith/home.mdx": {
      "status": "done",
      "translator": "your-name",
      "date": "2026-03-13"
    },
    "src/langsmith/alerts.mdx": {
      "status": "in-progress",
      "translator": "your-name",
      "date": ""
    },
    "src/oss/quickstart.mdx": {
      "status": "todo",
      "translator": "",
      "date": ""
    }
  }
}
```

状态值说明：

| 状态 | 含义 |
|------|------|
| `todo` | 尚未翻译 |
| `in-progress` | 翻译进行中 |
| `done` | 翻译完成 |
| `needs-update` | 官方更新后需要补翻 |

---

## ✅ Git Commit 规范

使用前缀区分不同类型的提交：

| 前缀 | 含义 | 示例 |
|------|------|------|
| `translate:` | 完成翻译 | `translate: langsmith/home.mdx` |
| `translate-wip:` | 翻译进行中（未完成） | `translate-wip: langsmith/alerts.mdx` |
| `translate-fix:` | 修正翻译 | `translate-fix: langsmith/home.mdx 修正术语` |
| `sync:` | 同步上游变更 | `sync: merge upstream changes 2026-03-13` |
| `chore:` | 其他杂项 | `chore: update translation-progress.json` |

---

## ⚡ 日常工作流速查

### 开始翻译一个新文件

```bash
# 1. 确保在 transform 分支
git checkout transform

# 2. 翻译 .mdx 文件

# 3. 更新进度文件 translation-progress.json

# 4. 提交
git add .
git commit -m "translate: langsmith/home.mdx"
git push
```

### 定期同步上游

```bash
# 1. 同步 main
git checkout main
git fetch upstream
git rebase upstream/main
git push origin main --force-with-lease

# 2. 合入 transform
git checkout transform
git rebase main
# 解决可能的冲突...
git push origin transform --force-with-lease
```

### 使用子任务分支（可选）

```bash
# 创建子任务分支
git checkout transform
git checkout -b transform/langsmith

# 翻译完成后合入 transform
git checkout transform
git merge transform/langsmith
git branch -d transform/langsmith
```

---

## 💡 Tips

1. **优先翻译高频使用的文档**（如 quickstart、getting-started），价值最高
2. **每次翻译前先 fetch upstream**，确认文件没有大的变动，避免做无用功
3. **小步提交**，每翻译完一个文件就提交一次，方便冲突解决和回滚
4. **善用 `git diff` 检查上游变更**：`git diff main..upstream/main -- src/langsmith/` 可以查看某个目录的变更
5. **翻译时保留原文注释**（可选）：在翻译内容上方用 HTML 注释保留原文，方便对照
   ```mdx
   <!-- Original: This is a guide to getting started with LangChain -->
   这是 LangChain 的入门指南
   ```
