# web-to-windows-desktop — 网页打包 Windows 桌面应用

把任意网页 URL 或本地 HTML 文件打包成原生 64 位 Windows 桌面应用，一次构建产出便携版 EXE 和安装程序两种格式。

## 功能

- **两种输入**：网页 URL 或本地 HTML 文件，均支持
- **双输出**：
  - **便携版 EXE** — 免安装，直接运行
  - **NSIS 安装程序** — 标准安装向导，支持桌面快捷方式、开始菜单和卸载
- **自动提取应用名**：从页面标题自动命名
- **自定义图标**：支持 PNG / ICO（推荐 256×256）
- **单实例锁**：默认只允许运行一个实例
- **安全默认**：默认开启 context isolation 与 sandbox

## 技术架构

Electron 33 + electron-builder 25，仅支持 64 位 Windows 目标。

## 触发示例

- “把 https://example.com 做成 Windows 桌面应用”
- “把这个网页转成 exe”
- “生成一个 Windows 安装程序”

## 可配置参数

| 参数 | 说明 |
|:---|:---|
| `url` / `htmlPath` | 网页地址或本地 HTML 路径（二选一必填） |
| `name` | 应用名称，用于窗口标题、快捷方式和安装目录 |
| `appId` | 反向域名格式的应用标识 |
| `icon` | 应用图标（推荐 256×256） |
| `width` / `height` | 默认窗口尺寸（1280×800） |
| `singleInstance` | 是否只允许单实例（默认 true） |

## 仓库内容

```
web-to-windows-desktop-skill/
├── SKILL.md                    # Skill 定义文档（英文，Agent 执行工作流）
├── web-to-windows-desktop.md   # 中文说明文档（架构、参数、常见问题）
└── READ_ME.txt                 # 包清单与使用指南
```
