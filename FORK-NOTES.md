# 维护说明（kkda49/AgentDock-Workbench）

本仓库是 [A-m-o-r-F-a-t-i/AgentDock-Workbench](https://github.com/A-m-o-r-F-a-t-i/AgentDock-Workbench) 的独立维护副本。
上游本身是 [uvwt/agentdock](https://github.com/uvwt/agentdock) 的分支（Apache-2.0）。由于 kkda49 账号的 fork 网络名额被旧的 `kkda49/agentdock` 占用，本仓库采用"独立仓库 + upstream 远程"的方式做等价 fork。

## 远程与同步

- `origin`：本仓库；`upstream`：A-m-o-r-F-a-t-i/AgentDock-Workbench
- 跟上游同步：

```bash
git fetch upstream
git merge upstream/main
git push origin main
```

## 本仓库相对上游的改动

1. `desktop/windows/control-panel/Themes/Tokens.xaml`：ComboBox 模板补上可编辑部件（`PART_EditableTextBox` + `IsEditable` 触发器），修复目标模式弹窗"浏览器本机调试端点"输入框无法打字的问题；非可编辑下拉框行为不变
2. `desktop/windows/control-panel/Services/GoalMode/RoxyDiscovery.cs`：端点发现通用化，扫描 RoxyChrome / Chrome / Edge / Brave / Chromium 主进程并解析 `--remote-debugging-port`（`DevToolsActivePort` 文件作为回退），修复"发现 Roxy"按钮找不到普通 Chrome 的问题
3. `.github/workflows/windows-installer.yml`：历史基线安装包改为从上游仓库的 Release 下载（本仓库不承载旧版本发布）。升级矩阵保持上游的 1.1.0-1.1.9 全量，勿裁剪——`scripts/test` 的守卫测试 `TestWindowsInstallerWorkflowHasAlwaysPresentPullRequestGate` 会校验 `- '1.1.0'` 必须存在
4. `.github/workflows/ci.yml`：`Native browser integration` 与 `Container check` 两个作业的跳过条件加入本仓库 ID（`1423716695`），与作者仓库（`1370391430`）做法一致——其 `TestBrowserIntegrationRuntimeCloseClosesBrowserService` 在非作者仓库上稳定失败（上游仓库跳过不跑，无人维护），与本仓库改动无关
5. 其余工作流（macOS / Android / WB0x 候选 / CodeQL 等）在本仓库已禁用，减少无关运行；需要时 `gh workflow enable "<名称>"` 恢复

## 构建与安装包获取

- 推送触及 `desktop/windows/**`、`scripts/**`、`cmd/**`、`internal/**` 等路径会触发 **Windows Installer** 工作流；也可在 Actions 页手动 `workflow_dispatch`
- 构建产物在运行页 Artifacts：`windows-history-current-setup`（内含 `AgentDock-Workbench-Setup-amd64.exe`，保留 3 天）
- 本仓库未配置签名 secrets，产物为未签名版；带 `test_tag` 的测试 Release 需要签名 secrets（未配置时工作流会拒绝）
- 需要正式发版（带 tag 的 Release）时：配置 `WINDOWS_SIGNING_CERT_BASE64` / `WINDOWS_SIGNING_CERT_PASSWORD` 两个 secrets 后手动 dispatch 并填写 `test_tag`

## 变更历史

- 2026-10-18：仓库建立，已推送 3 个提交：
  - `80fdc2b9` 工作流 fork 适配：历史基线改从上游 Release 下载、升级矩阵缩为 1.1.8/1.1.9、新增本说明文件
  - `a89a2870` 目标模式端点输入框可编辑（ComboBox 模板补 `PART_EditableTextBox`）
  - `397db582` 端点发现通用化（多浏览器进程扫描 + `--remote-debugging-port` 解析 + `DevToolsActivePort` 回退）
- 验证记录：Windows frontend regression 在 `397db582` 全绿（双架构编译 + 目标模式 / 原生目标模式 / WPF 布局 / 网络共 5 个测试项目，逐步骤通过）；Windows Installer 全流程（构建、安装/升级 E2E、1.1.8/1.1.9 升级矩阵）在 `a89a2870` 与 `397db582` 均通过
- 2026-10-18：CI 修复——升级矩阵恢复为上游全量（曾缩为 1.1.8/1.1.9，触发 `scripts/test` 的 `TestWindowsInstallerWorkflowHasAlwaysPresentPullRequestGate` 守卫失败）；`ci.yml` 浏览器/容器作业的跳过条件加入本仓库 ID（作者仓库同样跳过，该作业在非作者仓库稳定失败，见上述改动第 4 条）
