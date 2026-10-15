# 浏览器安装、验证与备用路线

核对日期：2026-10-15。首次配置在接收者自己的电脑执行。本包只提供步骤，不预装软件、不附带登录态，也不声称下列路线已在接收者环境测试。

## 先给Codex这段话

> 阅读本项目`skills/xhs-content-agent/SKILL.md`及`skills/xhs-content-agent/references/browser-setup.md`。先检测系统、现有浏览器和CLI，再引导安装Ego Lite并完成只读连接验证。Ego因系统支持、连接或渲染能力无法工作时，使用本地Chrome + Browser Use CLI作为备用。保留Ego优先规则，不启用云浏览器或另购模型API，不读取或复制Cookie。安装与常规检查由你执行；首次引导、登录验证码和浏览器权限由我完成。验证小红书前台和创作者中心实际账号，先不上传或发布。

## A. 首选：Ego Lite

1. 从[Ego Lite官网](https://lite.ego.app/)下载并安装。当前官网提供Mac下载，Windows显示等待通知；其他系统先核官网支持情况，不假设已有安装包。
2. 打开应用完成首次引导。浏览器数据迁移是使用者自己的选择，不是本工作流的前提；也可以在Ego内手动登录所需网站。
3. 首次引导会配置`ego-browser`命令及技能。在Codex中确认发现并阅读当前安装的`ego-browser`技能；没有发现时先查官网与本地安装状态，不猜SDK接口。[官方入口](https://lite.ego.app/)

在macOS终端检查CLI（不用发布任何内容）：

```bash
command -v ego-browser
ego-browser nodejs <<'JS'
console.log('ego-browser ready');
JS
```

命令找不到时，完成应用引导、重开终端；已安装技能的安装文档若确认CLI位于`~/.local/bin`，可临时补PATH再检查：

```bash
export PATH="$HOME/.local/bin:$PATH"
command -v ego-browser
```

打印ready只证明CLI能执行。下一步由Codex读取已安装技能后运行一个真正的公开网页检查：

```bash
ego-browser nodejs <<'JS'
const task = await taskSpace('browser-connection-check');
console.log({ taskSpaceId: task.spaceId });
const page = task.page('p1');
await page.goto('https://example.com');
console.log(await page.snapshot());
await task.finish({ keep: [] });
JS
```

示例基于已核对的Ego技能API；新版本以本机技能为准。网页检查失败时保留打印的空间ID，按当前技能恢复或交还，不能不断创建新空间重试。

实际任务：同一目标复用一个TaskSpace，记录空间ID和页面标签；成功后按技能结束。不要把Ego API当Playwright。用户接管、登录、验证码或权限弹窗需交还，确认完成后再继续。只在用户指定profile时选择它，不假设Chrome与Ego共享会话。

## B. 备用：Chrome + Browser Use CLI

### 安装

这里使用开源项目Browser Use的CLI，由现有Codex执行浏览器命令；官方把它作为编码Agent入口。它与另行运行一个带模型调用循环的Python Agent是不同用法。[项目README](https://github.com/browser-use/browser-use)

先安装Chrome及uv。uv按[官方安装页](https://docs.astral.sh/uv/getting-started/installation/)选择系统方式，例如已有Homebrew的macOS可运行`brew install uv`，Windows可运行`winget install --id=astral-sh.uv -e`。

```bash
uv tool install --python 3.12 browser-use
browser-use --help
browser-use skill install
uv tool list
```

安装前先检查是否已有版本，保留版本记录，不自动强制覆盖。上述以稳定发布版为起点；如果当前官方页面接口与已装版本不匹配，先查看本地help及skill，再决定升级方案，不混用不同版本API。[CLI文档](https://docs.browser-use.com/open-source/browser-use-cli)

让Codex读取安装后的Browser Use技能。该包的项目规则仍然是Ego优先；不要照搬上游技能中“所有网页操作都改走此工具”的建议，也不删除其他已有技能。

### 连接Chrome并只读验证

打开Chrome，在`chrome://inspect/#remote-debugging`启用远程调试；连接允许弹窗由使用者处理。Chrome官方的自动连接方案要求144或更高版本；本包不额外要求安装Chrome DevTools MCP。[Chrome自动连接文档](https://developer.chrome.com/docs/devtools/agents/use-cases/auto-connect)

然后执行：

```bash
browser-use --doctor
browser-use <<'PYCODE'
print(page_info())
PYCODE
```

用新标签检查公开网页：

```bash
browser-use <<'PYCODE'
new_tab("https://example.com")
print(page_info())
PYCODE
```

命令依据[Browser Use CLI用法](https://docs.browser-use.com/open-source/browser-use-cli)。连接成功后，只操作本任务标签；不要修改或关闭用户其他标签。

Windows PowerShell把heredoc改为here-string：

```powershell
@'
print(page_info())
'@ | browser-use
```

### 自动连接不适用时：独立Chrome资料目录

先检查doctor结果。如果属于连接方式不支持，而非用户拒绝授权，可选择独立实例。macOS示例：

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.browser-use-content-chrome"
```

在另一终端连接：

```bash
BU_CDP_URL=http://127.0.0.1:9222 browser-use <<'PYCODE'
print(page_info())
PYCODE
```

这个目录属于接收者本机，新窗口需要本人登录；不复制日常浏览器Cookie。端口仅用于本机，不转发到公网；若已占用先查明，不结束不相关进程。Chrome136起远程调试端口模式需非默认资料目录。[Chrome限制说明](https://developer.chrome.com/blog/remote-debugging-port) 其他系统按[Chrome官方配置](https://developer.chrome.com/docs/devtools/agents/get-started/configuration)调整可执行路径。

### 费用与本地记录

本地Chrome无需Browser Use API key；云浏览器是独立可选路线，本包不默认开启。[官方安装说明](https://github.com/browser-use/browser-harness/blob/main/install.md#cloud-browsers) 现有Codex使用额度仍按使用者自己的方案计算，不承诺零总成本。

不要把Codex订阅当可直接传给独立Python Agent的API密钥。若使用者另选Python Agent及模型服务，再单独配置其凭据和预算；本包没有自动切换该路线。

首次安装不自动开启浏览器录制；已有设置保留，截图/动作轨迹若要开启由使用者决定。日志、资料目录、认证文件和截图不要加入分享包。

## C. 小红书首次使用

连接检查通过后，在选定浏览器里打开[小红书前台](https://www.xiaohongshu.com/)与[创作者中心](https://creator.xiaohongshu.com/)，由使用者完成确有需要的登录。页面处于初始化过渡态时等待真实结果，不见一次登录框就判断失效。

读取前台实际身份，确认是用户要分析的本人账号；创作者中心在安装验证或任务明确需要时再核实，后台数据分析仍以用户请求为范围。配置还没有账号标识时先展示核对结果，经用户确认后补入本地配置；不一致先解决账号选择，不发布。

首次定位/IP任务完成连接后按[引导读取](stages.md#引导读取先看已有材料再补问)采集本人主页、代表内容与指定参考页面，形成草案并补问缺失信息。登录成功仅表示访问就绪，不等于定位、图文或发布已确认。

## D. 什么情况下切换

| 状态 | 处理 |
| --- | --- |
| Ego未支持当前系统或安装/连接确实不可恢复 | 记录问题，按本指南用Chrome + Browser Use |
| Ego页面渲染/操作能力存在具体缺口 | 保存进度与已发生效果，在同一任务范围切换 |
| 登录、验证码、权限提示、用户接管或网站拒绝 | 请使用者处理或停止相关动作，不换路线绕过 |
| 已点击发布但结果未知 | 先只读核验；切换也只用于核验，不能重新提交 |
| Browser Use也不可用 | 继续可独立完成的研究与文案，记录浏览器阻碍，不伪造页面结果 |

验收依次记录：CLI可运行、网页可读取、目标账号核实、具体平台动作回读。前一项不自动证明后一项。本包没有执行备用浏览器安装、登录或发布测试。
