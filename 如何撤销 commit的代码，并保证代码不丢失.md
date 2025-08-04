在 Git 中撤销 commit 但保留代码修改的完整解决方案如下：

# 1. **撤销最近一次 commit，保留修改到工作区（最常用）**
```bash
# 撤销 commit 但保留所有代码修改（回到未暂存状态）
git reset --soft HEAD~1
```
- 效果：代码修改保留在工作目录，commit 被撤销
- 使用场景：重新修改后再次提交

# 2. **撤销 commit 但保留修改到暂存区**
```bash
# 撤销 commit 但保留修改到暂存区（相当于已 git add 的状态）
git reset --mixed HEAD~1  # --mixed 是默认参数，可省略
```
- 效果：代码修改保留在暂存区（绿色状态），commit 被撤销

# 3. **撤销多个 commit**
```bash
# 撤销最近 3 次 commit，保留修改到工作区
git reset --soft HEAD~3
```

# 4. **已推送到远程仓库的撤销（强制推送）**
```bash
# 1. 本地撤销 commit
git reset --soft HEAD~1

# 2. 强制推送到远程（覆盖历史）
git push --force origin 分支名
```
⚠️ 警告：仅限个人分支使用！公共分支慎用，会覆盖他人历史记录

---

# 完整工作流示例：
```bash
# 1. 意外提交了不需要的 commit
git commit -m "错误的提交"

# 2. 撤销 commit 但保留代码修改
git reset --soft HEAD~1

# 3. 查看状态（应显示修改文件未暂存）
git status

# 4. 修改文件后重新提交
git add .
git commit -m "正确的新提交"

# 5. 推送到远程（如之前未推送过）
git push origin main
```

# 替代方案（保留历史记录）：
```bash
# 用新 commit 撤销旧 commit（保留历史痕迹）
git revert HEAD
```
- 效果：创建新的反向 commit 抵消之前的修改
- 适用场景：公共分支/需要保留撤销记录的情况

# 重要提示：
1. 使用 `git reflog` 可找回误删的 commit
2. `--soft` 和 `--mixed` 的区别：
   - `--soft`：修改保留在暂存区（git add 后的状态）
   - `--mixed`（默认）：修改保留在工作目录（未 git add 的状态）
3. 强制推送前确保：
   ```bash
   # 查看远程分支状态
   git fetch origin
   git status
   ```

> 📌 **最佳实践**：  
> - 未推送的 commit → 用 `git reset --soft`  
> - 已推送的 commit → 用 `git revert`（安全）或 `reset + force push`（谨慎）  
> - 操作前用 `git log --oneline` 确认 commit 位置
> - 




在 IntelliJ IDEA 中使用 Git 撤销 commit 并保留修改的完整操作指南：

# 🛠️ 图形界面操作步骤（推荐）

1. **打开 Git 日志**
   - 顶部菜单：`View → Tool Windows → Git`
   - 或使用快捷键：`Alt+9` (Windows/Linux) / `Command+9` (Mac)

