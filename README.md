# local-service-green-package

> 把本机常驻服务类程序（new-api.exe、各种网关/服务 exe）打包成「解压后双击即用」的绿色免安装包，发给朋友或拷到另一台电脑直接运行，并配套一份「丢给 AI 就能部署」的提示词。

[English README](README_EN.md)

## 简介

这是一个面向 WorkBuddy / Claude Code 的 AI 技能（Skill）仓库。它解决一个非常具体的问题：你在自己电脑上跑着一个好用的本地服务（比如 new-api 这类 AI 网关），想分享给朋友用，或者拷到另一台电脑上用，但对方没有开发环境、不会装依赖、也不会排查端口冲突。

本技能指导 AI 把这类「一个 exe + 一个配置/数据文件 + 需要后台常驻」的自建工具，打包成一个标准化的绿色交付 zip：接收方解压、双击启动器即可使用，无需安装任何东西。同时包里附带一份自包含的《AI部署提示词.md》，接收方把它丢给自己的 AI 助手，AI 就能按步骤带他完成部署。

适合人群：

- 想把本机 AI 网关/工具分享给朋友的个人开发者
- 需要把自建服务拷到多台 Windows 机器上免安装运行的人
- 用 AI 助手做打包、部署自动化的高级用户

## 触发词

打包给朋友、绿色版、免安装包、解压即用、把 XXX 打包发给别人、拷到另一台电脑用、一键部署提示词、给别人装一套。

## 功能特性

- **标准化交付包结构**：定义了「启动器.exe + 服务主程序.exe + 配置/数据文件 + 使用说明.txt + AI部署提示词.md」的统一目录布局，顶层套一层文件夹，避免解压后文件散落一地。
- **七步标准打包流程**：摸清需求 → 造干净数据文件 → 写启动器 → PyInstaller 打包 → 双场景终验 → 写两份文档 → 打 zip 并解压实测，每一步都有明确做法。
- **干净数据快照**：用 `sqlite3` 的 `backup()` 做数据库快照（自动处理 WAL），在副本上清空运行数据、保留配置，绝不动用户本机的库。
- **智能启动器**：tkinter GUI 小面板 + 命令行模式，核心职责是「自动选空闲端口 → 拉起后端 → 等就绪 → 打开浏览器」。
- **8 条血泪铁律**：涵盖 Windows 端口探测陷阱、tkinter 缺失陷阱、沙箱进程回收、进程识别、bcrypt 密码重置、文档编码、开机自启注册表方案等真实踩坑。
- **最终验收清单**：9 项逐项打勾的交付前检查，确保「只跑一遍 happy path」不算完。

## 工作原理 / 技术栈

本仓库是一份纯文档型 AI 技能（无代码），核心是 `SKILL.md` 中的方法论，涉及的技术点：

- **PyInstaller**：`--onefile --noconsole` 把 Python 启动器打成单文件 GUI exe。
- **tkinter**：启动器 GUI 面板（强调打包环境必须带 tkinter，否则「双击没反应」）。
- **sqlite3 backup API**：做含 WAL 的数据库安全快照。
- **netstat -ano -p TCP**：Windows 上唯一可靠的端口占用判断方式（Windows 允许重复 bind，不能用 bind 探测）。
- **CreateToolhelp32Snapshot + QueryFullProcessImageNameW**：按完整路径识别进程，只杀自己目录下的后端，绝不按进程名 taskkill 误杀。
- **bcrypt**：给接收方重置账号密码（哈希不可反推，只能重置）。
- **注册表 HKCU Run 项**：免管理员权限的开机自启方案。
- **UTF-8 BOM + CRLF**：保证使用说明.txt 在老版记事本里不乱码。

### 关键技术要点示例

Windows 端口探测铁律：Windows 允许重复绑定，即使 `0.0.0.0:3000` 已被监听，再 bind 同端口也会成功。正确做法是读 `netstat -ano -p TCP` 的 LISTENING 结果（2 秒短缓存），且启动成功判定必须三条同时成立：本目录的后端进程活着 + 该进程正在监听这个端口 + 接口有响应。

## 安装与使用

这是一个 AI 技能仓库，不是独立软件。安装方法：把本仓库目录复制到你的技能目录下，文件夹名保持 `local-service-green-package`：

- WorkBuddy / CodeBuddy：`~/.workbuddy/skills/local-service-green-package/`
- Claude Code：`~/.claude/skills/local-service-green-package/`

重启会话后，在对话中说出触发词（如「把这个服务打包成绿色版发给我朋友」），AI 即会按 SKILL.md 的标准流程执行打包。

## 项目结构

```
local-service-green-package/
├── SKILL.md      ← 技能正文：交付包结构、七步流程、8 条铁律、验收清单
├── README.md     ← 本文件（中文详细版）
├── README_EN.md  ← 英文详细版
└── .gitignore
```

## 注意事项

- 技能中的方法论主要面向 **Windows** 环境（注册表自启、netstat 解析、PyInstaller exe 打包均是 Windows 语义）。
- 打包 GUI 启动器时必须用带 tkinter 的 Python（Windows 系统 Python 一般都带；隔离环境的 managed Python 常缺 tkinter）。
- 沙箱里 Bash 命令结束会回收子进程，「启动 → 验证 → 停止」必须放在同一条命令内完成。
- 修改数据库只做副本，绝不动用户本机正在运行的服务与数据。
- 最终必须从打好的 zip 解压到全新目录实跑一遍，不能只在本机原目录验证。

## License

MIT

## 作者

sheen945
