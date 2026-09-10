# Charter 管理的 Oracle 安装

2026-09-11，Asia/Singapore。本 fork 保存产品代码与 Oracle Skill 正本；
上游为 https://github.com/steipete/oracle，人工择取更新，不自动跟随 main。

- 收编基线：`67253e7b1c2deae777761d3491bdd1b62c4c85d4`。
- CLI：npm `@steipete/oracle@0.20.0`，对应上游 `v0.20.0`
  （`261d3cb8f7da9063e1a7d97d1dbeda8e1269f4ca`）。
- 发布包 integrity：`sha512-BcrY4W6Wp1qwAZ1H1wD9vMLHhrzMKXYn82ArTC2ISBFSRZsK5WN+udXTXIeDpTkONfOu0SUsrVYthApoEJ9gWg==`。
- 本地适配：Skill 使用受管 `oracle`，移除已不适用的 0.15.2 临时安装回退；
  浏览器登录与模型替换遵循当前用户选择。CLI 运行发布包，不从此工作树临时编译。

## 所有权与接线

| 对象 | 权威与写者 |
|---|---|
| Skill 内容 | 本仓 `skills/oracle/`，通过 PR 维护 |
| 全域授权 | `~/ns/base/skills/manifest.toml`，`repos = "workspace"` 覆盖所有合法域工作区及有效在册 worktree |
| 客户端链接 | Charter `skills-sync` 生成 `.agents/skills/oracle` 和 `.claude/skills/oracle` |
| 启动脚本与版本选择 | 本仓 `scripts/oracle-managed`，通过 PR 维护 |
| 全机命令入口 | `~/.local/bin/oracle` 链接到本仓启动脚本，由安装步骤接线 |
| 已安装 CLI 与依赖锁 | `~/.local/share/oracle/releases/0.20.0/`，npm 写入 |
| 机器配置、会话与持久浏览器登录 | `~/.local/state/oracle/`，Oracle 原生管理，不入 Git |

默认用户配置使用 browser 引擎和 manualLogin，cookieSync 与 manualLoginCookieSync
均为 false；不预填账号、token 或 API key。首次网页调用在 Oracle 的持久浏览器中
登录。模型和思考等级按任务显式传入。已有运行超时后用 `oracle session <id>` 恢复。

## 安装、检查与升级

安装当前 CLI（Node >=24；`~/.local/bin` 已在各域 PATH 中）：

```sh
npm --cache "$HOME/.cache/npm" install --prefix "$HOME/.local/share/oracle/releases/0.20.0" --save-exact --omit=dev @steipete/oracle@0.20.0
ln -s "$HOME/work/personal/oracle/scripts/oracle-managed" "$HOME/.local/bin/oracle"
oracle --version
oracle --engine browser --browser-manual-login --dry-run summary -p "Installation check; do not submit."
```

重复安装时，已有命令链接仅在目标相同时复用；若目标不同则先核对归属，不强制覆盖。
从旧 personal 授权迁移时，先接好全机命令，再移除 `~/ns/personal/bin/oracle`
和 personal manifest 中的重复授权；保留机器配置与已登录 profile。各域复用本机同一
Oracle 登录状态，授权本身不复制账号或会话数据。

在配置和授权就位后，对每个目标仓用
`~/ns/.charter/scripts/skills-sync --repo <仓路径> --dry-run` 检查计划，
确认仅包含预期变更后去掉 `--dry-run` 应用，再次检查应无漂移。
`inspect --json` 可查询有效授权和来源。新仓与在册 worktree 继续由 Charter
的现有入口收敛。卸载同样逐仓对账；不使用仅筛选 manifest 的全局清理调用。

升级时先审核上游 release 与 Skill 差异，把候选 CLI 安装到新的版本目录，
验证版本、帮助和无提交预览后，通过 PR 同步启动脚本版本、本页 provenance 和必要的
Skill 改动；真实浏览器调用还需验证所选模型。依赖重装可在现有版本目录用
`npm ci --omit=dev` 复用 npm 保存的 package-lock.json。

卸载时先删除 manifest 授权并运行 skills-sync 回收链接，再移除 `~/.local/bin/oracle` 链接。
安装包目录可按版本清理；浏览器登录与会话数据保留，删除须由用户另行决定。
若将来由客户端原生安装机制接管本 Skill，先移除这份授权和全机命令链接，避免双写。
