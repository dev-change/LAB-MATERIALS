# 仓库维护手册

> 本文档面向仓库维护人（管理员）。

---

## 仓库信息

| 项目 | 内容 |
|------|------|
| 地址 | https://github.com/dev-change/LAB-MATERIALS |
| 类型 | Public |
| 提交方式 | Fork + Pull Request |

---

## 分支策略

### 分支说明

| 分支 | 用途 | 保护状态 |
|------|------|---------|
| `main` | 精选资料，面向所有人长期可用 | 建议开启保护 |
| `2024` | 2024 届资料存档 | 可选保护 |
| `2025` | 2025 届资料存档 | 可选保护 |
| `2026` | 2026 届资料存档 | 可选保护 |

### 合并规则

收到 PR 后，根据资料质量决定合并目标：

| 质量评级 | 合并目标 | 标准 |
|---------|---------|------|
| ⭐⭐⭐ 高 | `main` | 内容准确、结构清晰、通用性强、可长期复用 |
| ⭐⭐ 中 | `main` 或对应届分支 | 内容不错但有局限，或属于特定届的特色资料 |
| ⭐ 一般 | 对应届分支 | 内容尚可但不具备通用性，或格式不够规范 |

**操作**：在 PR 页面选择 "Squash and merge"，然后选择目标分支。如果 PR 目标是 main 但你想合并到届分支，可以让提交者重新提 PR，或自己 cherry-pick。

---

## GitHub 仓库设置建议

### 1. 分支保护（推荐开启）

Settings → Branches → Add rule：

- **Branch name pattern**: `main`
- ✅ **Require a pull request before merging**
  - ✅ **Require approvals**: 1（至少一个人 review）
- ✅ **Require status checks to pass before merging**（可选）
- ✅ **Restrict pushes that create files larger than 100 MB**（GitHub 自带）

### 2. 自动删除分支

Settings → General → Pull Requests：
- ✅ **Automatically delete head branches**

合并后自动删除 feature 分支，保持仓库整洁。

### 3. PR 模板（可选）

在仓库根目录创建 `.github/pull_request_template.md`：

```markdown
## 提交内容

- [ ] 新增资料
- [ ] 修正错误
- [ ] 其他

## 说明

简要描述提交了什么、为什么提交。

## 质量自评

- [ ] 内容经过验证
- [ ] 格式规范（Markdown）
- [ ] 文件名和路径正确
```

---

## 日常维护操作

### 同步届分支

每届分支应定期从 main 同步通用更新：

```bash
git checkout 2026
git merge main
# 如有冲突，手动解决后提交
git push origin 2026
```

### 归档旧届分支

某届毕业后，将其分支设为只读（通过分支保护规则），不再接受新 PR。

### 清理 Fork

定期检查并删除自己的 Fork 中已合并的远程分支：

```bash
git remote prune origin
```

---

## 注意事项

1. **不要直接在 main 上 push**，始终通过 PR 合并
2. **合并方式**：优先使用 "Squash and merge"，保持主分支历史简洁
3. **定期 review**：积压的 PR 不要拖太久，每周集中处理一次
4. **质量把关**：宁缺毋滥，main 分支的内容代表仓库的面子

---

*最后更新：2026-05-09*
