# StarVoyage — 互动太阳系 iOS P0

一个面向天文兴趣用户的 iOS 互动太阳系原型：观察太阳与八大行星，转动天体触发问题引导，并通过长按话筒向 AI 提问。

> [!IMPORTANT]
> 当前仓库处于 **P0 规划完成、代码尚未初始化** 状态。目标是在 7 天内完成一台指定 iPhone 可用的内部测试版；项目通过 Xcode 私有安装，不公开上架 App Store。

- [完整 P0 产品、排期与验收计划](./P0_PLAN.md)
- 开发模式：1 名产品经理 + AI 开发协作
- 使用范围：非商业原型验证

## 核心体验

```text
浏览太阳系
→ 选择并聚焦行星
→ 转动行星
→ 查看知识卡与最多3个推荐问题
→ 点击问题，或长按话筒自由提问
→ AI返回简短中文文字答案
```

## P0 功能

### 太阳系 3D 探索

- 太阳与八大行星；
- 教学比例和简化轨道；
- 行星自转与简化公转；
- 单指旋转、双指缩放；
- 点击选中、聚焦和返回全景；
- 聚焦后旋转观察当前天体。

### 知识卡与问题引导

- 九个主要天体的本地知识卡；
- 每个天体最多三个预设问题；
- 用户转动天体并停止后显示问题；
- 点击问题后调用 AI 回答。

### 语音 AI 提问

- 长按话筒开始、松开结束；
- Apple Speech 将普通话转成文字；
- 将当前天体与问题提交到 Cloudflare Worker；
- Gemini 根据审核知识包生成简短答案；
- iOS 显示文字答案，不保存录音和聊天历史。

## 当前状态

| 阶段 | 状态 |
|---|---|
| P0 范围与验收标准 | 已完成 |
| iOS 与后端技术方案 | 已完成 |
| Xcode 工程 | 未开始 |
| Cloudflare Worker | 未开始 |
| 太阳系 3D 场景 | 未开始 |
| 知识卡与推荐问题 | 未开始 |
| 语音提问与 AI 回答 | 未开始 |
| 目标 iPhone 真机验收 | 未开始 |

