# StarVoyage — 互动太阳系 iOS P0

一个面向天文兴趣用户的 iOS 互动太阳系原型。用户可以旋转、缩放和聚焦太阳与八大行星，查看天体知识卡，在转动行星后获得最多三个推荐问题，并通过长按话筒向 AI 提问。

> 当前状态：仅完成 P0 产品与技术规划，应用代码尚未初始化。  
> 目标：7 天完成指定 iPhone 可用的内部测试版。  
> 分发：Xcode 私有安装，不公开上架 App Store。

完整范围、排期和验收标准见 [`P0_PLAN.md`](./P0_PLAN.md)。

## P0 核心流程

```text
浏览太阳系
→ 选择并聚焦行星
→ 转动行星
→ 查看知识卡与最多3个推荐问题
→ 点击问题，或长按话筒自由提问
→ AI返回简短中文文字答案
```

## P0 功能

### 1. 太阳系 3D 探索

- 太阳与八大行星；
- 教学比例和简化轨道；
- 行星自转与简化公转；
- 单指旋转镜头；
- 双指缩放；
- 点击选中、聚焦和返回全景；
- 聚焦后旋转观察当前天体。

### 2. 知识卡与推荐问题

- 九个主要天体的本地知识卡；
- 每个天体最多三个预设问题；
- 用户转动天体并停止后显示问题；
- 点击问题后调用 AI 回答。

### 3. 语音 AI 提问

- 长按话筒开始、松开结束；
- Apple Speech 将普通话转成文字；
- 当前天体和问题提交到 Cloudflare Worker；
- Gemini 根据审核知识包生成简短答案；
- iOS 页面显示文字答案；
- P0 不保存录音和聊天历史。

## P0 不包含

- 数据库、CMS 或向量检索；
- 用户账号、收藏、进度与埋点；
- AI 语音朗读和持久化多轮聊天；
- 真实比例和指定日期精确星历；
- 卫星、小行星、黑洞、银河系和 AR；
- Android、iPad 或公开 App Store 上架。

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

## 为什么没有数据库

P0 只有太阳和八大行星，知识规模很小，可以随 iOS App 和 Worker 作为静态文件部署。P0 不保存用户、收藏、进度、录音或聊天记录，因此不需要数据库。

数据库仅在未来需要账号、跨设备同步、大规模天体内容、持久化聊天或完整 RAG 时引入。

## 计划中的仓库结构

应用代码初始化后，仓库计划采用：

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

> 上述 `ios/` 和 `worker/` 目录尚未创建，是后续开发阶段的目标结构。

## 前置条件

### 本地环境

- macOS；
- 可用的 Xcode；
- 一台目标测试 iPhone；
- iPhone 已开启开发者模式；
- Apple ID 已登录 Xcode；
- Node.js，用于 Worker 开发；
- Git。

### 云端账号

- Cloudflare 账号；
- Google AI Studio/Gemini API Key。

### 开发前确认

- 目标 iPhone 型号和 iOS 版本；
- Gemini API 在当前网络中可访问；
- 行星纹理来源及使用说明；
- 唯一的 iOS Bundle Identifier。

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
sun
mercury
venus
earth
mars
jupiter
saturn
uranus
neptune
```

## 本地知识数据示例

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

所有科学数据在进入测试版本前需要人工复核，并记录来源。

## Worker 环境变量

真实密钥不得提交到 Git。后续 Worker 初始化后，使用 Secret 配置：

```bash
npx wrangler secret put GEMINI_API_KEY
```

本地开发可以使用不提交版本控制的 `.dev.vars`：

```text
GEMINI_API_KEY=replace_with_local_key
```

计划加入 `.gitignore`：

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

## Worker 开发与部署

以下命令在 `worker/` 工程创建后使用：

```bash
npm install
npx wrangler dev
npx wrangler deploy
```

Cloudflare Worker 负责：

- 隐藏 Gemini API Key；
- 校验天体 ID；
- 限制问题长度；
- 限制请求频率；
- 注入当前天体的审核知识；
- 统一返回用户可理解的错误；
- 不保存问题或答案。

## iOS 私有安装

P0 使用 Xcode Personal Team 安装，不公开上架：

1. 使用 Xcode 打开项目；
2. 在 Signing & Capabilities 中选择 Personal Team；
3. 设置唯一 Bundle Identifier；
4. 连接目标 iPhone；
5. 在 iPhone 开启开发者模式；
6. 选择真机并运行；
7. 根据系统提示信任开发者。

免费签名有效期较短，需要定期通过 Xcode 重新签名和安装。

## 隐私与安全原则

- 仅在用户长按话筒时启用麦克风；
- 不保存原始录音；
- 不保存聊天历史；
- 不收集姓名、位置、学校等个人信息；
- Gemini API Key 只存放在 Worker Secret；
- 免费模型服务的数据条款在扩大测试前需要再次确认；
- 儿童使用时，只发送当前天体 ID 与问题文本；
- AI 回答限定为当前天体相关的天文学内容。

## 开发计划

推荐 7 天内部测试版：

| 天数 | 目标 |
|---|---|
| Day 1 | Xcode工程、太阳和八大行星、真机运行 |
| Day 2 | 旋转、缩放、选中、聚焦与返回全景 |
| Day 3 | 知识卡、静态数据、推荐问题与触发逻辑 |
| Day 4 | 长按话筒与 Apple Speech |
| Day 5 | Cloudflare Worker、Gemini与AI回答 |
| Day 6 | 异常处理、性能与回答边界 |
| Day 7 | 用户测试、修复与最终私有安装 |

4 天演示版会减少异常处理、视觉优化和设备适配，仅适合演示。详细计划见 [`P0_PLAN.md`](./P0_PLAN.md#11-七天交付计划)。

## 当前状态

- [x] 明确 P0 产品范围
- [x] 明确 iOS 与后端技术方案
- [x] 明确 7 天计划和验收标准
- [ ] 初始化 Xcode 工程
- [ ] 初始化 Cloudflare Worker
- [ ] 完成太阳系 3D 场景
- [ ] 完成知识卡与推荐问题
- [ ] 完成语音提问
- [ ] 完成 Gemini AI 回答
- [ ] 完成目标 iPhone 真机验收

## 文档

- [P0 产品与交付计划](./P0_PLAN.md)

## 许可

当前项目用于非商业原型验证，尚未选择开源许可证。未经明确授权，不应假定仓库代码、设计或第三方素材可以被重新分发。