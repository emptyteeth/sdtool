# systemd 服务文件生成器

在浏览器里填写表单，实时生成 systemd unit 文件，附带语法检查和部署命令。

**在线使用：** https://emptyteeth.github.io/sdtool/

单文件、无构建、无后端。所有计算都在浏览器里完成，填写的内容只存在本机 `localStorage`，不会上传到任何地方。

## 功能

**生成**
- 按 `[Unit]` / `[Service]` / `[Install]` 三段分组，约 90 个常用指令
- 依赖与启动顺序：`After` / `Wants` / `Requires` / `Before` / `PartOf` / `Conflicts`，以及 `StartLimitIntervalSec` + `StartLimitBurst` 熔断阈值
- 进程：`Type`（simple / exec / forking / oneshot / notify / dbus / idle）、`ExecStart` 全系列（Pre / Post / Stop / StopPost / Reload）
- 重启与超时：`Restart` 策略、`RestartSec`、`TimeoutStopSec`、`WatchdogSec`、`RemainAfterExit`
- 加固：`NoNewPrivileges`、`ProtectSystem` / `ProtectHome`、`SystemCallFilter`、`CapabilityBoundingSet` 等，带一键「套用推荐加固」
- 资源限制：`MemoryMax`、`CPUQuota`、`LimitNOFILE`、`Nice`、`UMask` 等

**三个重点指令**
- `DynamicUser` —— 勾选后输出 `DynamicUser=yes`。与 `User=` 冲突时会提示，并把被忽略的 `User=` 从输出里去掉，避免踩坑
- `Environment` —— 多行 `KEY=VALUE` 输入，逐行生成 `Environment=`；含空格的值自动加引号转义；`#` 开头的行保留为注释；格式错误的行会报错
- `StateDirectory` —— 连同 `CacheDirectory`、`LogsDirectory`、`ConfigurationDirectory`、`RuntimeDirectory` 一起提供。只接受目录名，填绝对路径或含非法字符会报错，且不写进输出

**检查与辅助**
- 实时语法检查，错误 / 建议 / 说明三级，例如：
  - `ExecStart` 不是绝对路径
  - `Restart` 配了但没设 `RestartSec`（默认只有 100ms，崩溃循环会刷满日志）
  - `ProtectSystem=strict` 却没配 `ReadWritePaths`
  - `WatchdogSec` 配在非 `notify` 类型上
  - `MemoryDenyWriteExecute` 会杀掉需要 JIT 的运行时
- 「部署命令」标签页：写文件、`systemd-analyze verify`、`daemon-reload`、`enable --now`、`journalctl -f` 的完整命令序列
- 中英双语（右上角切换，首次访问按浏览器语言判断）
- 深色 / 浅色 / 跟随系统三态主题
- 桌面双栏，手机上输出区收成底部抽屉

## 本地使用

直接用浏览器打开 `index.html` 即可，不需要安装任何东西。

想跑本地服务器（可选）：

```sh
python3 -m http.server 8000
# 然后访问 http://localhost:8000
```

## 项目结构

```
index.html                       整个应用,HTML + CSS + JS 都在里面
.github/workflows/pages.yml      GitHub Pages 部署
```

代码里几处关键结构：

- `SCHEMA` —— 字段定义。每个分组对应 unit 文件的一段，字段的就近文案写成 `{zh, en}` 对
- `STR` —— 界面文字、校验消息、生成文件注释的文案表，按 key 取用，`{0}` 占位符插值
- `buildLines()` —— 由表单值生成 unit 文件的指令序列
- `validate()` —— 语法检查规则
- `render()` —— 每次输入后重跑以上全部并更新界面

## 部署

仓库已配置 GitHub Actions 发布 Pages。推送到 `main` 后工作流会自动把 `index.html` 作为站点根发布，也可以在 Actions 页面手动触发。

首次部署需要在仓库 **Settings → Pages → Build and deployment → Source** 中选择 **GitHub Actions**。

## 已知限制

- 语法检查是启发式的，覆盖常见坑，但不等于 `systemd-analyze verify`。真正部署前请按生成页面里给出的命令跑一次 `systemd-analyze verify`
- 未做「导入现有 unit 文件」的反向解析
- 指令集覆盖常用项，不是 systemd 全部指令