详细任务顺序见 [`P0_PLAN.md`](./P0_PLAN.md#11-七天交付计划)。

## 技术栈

### iOS

| 类型 | 技术 |
|---|---|
| IDE | Xcode |
| 语言 | Swift |
| UI | SwiftUI |
| 3D | RealityKit |
| 音频 | AVFoundation |
| 语音识别 | Apple Speech |
| 网络 | URLSession |
| 本地数据 | JSON + Codable |

### 后端

| 类型 | 技术 |
|---|---|
| 平台 | Cloudflare Workers |
| 语言 | TypeScript |
| AI | Gemini Developer API |
| 知识 | 静态 TypeScript/JSON |
| 数据库 | 无 |

## 系统架构

```text
iPhone
├── SwiftUI：知识卡、问题、话筒、答案
├── RealityKit：太阳系3D场景
├── Local JSON：知识卡与推荐问题
└── Apple Speech：语音转文字
        │
        │ POST /api/ask
        ▼
Cloudflare Worker
├── 校验planetId和问题
├── 读取服务端审核知识包
├── 组合提示词
├── 保护Gemini API Key
└── 调用Gemini API
        │
        ▼
iPhone显示AI文字答案
```

P0 只有九个主要天体，并且不保存用户、进度、录音或聊天记录，因此暂不需要数据库。未来需要账号、跨设备同步、大规模内容或完整 RAG 时再引入。

## 计划中的仓库结构

```text
.
├── README.md
├── P0_PLAN.md
├── ios/
│   ├── StarVoyage.xcodeproj
│   └── StarVoyage/
│       ├── App/
│       ├── Features/
│       │   ├── SolarSystem/
│       │   ├── PlanetCard/
│       │   ├── SuggestedQuestions/
│       │   └── VoiceQuestion/
│       ├── Services/
│       │   ├── SpeechService.swift
│       │   └── AIService.swift
│       └── Resources/
│           ├── Knowledge/
│           └── Textures/
└── worker/
    ├── src/
    │   ├── index.ts
    │   ├── planets.ts
    │   └── prompt.ts
    ├── package.json
    └── wrangler.jsonc
```

> `ios/` 和 `worker/` 尚未创建，以上是后续工程初始化的目标结构。

## Getting Started

### 前置条件

- macOS 与可用的 Xcode；
- 一台开启开发者模式的测试 iPhone；
- Apple ID 已登录 Xcode；
- Node.js 与 Git；
- Cloudflare 账号；
- Google AI Studio/Gemini API Key；
- 当前网络能够访问所选 AI API。

还需要在开发开始前确认：

- 目标 iPhone 型号和 iOS 版本；
- 唯一的 iOS Bundle Identifier；
- 行星纹理的来源与使用条件。

### 当前仓库

目前仓库仅包含规划文档，没有可构建的 Xcode 或 Worker 工程。代码初始化后，本节将更新为可直接复制执行的启动步骤。

### 计划中的 iOS 启动方式

1. 使用 Xcode 打开 `ios/StarVoyage.xcodeproj`；
2. 在 Signing & Capabilities 中选择 Personal Team；
3. 配置唯一 Bundle Identifier；
4. 选择目标 iPhone；
5. 构建并运行。

### 计划中的 Worker 启动方式

工程创建后，在 `worker/` 目录执行：

```bash
npm install
npx wrangler dev
```

部署测试 Worker：

```bash
npx wrangler deploy
```

## 配置与密钥

真实密钥不得提交到 Git。Gemini Key 通过 Cloudflare Secret 配置：

```bash
npx wrangler secret put GEMINI_API_KEY
```

本地 Worker 开发使用不纳入版本控制的 `.dev.vars`：

```text
GEMINI_API_KEY=replace_with_local_key
```

代码初始化时将创建 `.gitignore`，至少包含：

```gitignore
.DS_Store
DerivedData/
*.xcuserstate
xcuserdata/
node_modules/
.dev.vars
.env
.env.*
```

## 后端接口

### `POST /api/ask`

请求：

```json
{
  "planetId": "jupiter",
  "question": "人可以站在木星上吗？"
}
```

成功响应：

```json
{
  "planetId": "jupiter",
  "answer": "不能。木星主要由气体组成，没有像地球一样坚硬的地面。"
}
```

失败响应：

```json
{
  "error": {
    "code": "AI_UNAVAILABLE",
    "message": "暂时无法回答，请稍后再试。"
  }
}
```

允许的 `planetId`：

```text
sun, mercury, venus, earth, mars,
jupiter, saturn, uranus, neptune
```

Worker 不保存问题或答案，并负责校验天体 ID、限制问题长度、注入审核知识及保护 Gemini API Key。

## 本地知识格式

```json
{
  "id": "jupiter",
  "name": "木星",
  "englishName": "Jupiter",
  "summary": "木星是太阳系中最大的行星，主要由氢和氦组成。",
  "diameter": "约139,820千米",
  "temperature": "云顶平均约-110°C",
  "rotationPeriod": "约10小时",
  "orbitalPeriod": "约11.86个地球年",
  "facts": [
    "木星没有可以让人站立的固体表面。",
    "木星的大红斑是一场规模巨大的风暴。"
  ],
  "questions": [
    "木星为什么这么大？",
    "大红斑为什么一直没有消失？",
    "人可以站在木星上吗？"
  ]
}
```

所有科学内容进入测试版本前都需要人工复核并记录来源。

## 私有部署

P0 使用 Xcode Personal Team 安装，不公开上架：

1. 在 Xcode 登录 Apple ID；
2. 连接目标 iPhone；
3. 选择 Personal Team 和真机；
4. 在 iPhone 开启开发者模式；
5. 构建、安装并根据系统提示信任开发者。

免费签名有效期较短，需要定期通过 Xcode 重新签名和安装。

## 常见问题

### 为什么看不到可以运行的项目？

当前只完成规划，Xcode 与 Worker 工程尚未初始化。请查看上方“当前状态”。

### 为什么不直接从 iPhone 调用 Gemini？

把 Gemini API Key 放入 App 后可能被提取和滥用，因此所有 AI 请求都通过 Cloudflare Worker 转发。

### 为什么不使用数据库？

P0 内容少且不保存用户数据。静态 JSON 足够，数据库会增加不必要的开发与维护成本。

### 为什么行星位置不完全真实？

P0 使用教学比例和简化轨道，不接入指定日期的精确星历。

## 隐私与安全

- 仅在用户长按话筒时启用麦克风；
- 不保存原始录音和聊天历史；
- 不收集姓名、位置、学校等个人信息；
- 只向 AI 服务发送当前天体 ID 与问题文本；
- Gemini API Key 只存放在 Worker Secret；
- 免费模型服务的数据条款在扩大测试前需要再次确认；
- 如果密钥意外提交，应立即撤销并重新生成，不能只从最新提交中删除。

## P0 非目标

- 用户账号、收藏、进度、埋点与数据库；
- AI 语音朗读和持久化多轮聊天；
- 真实比例与指定日期精确星历；
- 卫星、小行星、黑洞、银河系与 AR；
- Android、iPad 或公开 App Store 上架；
- 商业化、订阅和支付。

## 反馈与贡献

P0 阶段优先保证范围稳定：

- Bug、文档问题和科学内容纠错可以通过 GitHub Issues 提交；
- 新功能建议可以记录，但默认不进入 P0；
- 提交 Pull Request 前，应先说明修改目标和影响范围；
- 不得提交真实 API Key、个人信息、无来源素材或未经核实的天文内容。

当项目开始接受外部贡献时，将补充独立的 `CONTRIBUTING.md` 和行为准则。

## 数据与素材

- P0 行星知识将整理为本地静态数据，并在内容文件中记录来源；
- 行星纹理、图片、字体和音频必须逐项确认使用条件；
- 当前仓库尚未包含第三方图片、模型、字体或音频素材；
- 后续如使用 NASA/JPL 等公开资源，将在此处和对应素材目录中补充署名与来源说明。

## 文档

- [P0 产品、排期与验收计划](./P0_PLAN.md)

## 后续 README 更新条件

完成首个可运行版本后，应补充：

- 一张真实主界面截图；
- 一段 10～20 秒核心交互 GIF 或视频链接；
- 已验证的 Xcode、iOS 与依赖版本；
- 可复制执行的完整安装步骤；
- 实际仓库目录，而不是计划目录；
- 构建、测试或代码质量状态。

## 许可

当前项目用于非商业原型验证，尚未选择开源许可证。未经明确授权，不应假定仓库代码、设计或第三方素材可以被复制、修改或重新分发。
