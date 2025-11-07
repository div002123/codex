# git_info.rs

## 文件作用

`git_info.rs` 提供 Git 仓库信息收集功能，包括检测仓库、获取分支和提交信息、计算与远程的差异等。所有操作都有超时保护，避免在大型仓库中阻塞。

## 主要结构体

### GitDiffToRemote
到远程分支的差异信息。

**字段:**
- `sha: GitSha` - 最近的共同祖先提交SHA
- `diff: String` - 差异内容

### CommitLogEntry
提交日志条目（用于选择器）。

**字段:**
- `sha: String` - 提交SHA
- `timestamp: i64` - Unix时间戳（秒）
- `subject: String` - 提交主题（单行）

## 主要函数

### get_git_repo_root()
获取 Git 仓库根目录（轻量级检查）。

**参数:**
- `base_dir: &Path` - 起始目录

**返回:** `Option<PathBuf>` - 仓库根目录

**功能:**
- 向上遍历目录树查找 `.git`
- 不依赖 git 命令或 git2 库
- 不检测 worktree

### collect_git_info()
收集 Git 仓库信息。

**参数:**
- `cwd: &Path` - 工作目录

**返回:** `Option<GitInfo>` - Git 信息

**功能:**
1. 检查是否在 Git 仓库中
2. 并发执行以下命令：
   - `git rev-parse HEAD` - 获取提交哈希
   - `git rev-parse --abbrev-ref HEAD` - 获取分支名
   - `git remote get-url origin` - 获取远程URL
3. 聚合结果
4. 如果分支名为 "HEAD"（detached），不返回分支名

### recent_commits()
获取最近的提交列表。

**参数:**
- `cwd: &Path` - 工作目录
- `limit: usize` - 最大提交数

**返回:** `Vec<CommitLogEntry>` - 提交列表

**功能:**
使用 `git log` 获取最近的提交，包含SHA、时间戳、主题。

### git_diff_to_remote()
获取相对于远程分支的差异。

**参数:**
- `cwd: &Path` - 工作目录

**返回:** `Option<GitDiffToRemote>` - 差异信息

**功能:**
1. 获取所有远程分支
2. 获取当前分支的祖先链
3. 找到最近的共同祖先（在远程上存在）
4. 计算与该SHA的差异

### run_git_command_with_timeout() (私有)
带超时的 Git 命令执行。

**超时:** 5秒

**功能:**
防止大型仓库上的 Git 操作阻塞。

### get_git_remotes() (私有)
获取远程列表。

### branch_ancestry() (私有)
获取分支祖先链。

### find_closest_sha() (私有)
找到最近的在远程上存在的SHA。

### diff_against_sha() (私有)
计算与指定SHA的差异。

## 常量

- `GIT_COMMAND_TIMEOUT` - Git 命令超时（5秒）

## 与其他模块的关系

**依赖模块:**
- `codex_protocol::protocol::GitInfo` - Git 信息类型

**被依赖模块:**
- `codex::Session` - 收集仓库信息
- UI/CLI - 显示仓库状态
- 审查功能 - 获取差异信息

**核心流程:**
会话初始化 → 检测 Git 仓库 → 收集信息 → 发送给客户端 → UI 显示

## 设计说明

### 并发执行
多个 Git 命令并发执行，减少总延迟。

### 超时保护
所有 Git 命令都有5秒超时，避免在大型仓库或网络问题时阻塞。

### 容错性
任何 Git 操作失败都返回 None，不中断会话。

### 轻量级检查
`get_git_repo_root()` 只检查文件系统，不调用 git 命令，非常快速。
