# LeaferJS Flash

**可独立运行的 2D 动画短片生成器**

> 一句自然语言描述，得到一段 **12–20 秒、打开即播、可暂停可重播** 的 2D 动画短片。
> 自带任意大模型即可运行；产物是零依赖的单文件 HTML。

![BYOM](https://img.shields.io/badge/model-any%20LLM-6f5cff)
![LeaferJS](https://img.shields.io/badge/LeaferJS-2.3.4-6f5cff)
![Single File](https://img.shields.io/badge/output-single--file%20HTML-2ea44f)
![No Build](https://img.shields.io/badge/build-none-lightgrey)

---

## 简介

**LeaferJS Flash 是一个可独立运行的 AI 工具**，不是某个平台的插件，也不需要登录任何账号。

它的全部能力来自一份系统提示词 [`appPrompt.md`](./appPrompt.md)：把这份提示词交给任意支持长输出的 LLM，模型就会把你的描述变成一部电影感 2D 动画，并以严格 JSON 交付。取 JSON 里的 `code` 字段保存为 `.html`，双击就能播放。

```text
你的描述 ──► 任意 LLM ──► JSON ──► animation.html ──► 浏览器
                ▲
          appPrompt.md
```

工具链上没有隐藏服务：没有后端、没有数据库、没有构建步骤。你需要的只是一个模型端点（云端 API 或本地模型）和一个浏览器。

## 为什么它可以独立运行

| 维度 | 说明 |
| --- | --- |
| **模型自由** | OpenAI、DeepSeek、Claude、Gemini、Ollama、vLLM 均可；只要支持 system prompt 与长输出 |
| **无平台锁定** | 不调用 transweb.cn 或任何私有接口；平台页面只是同款提示词的参考实现 |
| **本地执行** | 提示词、调用脚本、产物都在你的机器上；使用本地模型时可完全离线生成 |
| **产物可移植** | 输出是单个 `.html` 文件，可双击打开、邮件发送、静态托管或嵌入 iframe |
| **过程可审计** | 提示词、协议和生成结果全部可见，可 diff、可版本化、可二次修改 |

## 它生成什么

一部完整的短片，而不是几张会动的图：

- **叙事完整**：12–20 秒内完成「建立 → 发展 → 转折 → 高潮 → 收束」
- **电影化镜头**：3–5 个镜头，各有明确任务；支持推进、拉回、跟随与分层视差
- **角色表演**：关键姿态驱动，带预备、跟随、回弹和次级动作延迟
- **可交互播放**：自动播放、暂停、继续、重播、进度条拖动、键盘控制
- **自适应画幅**：16:9 电影式信箱画幅等比居中，竖屏不裁切主体
- **程序化美术**：不使用图片、字体、音频文件，画面全部由 LeaferJS 图形构成
- **错误兜底**：CDN 加载失败或初始化异常时显示错误面板，不白屏

## 快速开始（独立运行）

### 前置条件

- Node.js 18+（内置 `fetch`，无需 `npm install`）
- 一个 OpenAI 兼容的模型端点；建议单次输出上限 **≥ 8K tokens**

### 1. 获取工具

```bash
git clone https://github.com/liuwenjin/LeaferJS-flash.git
cd LeaferJS-flash
```

### 2. 创建本地运行器

把下面的代码保存为 `run.mjs`（放在仓库根目录）：

> 不想用 Node.js？任何 HTTP 客户端都可以：把 `appPrompt.md` 作为 system message、需求作为 user message 发送，再解析返回的 JSON。

```js
// run.mjs — Node.js 18+，零依赖
import fs from 'node:fs';

const system = fs.readFileSync('appPrompt.md', 'utf8');
const idea = process.argv.slice(2).join(' ') || '一只纸鹤穿过暮色，飞向月亮。';
const base = (process.env.LLM_BASE_URL || 'https://api.deepseek.com').replace(/\/$/, '');

const res = await fetch(`${base}/chat/completions`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${process.env.LLM_API_KEY}`
  },
  body: JSON.stringify({
    model: process.env.LLM_MODEL || 'deepseek-chat',
    messages: [
      { role: 'system', content: system },
      { role: 'user', content: idea }
    ],
    max_tokens: Number(process.env.LLM_MAX_TOKENS || 8000),
    temperature: 0.7
  })
});

if (!res.ok) throw new Error(`HTTP ${res.status}: ${await res.text()}`);
const body = await res.json();
if (body.choices?.[0]?.finish_reason === 'length') {
  throw new Error('输出被截断：请提高 LLM_MAX_TOKENS，或换用输出上限更大的模型');
}

const raw = body.choices[0].message.content.trim()
  .replace(/^`{3}(?:json)?\s*/, '')
  .replace(/\s*`{3}$/, '');
const data = JSON.parse(raw);
fs.writeFileSync('animation.html', data.code, 'utf8');
console.log(`✓ ${data.title} → animation.html（${data.code.split('\n').length} 行）`);
```

### 3. 生成并播放

```bash
LLM_API_KEY=sk-... node run.mjs "一只发光的鲸鱼在深海中缓缓游过，身后拖着星尘，最后消失在光里"
open animation.html     # macOS；其他系统直接双击
```

页面加载 LeaferJS 后自动播放。动画产物本身不需要 Node.js，也不需要这个运行器。

### 4. 换一个模型

| 服务 | `LLM_BASE_URL` | `LLM_MODEL` 示例 |
| --- | --- | --- |
| OpenAI | `https://api.openai.com/v1` | `gpt-4.1` |
| DeepSeek | `https://api.deepseek.com` | `deepseek-chat` |
| Ollama（本地） | `http://localhost:11434/v1` | `qwen2.5:14b` |
| vLLM（本地） | `http://localhost:8000/v1` | 你部署的模型名 |
| 其他网关 | 任意 OpenAI 兼容地址 | 以服务商文档为准 |

```bash
LLM_BASE_URL=https://api.openai.com/v1 LLM_MODEL=gpt-4.1 \
LLM_API_KEY=sk-... node run.mjs "雨夜里，一只猫跳上屋顶，追逐一盏飘走的天灯"
```

> Claude、Gemini 的原生 API 协议不同，可通过 OpenAI 兼容网关接入，或自行替换运行器中的请求部分。模型不是工具的一部分，可随时替换。

## 交付协议（硬约束）

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

## 技术栈

| 层 | 选型 |
| --- | --- |
| 生成引擎 | `appPrompt.md`（系统提示词）+ 任意 LLM |
| 运行器 | 约 40 行 Node.js 脚本，仅用内置 `fetch` |
| 渲染引擎 | LeaferJS 2.3.4（唯一 CDN 依赖） |
| 动画驱动 | `requestAnimationFrame` + 手动时间轴 |
| 画布 | `new L.Leafer({ view: window, type: 'draw', ... })`，关闭默认缩放/平移 |
| 自适应 | 设计分辨率 960×540，`stage` 等比缩放居中，每帧检测窗口尺寸 |
| 产物 | 单文件 HTML，无构建、无框架、无 `import` / `require` |

LeaferJS 只允许引入一次：

```text
https://unpkg.com/leafer-ui@2.3.4/dist/web.min.js
```

## 目录结构

```text
LeaferJS-flash/
├── appPrompt.md     # 工具引擎：完整系统提示词（必需）
├── README.md        # 本文件
├── appCode.vue      # 可选参考实现：平台端 UI（独立运行不需要）
├── appPage.html     # 可选参考实现：iframe 嵌入页
└── manifest.json    # 元数据
```

`run.mjs` 不在仓库内，按「快速开始」创建即可；`appCode.vue` 与 `appPage.html` 只是同款提示词在 transweb.cn 上的 Web 外壳，**不参与**独立运行链路，删掉它们不影响工具使用。

## 提示词设计要点

- **格式优先于内容**：JSON 不合法，动画再好也是 0 分。
- **可运行优先于炫技**：能完整播放一遍，胜过画面复杂但跑不起来。
- **一个清晰的视觉叙事**胜过十个互不相干的特效：一个场景、一个主体、一个核心想法。
- 输出前按 12 条清单机械自检，并在脑内播放一遍完整流程。
- 篇幅预算 400–850 行；预算不足时按 `P2 → P1 → 不降 P0` 降级，优先补完结尾与重播闭环。

## 交付验收清单

- [ ] 打开页面 1 秒内自动播放
- [ ] 12–20 秒内走完「建立 → 发展 → 转折 → 高潮 → 收束」
- [ ] 暂停真的冻结画面，继续从原时间点播放
- [ ] 重播回到 0 秒，无需刷新页面
- [ ] 进度条点击/拖动后画面与时间一致
- [ ] 结束有收束画面，播放按钮变为「重播」
- [ ] 切到后台再回来，时间不跳变
- [ ] 手机竖屏完整保留 16:9 画幅，不裁切主体
- [ ] 窗口 resize 后不拉伸、不变形
- [ ] 无控制台异常、无 404 资源
- [ ] 主体在远景、近景、高潮镜头中都清晰可读
- [ ] 至少有一个值得截图的记忆画面

## 常见问题

**必须联网吗？**
生成阶段需要访问模型；播放阶段需要加载 LeaferJS CDN。要完全离线，可下载 `web.min.js` 到本地并替换产物里的脚本地址，再配合本地模型使用。

**输出被截断怎么办？**
优先提高 `LLM_MAX_TOKENS`，或换用输出上限更大的模型。提示词已经要求模型在预算内优先保证 `</html>` 完整闭合。

**会保存我的数据吗？**
仓库没有后端。请求由你的运行器直接发往你选择的模型服务商；是否留存数据取决于该服务商的策略。

**可以商用吗？**
本仓库暂未声明许可证。提示词与生成代码的版权归各自作者所有，商用前请自行确认授权范围。

## 参考实现（可选）

- 在线效果参考：<https://transweb.cn/agent.html?id=1bb714f792cb885ffbfd3611e8b69dc3>
- `appPage.html` 通过 iframe 嵌入同一在线版本；它只用于预览，不是运行本工具的必要条件。

## 相关链接

- LeaferJS 官网：<https://www.leaferjs.com/>
- LeaferJS 文档：<https://www.leaferjs.com/ui/guide/>
- 本仓库：<https://github.com/liuwenjin/LeaferJS-flash>

## 许可

本仓库暂未指定开源许可证。提示词文本与生成的动画代码版权归各自作者所有；使用前请自行确认授权范围。
