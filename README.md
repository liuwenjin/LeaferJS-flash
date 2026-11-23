# LeaferJS Flash

**可独立运行的网页版 2D 动画生成器**

> 纯前端 + AIGC：输入一句需求，生成一段 12–20 秒、打开即播的 2D 动画短片。
> 仓库包含 AIGC 结构化提示词、前端结果渲染组件与本地网页入口；AI 推理由 transweb.cn 的 serverless 服务提供。

![Pure Frontend](https://img.shields.io/badge/architecture-pure%20frontend-6f5cff)
![Vue SFC](https://img.shields.io/badge/component-Vue%20SFC-42b883)
![Serverless](https://img.shields.io/badge/AI-transweb.cn%20serverless-2ea44f)
![LeaferJS](https://img.shields.io/badge/LeaferJS-2.3.4-6f5cff)
![License: MIT](https://img.shields.io/badge/license-MIT-2ea44f)

---

## 项目简介

LeaferJS Flash 是一个**可独立运行的网页版 2D 动画生成器**，也是一个纯前端项目。

仓库内没有自建后端、没有数据库、没有构建配置，也不需要在本地安装依赖。打开 [`appPage.html`](./appPage.html)，输入一句需求，前端会把用户输入与当前应用标识提交给 **transweb.cn 的 serverless 服务**；服务按 AIGC 结构化提示词调用大模型，流式返回 JSON，前端组件解析后把结果渲染成可播放的动画，并支持版本管理、编辑与导出。

一句话概括：

```text
本地网页（纯前端） ──► transweb.cn serverless ──► 大模型 ──► 结构化 JSON ──► 前端渲染
```

## 项目组成

| 组成 | 文件 | 职责 |
| --- | --- | --- |
| **AIGC 结构化提示词** | [`appPrompt.md`](./appPrompt.md) | 约束模型的角色、交付格式、转义规则、运行时契约与质量验收 |
| **前端 AI 结果渲染组件** | [`appCode.vue`](./appCode.vue) | Vue 单文件组件：输入向导、流式状态、结果渲染、版本历史、编辑与导出 |
| **本地浏览器网页** | [`appPage.html`](./appPage.html) | 本地入口页，打开即用；以 iframe 加载 transweb.cn 上的同款应用 |
| **元数据** | [`manifest.json`](./manifest.json) | 应用来源、创建时间与文件清单 |

## 架构与数据流

```text
┌────────────────────────── 浏览器（纯前端） ──────────────────────────┐
│  appPage.html（本地入口）                                            │
│        │ iframe                                                      │
│        ▼                                                             │
│  appCode.vue（Vue SFC）                                              │
│   ├─ 输入需求 / 示例 / 附件                                          │
│   ├─ 提交 context + documentId                                       │
│   ├─ 接收流式结果、解析 JSON                                         │
│   └─ 渲染动画 / 版本 / 编辑 / 导出                                   │
└───────────────┬──────────────────────────────────────────────────────┘
                │ HTTPS 流式请求
                ▼
   transweb.cn serverless（提示词编排 / 模型调用 / 结果存储 / 登录与额度）
                │
                ▼
             大模型 → 结构化 JSON
```

### 提示词与前端的关系

- `appPrompt.md` 是本项目的 **AIGC 结构化提示词**，也是模型输出必须遵守的契约。
- 在线应用按 `documentId` 绑定平台侧的提示词配置；前端提交的是用户输入（`context`）与应用标识，而不是每次都上传整份提示词。
- 修改仓库里的 `appPrompt.md` 不会自动改变线上应用；需要在平台更新提示词配置，或把提示词接入你自建的 serverless 服务。

## 为什么是「纯前端 + serverless」

| 维度 | 说明 |
| --- | --- |
| **无自建后端** | 仓库只有 HTML、Vue SFC、Markdown 与 JSON，没有服务端代码 |
| **无需构建** | 没有 `package.json`、打包器或编译步骤；打开网页即可使用 |
| **前端负责交互与渲染** | 输入、流式状态、JSON 解析、动画渲染、版本与导出都在浏览器完成 |
| **AI 在 serverless** | 模型调用、流式返回、结果存储由 transweb.cn 承担，前端不持有模型密钥 |
| **静态可托管** | 入口页可放在任意静态服务器、对象存储或本地目录 |

**独立运行的含义**：不依赖本地开发环境、不需要部署自己的后端、不需要安装 npm 包。它不是「离线运行」，生成能力依赖 transweb.cn 的 serverless 服务与网络。

## 运行方式

### 方式一：直接打开（最简单）

双击 [`appPage.html`](./appPage.html)，或在浏览器里打开它。页面会以 iframe 加载 transweb.cn 上的同款应用。

### 方式二：本地静态服务（推荐）

某些浏览器对 `file://` 下的 iframe、剪贴板或跨域行为有限制，建议起一个静态服务：

```bash
# 在仓库根目录
python3 -m http.server 8080
# 打开 http://localhost:8080/appPage.html
```

或者使用任意静态服务器（Nginx、Caddy、VS Code Live Server 等）。

### 运行前提

- 能访问 `transweb.cn`（AI 生成与结果存储依赖该服务）
- 现代浏览器（Chrome、Edge、Safari、Firefox）
- 无需 Node.js、npm、构建工具
- 登录状态与调用额度以 transweb.cn 当前策略为准

## serverless 依赖说明

| 能力 | 是否在仓库内 | 说明 |
| --- | --- | --- |
| 模型调用 | 否 | 由 transweb.cn serverless 调用大模型 |
| 提示词编排 | 否 | 平台按 `documentId` 绑定 AIGC 提示词 |
| 流式返回 | 否 | 前端通过 `aigcInterface` 接收流式结果 |
| 结果存储与版本 | 否 | 历史版本由 transweb.cn 保存 |
| 输入、解析、渲染、导出 | 是 | `appCode.vue` 在浏览器内完成 |

前端不保存模型 API Key，也不直接连接模型服务；所有生成请求都经过 transweb.cn。服务不可用、未登录或额度不足时，页面会给出对应提示。

## 目录结构

```text
LeaferJS-flash/
├── appPrompt.md     # AIGC 结构化提示词（模型契约）
├── appCode.vue      # 前端 AI 结果渲染组件（Vue SFC，基于 transweb 前端运行时）
├── appPage.html     # 本地浏览器入口（iframe 加载在线应用）
├── manifest.json    # 应用元数据与来源
├── LICENSE          # MIT 许可证
└── README.md        # 本文件
```

## 它生成什么

一部完整的 2D 动画短片，而不是几张会动的图：

- **叙事完整**：12–20 秒内完成「建立 → 发展 → 转折 → 高潮 → 收束」
- **电影化镜头**：3–5 个镜头，各有明确任务；支持推进、拉回、跟随与分层视差
- **角色表演**：关键姿态驱动，带预备、跟随、回弹和次级动作延迟
- **可交互播放**：自动播放、暂停、继续、重播、进度条拖动、键盘控制
- **自适应画幅**：16:9 电影式信箱画幅等比居中，竖屏不裁切主体
- **程序化美术**：不使用图片、字体、音频文件，画面全部由 LeaferJS 图形构成
- **错误兜底**：CDN 加载失败或初始化异常时显示错误面板，不白屏

## AIGC 结构化提示词

[`appPrompt.md`](./appPrompt.md) 把模型设定为「电影级 2D 动画短片团队」，并强制约束输出。核心协议如下。

### 交付格式（硬约束）

模型必须**只输出一个合法 JSON 对象**，前后不允许任何解释文字、Markdown 代码围栏或注释。顶层固定 5 个字段，顺序固定：

| 字段 | 类型 | 约束 |
| --- | --- | --- |
| `type` | `string` | 固定为 `animation` |
| `title` | `string` | 中文名称，≤ 16 字，不含句号、引号、书名号 |
| `description` | `string` | 一句话摘要，≤ 30 字，不换行 |
| `code` | `string` | 完整单文件 HTML，源码 400–850 行，以 `<!DOCTYPE html>` 开头、`</html>` 结尾 |
| `highlights` | `string[3]` | 恰好 3 条，每条 ≤ 20 字，不写序号 |

结构示意（`code` 已省略）：

```json
{
  "type": "animation",
  "title": "纸鹤 · 夜航",
  "description": "一只纸鹤穿过暮色，飞向月亮。",
  "code": "<!DOCTYPE html> ... </html>",
  "highlights": [
    "分层视差与镜头推进拉回",
    "纸鹤分部件振翅与尾羽延迟",
    "纯时间轴驱动 + 暂停重播拖动"
  ]
}
```

### `code` 字段的转义铁律

`code` 的值本身是一段 JSON 字符串。为了不让裸双引号提前结束字符串，提示词要求：

1. `code` 源码内**零双引号**：HTML 属性、JS 字符串、CSS 字体名全部用单引号，插值用反引号模板字符串。
2. `code` 源码内**零反斜杠**：不写 `\n`、`\t`、`\u`、正则转义或 `<\/script>`；换行直接写真实换行。
3. JSON 序列化时，真实换行写成 `\n`，这是 `code` 值里唯一允许的反斜杠形式。
4. 禁止尾随逗号、注释和代码围栏；缩进只用空格。

## 前端渲染组件

[`appCode.vue`](./appCode.vue) 是平台运行时下的 Vue 单文件组件，负责从输入到成品的完整前端链路：

- **输入**：需求文本、示例模板、附件读取，`solo` 模式与嵌入模式自适应
- **生成**：调用 `aigcInterface.streamRequest` 发起流式请求，展示加载进度与阶段提示，支持中止
- **解析**：用 `WebTool.extractJSON` 提取 JSON，格式异常时提示并允许重试
- **渲染**：把 `code` 交给结果面板渲染为 LeaferJS 动画，提供播放与错误面板
- **版本**：历史版本列表、版本对比、切换与恢复
- **编辑**：基于当前内容继续修改，生成新版本
- **导出**：存档 HTML、下载 Markdown、导出图片、复制分享链接
- **反馈**：内置问题反馈入口

> 组件依赖 transweb 前端运行时与 serverless 接口，不是可以直接 `import` 到任意 Vue 工程的 npm 包；它的价值在于展示 AIGC 结果如何被解析、渲染和管理。

## 技术栈

| 层 | 选型 |
| --- | --- |
| 本地入口 | 原生 HTML + iframe |
| 前端组件 | Vue 单文件组件（Options API）、Element 组件库、Tailwind 原子类 |
| AIGC 提示词 | Markdown 结构化提示词（`appPrompt.md`） |
| AI 服务 | transweb.cn serverless，流式返回 |
| 结果渲染 | LeaferJS 2.3.4（生成物内唯一 CDN 依赖） |
| 动画驱动 | `requestAnimationFrame` + 手动时间轴 |
| 结果存储 | transweb.cn 版本与历史记录 |

生成的动画产物只依赖一个 LeaferJS 脚本：

```text
https://unpkg.com/leafer-ui@2.3.4/dist/web.min.js
```

## 独立运行的边界

| 可以 | 不可以 |
| --- | --- |
| 本地双击或静态托管 `appPage.html` | 完全离线生成（需要 transweb.cn） |
| 不安装 Node.js、不构建、不部署后端 | 自带模型或本地模型直接推理 |
| 浏览器内完成输入、渲染、版本、导出 | 不依赖账号/额度无限调用 |
| 把生成的 HTML 单独保存、分享、嵌入 | 直接修改 `appPrompt.md` 就让线上生效 |

如果要把 AI 能力迁到自己的基础设施，需要实现与 `aigcInterface` 兼容的流式接口，或把提示词接入自己的模型网关；这部分不在本仓库内。

## 交付验收清单

- [ ] 打开 `appPage.html` 能看到应用界面，而不是白屏
- [ ] 输入需求后能看到流式生成状态
- [ ] 生成结果能正常播放，暂停/重播/进度条有效
- [ ] 12–20 秒内走完「建立 → 发展 → 转折 → 高潮 → 收束」
- [ ] 手机竖屏完整保留 16:9 画幅，不裁切主体
- [ ] 历史版本可切换，编辑后可生成新版本
- [ ] 存档 HTML、下载 Markdown、导出图片可用
- [ ] 网络异常或服务不可用时有明确提示
- [ ] 生成物无控制台异常、无 404 资源

## 常见问题

**打开 `appPage.html` 一直白屏或转圈？**
先确认能访问 `https://transweb.cn`，再打开浏览器控制台看 iframe 是否被拦截。必要时改用本地静态服务（见「运行方式」），或直接访问在线应用地址。

**需要安装 Node.js 或 npm 吗？**
不需要。项目没有构建步骤；本地静态服务只是可选的更稳妥的打开方式。

**需要登录吗？**
以 transweb.cn 当前策略为准。部分功能（历史记录、额度、导出）可能要求登录。

**可以离线使用吗？**
不能离线生成。生成的动画 HTML 可以保存到本地播放，但它仍会加载 LeaferJS CDN；需要完全离线时，可把 `web.min.js` 下载到本地并替换脚本地址。

**能换成自己的模型吗？**
可以，但需要替换 `aigcInterface` 适配层或自建兼容的 serverless 服务，并自行实现流式协议。仓库不包含这部分实现。

**为什么改了 `appPrompt.md` 线上没变化？**
线上应用读取的是平台侧绑定的提示词配置；仓库里的文件是提示词契约与版本记录。要生效，需要在平台更新配置，或接入你自建的生成服务。

## 相关链接

- 在线应用：<https://transweb.cn/agent.html?id=1bb714f792cb885ffbfd3611e8b69dc3>
- 本地入口：[`appPage.html`](./appPage.html)
- LeaferJS 官网：<https://www.leaferjs.com/>
- LeaferJS 文档：<https://www.leaferjs.com/ui/guide/>
- 本仓库：<https://github.com/liuwenjin/LeaferJS-flash>

## 许可

本项目采用 [MIT License](./LICENSE)，与所引用的前端资源保持一致。

| 资源 | 许可证 | 用途 |
| --- | --- | --- |
| [LeaferJS](https://github.com/leaferjs/leafer-ui) | MIT | 生成动画的 2D 渲染引擎 |
| Vue / Element 组件库 / [Tailwind CSS](https://github.com/tailwindlabs/tailwindcss) | MIT | 前端运行时与界面样式 |

`transweb.cn` 的 serverless 服务不在本仓库内，按其服务条款使用；本仓库的 MIT 许可仅覆盖仓库中的提示词、组件与页面源码。

Copyright (c) 2026 liuwenjin。

再分发时请保留所引用资源的版权与许可声明；生成内容的许可由使用者自行决定。