2. **定位要撤销的 commit**
   - 在左侧 "Log" 标签页中，找到要撤销的 commit
   - **右键点击** 该 commit → 选择 `Undo Commit`

   ![IDEA Undo Commit](https://resources.jetbrains.com/help/img/idea/2023.3/undo_commit.png)

3. **确认撤销操作**
   - 修改会自动保留在工作区（未暂存状态）
   - 在 "Local Changes" 标签页可查看所有保留的修改

4. **修改代码**
   - 直接编辑文件，IDEA 会自动检测变更

5. **重新提交**
   - 在 "Commit" 工具窗口（`Ctrl+K`/`Cmd+K`）
   - 勾选要提交的文件 → 填写提交信息 → 点击 "Commit"

# ⌨️ 命令行操作（Terminal 内）
```bash
# 1. 撤销最近一次 commit（保留修改）
git reset --soft HEAD~1

# 2. 修改代码...

# 3. 重新添加并提交
git add .  # 或添加特定文件
git commit -m "新的提交信息"
```

# ⚠️ 特殊情况处理

**场景1：已推送到远程仓库**
```bash
# 1. 本地撤销
git reset --soft HEAD~1

# 2. 修改代码后重新提交
git add .
git commit -m "修正后的提交"

# 3. 强制推送（覆盖远程）
git push --force origin 分支名
```
> **警告**：仅限个人分支！团队协作分支用 `git revert`

**场景2：撤销特定 commit（非最近）**
```bash
# 1. 找到要撤销的 commit ID（git log查看）
git log --oneline

# 2. 回退到该 commit 之前（保留修改）
git reset --soft <commit-id>
```

# 🔄 替代方案：Revert Commit（保留历史记录）
1. 右键点击 commit → `Revert Commit`
2. IDEA 会自动创建抵消修改的新 commit
3. 适合公共分支，避免强制推送风险

# 💡 最佳实践提示
1. **撤销前备份**：重要修改可先创建临时分支
   ```bash
   git branch temp-backup
   ```
2. **修改保留位置**：
   - `--soft`：修改保留在暂存区（相当于已 `git add`）
   - `--mixed`（默认）：修改保留在工作目录（需重新 `git add`）
3. **找回误删 commit**：
   - 使用 `git reflog` 查看操作历史
   - 用 `git reset --hard <ref>` 恢复

### 📊 撤销方式对比表
| **方法**         | 适用场景                     | 是否修改历史 | 风险等级 |
|------------------|----------------------------|-------------|---------|
| `Undo Commit`    | 本地未推送的提交             | ✅ 修改      | ⭐ 低    |
| `git reset --soft`| 精确控制撤销位置             | ✅ 修改      | ⭐⭐ 中  |
| `Revert Commit`  | 已推送/公共分支              | ❌ 保留      | ⭐ 低    |
| `--force push`   | 个人分支覆盖提交             | ✅ 修改      | ⭐⭐⭐ 高 |

> 操作完成后，可在 IDEA 右下角查看当前分支状态，确保代码修改完整保留：  





在 IntelliJ IDEA 中撤销最近一次的 Git commit 但保留代码更改（即撤销 commit 但不丢失代码），然后修改代码并重新提交，可以按照以下步骤操作：

# ✅ 目标：
- 撤销最近一次的 `commit`（即把 commit 撤销，回到“已修改但未提交”状态）
- 保留所有代码修改（不丢失）
- 修改代码后重新提交

---

# ✅ 方法一：使用 IDEA 的图形化界面（推荐）

## 步骤 1：打开 Git 工具窗口
1. 在 IDEA 右下角点击 **Git** 工具窗口（或通过菜单 `View → Tool Windows → Git`）。
2. 切换到 **Log** 标签页，查看提交历史。

## 步骤 2：找到你刚提交的 commit
- 在提交历史中，找到你刚刚提交的那条 commit。

## 步骤 3：执行 `Reset Current Branch to Here`
1. 右键点击该 commit。
2. 选择 **Reset Current Branch to Here...**
3. 弹出对话框后，选择：
   - **Soft**：保留工作区和暂存区的更改（推荐！）
     - 代码不会丢失，所有修改保留在“Changes”中，可以继续编辑并重新提交。
   - Mixed（默认）：保留工作区修改，但取消暂存（也可以）
   - Hard：**危险！会删除所有更改，不要选！**

✅ 推荐选择：**Soft**

## 步骤 4：修改代码
- 此时你会看到之前提交的文件重新出现在 **Local Changes** 中（未提交的更改）。
- 你可以继续修改代码，修复问题。

## 步骤 5：重新提交
1. 修改完成后，回到 **Commit** 窗口。
2. 选择要提交的文件，写好 commit 信息。
3. 点击 **Commit and Push** 或 **Commit**。

---

# ✅ 方法二：使用命令行（等效操作）

在 IDEA 的 Terminal 中执行：

```bash
git reset --soft HEAD~1
```

- `HEAD~1` 表示撤销最近一次 commit。
- `--soft` 表示保留代码修改在工作区中。
- 执行后，之前的更改会回到“未提交更改”状态。

然后：
1. 修改代码。
2. 重新 `git add .` 和 `git commit -m "new message"`。

---

# ⚠️ 注意事项
- 如果你已经 `git push` 到远程仓库，撤销 commit 后再次提交，需要使用 `git push --force`（强制推送），但要小心，避免影响他人。
- 如果团队协作，建议使用 `git revert`（生成一个反向提交）而不是 `reset`，避免历史混乱。

---

# ✅ 总结

| 操作 | 效果 |
|------|------|
| `git reset --soft HEAD~1` | 撤销最后一次 commit，保留代码修改 |
| 在 IDEA 中选择 **Reset → Soft** | 同上，图形化操作更安全 |
| 修改代码后重新提交 | 正常提交即可 |

这样你就可以安全地撤销 commit，修改代码，再重新提交，且不会丢失任何内容。

