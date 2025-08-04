在 Git 中撤销 commit 但保留代码修改的完整解决方案如下：

### 1. **撤销最近一次 commit，保留修改到工作区（最常用）**
```bash
# 撤销 commit 但保留所有代码修改（回到未暂存状态）
git reset --soft HEAD~1
```
- 效果：代码修改保留在工作目录，commit 被撤销
- 使用场景：重新修改后再次提交

### 2. **撤销 commit 但保留修改到暂存区**
```bash
# 撤销 commit 但保留修改到暂存区（相当于已 git add 的状态）
git reset --mixed HEAD~1  # --mixed 是默认参数，可省略
```
- 效果：代码修改保留在暂存区（绿色状态），commit 被撤销

### 3. **撤销多个 commit**
```bash
# 撤销最近 3 次 commit，保留修改到工作区
git reset --soft HEAD~3
```

### 4. **已推送到远程仓库的撤销（强制推送）**
```bash
# 1. 本地撤销 commit
git reset --soft HEAD~1

# 2. 强制推送到远程（覆盖历史）
git push --force origin 分支名
```
⚠️ 警告：仅限个人分支使用！公共分支慎用，会覆盖他人历史记录

---

### 完整工作流示例：
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

### 替代方案（保留历史记录）：
```bash
# 用新 commit 撤销旧 commit（保留历史痕迹）
git revert HEAD
```
- 效果：创建新的反向 commit 抵消之前的修改
- 适用场景：公共分支/需要保留撤销记录的情况

### 重要提示：
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
