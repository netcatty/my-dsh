# ~/.dsh 配置仓库

只同步**配置文本**，不同步任何会话、缓存、凭据。

## 进仓库的

- `AGENTS.md`：全局规则
- `profiles/<name>/cordis.yml`、`cordis.patch.yml`：profile 配置
- `profiles/<name>/package.json`、`pnpm-lock.yaml`、`pnpm-workspace.yaml`：依赖声明
- `.githooks/pre-commit`：提交前检查

## 明确不进仓库的

`.credentials.yaml` · `*.env` · `dsh-plugins/**/settings.json`（含 MCP 的 env/headers/oauth） · `*.bak` · `backups/` · `sessions/` · `storages/` · `logs/` · `cache/` · `attachments/` · `node_modules/`

## 新机器恢复

```powershell
git clone <repo> ~\.dsh-config-tmp
Copy-Item ~\.dsh-config-tmp\* ~\.dsh\ -Recurse -Force
cd ~\.dsh; git init -q; git config core.hooksPath .githooks
```

凭据、MCP 服务器配置、会话数据需要自己另填。

## 提交前钩子

`core.hooksPath` 指向 `.githooks`，不随仓库同步（`.git/config` 不版本化），新机器要手动设一次：

```bash
git config core.hooksPath .githooks
```

钩子做三件事：拦疑似私密文件名、拦密钥形态、拦 `.md` 里的本机绝对路径。报错就改，别用 `--no-verify` 绕过。
