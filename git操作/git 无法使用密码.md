
remote: Invalid username or token.
Password authentication is not supported for Git operations.

## 方法一：使用 Personal Access Token（PAT）（推荐 HTTPS）

### 1. 创建 PAT（Personal Access Token）
登录 GitHub，在 Settings → Developer settings → Personal access tokens 中创建一个 Token。
如果是操作私有仓库，需要至少授予 repo 权限。
### 2. 使用 Token 代替密码
执行：
`git push`
Git 提示输入：
`Username:
输入你的 GitHub 用户名。
然后提示：
`Password:`
不要输入 GitHub 登录密码！
这里需要粘贴刚刚生成的 PAT Token。
