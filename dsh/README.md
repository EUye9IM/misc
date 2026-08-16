# dsh/ — DSH Web 启动脚本 + 配置备份

这个目录收纳两样东西：

- **`dshweb`** — DeepSeek Harness Web 的后台启动脚本（按需拉起、探活、停服、打开浏览器）。
- **`profile/` + `settings.yaml`** — 当前 `web` profile 的插件清单与设定归档，用于跨机器迁移/恢复。

## 目录结构

```
dsh/
├── dshweb                  # Bash 启动脚本（见下方用法）
├── settings.yaml           # 用户级设定（默认模型、引导状态等）
└── profile/                # web profile 的插件配置归档（恢复时放到 ~/.dsh/profiles/web/）
    ├── package.json        # 插件依赖 + dsh.profile.bundles（按什么顺序加载）
    ├── cordis.patch.yml    # 你的插件设定/启停/覆盖补丁层
    └── pnpm-lock.yaml      # 锁文件（保证按精确版本重装，跨机器 100% 一致）
```

### 各文件对应 DSH 的位置

| 本目录文件 | DSH 实际位置 | 作用 |
|-----------|-------------|------|
| `dshweb` |（项目内运行即可）| 拉起 DSH Web 后台服务 |
| `settings.yaml` | `~/.dsh/settings.yaml` | 默认模型、UI 引导等用户级设定 |
| `profile/package.json` | `~/.dsh/profiles/web/package.json` | 插件依赖 + bundle 加载顺序 |
| `profile/cordis.patch.yml` | `~/.dsh/profiles/web/cordis.patch.yml` | 你的插件设定补丁层 |
| `profile/pnpm-lock.yaml` | `~/.dsh/profiles/web/pnpm-lock.yaml` | 精确版本锁定 |

## dshweb 用法

```bash
./dshweb                 # 确保后台服务在跑（不在则拉起），打印 URL，默认打开浏览器
./dshweb --no-open       # 只启动/确认，不打开浏览器
./dshweb --port 8080     # 换端口（默认 3080，也可用环境变量 DSH_WEB_PORT）
./dshweb --stop          # 停止后台服务
./dshweb -h              # 帮助
```

设计要点：重复运行自动复用已启动实例；探活以页面是否注入 `__DSH_BOOT__` 为准；对 `127.0.0.1` 的请求自动绕过代理（避免本地请求被代理拦截返回 503）。Linux 原生 / WSL 环境都支持。

## 如何用这些文件恢复 / 迁移

在**另一台机器**上还原当前 DSH Web 环境（不装任何额外插件）：

1. 新机器装好 DSH、`node` 和 `pnpm`，先启动一次让 `~/.dsh` 与 `~/.dsh/profiles/web` 骨架初始化。
2. 把配置归档放到位：
   ```bash
   cp -r dsh/profile/* ~/.dsh/profiles/web/
   cp dsh/settings.yaml ~/.dsh/settings.yaml
   ```
3. 安装插件依赖（按 lock 精确复刻版本）：
   ```bash
   cd ~/.dsh/profiles/web
   pnpm install --frozen-lockfile   # 用 lockfile 锁定版本
   ```
4. 重启 Harness，确认生效：
   ```bash
   dsh --profile web --dump-config   # 打印组合后的配置树，核对 bundle/patch
   ```

> 说明：恢复的是"插件清单 + 你的补丁设定 + 用户设定"。会话历史/缓存（`~/.dsh/sessions`、`~/.dsh/storages`）不在本归档内，如需一并迁移请用 `dsh-backup` 之类的整体备份方案。`profile/cordis.yml` 无需归档——它由 bundle + patch 启动时自动生成。
