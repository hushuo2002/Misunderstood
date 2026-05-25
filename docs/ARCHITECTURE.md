# 推荐架构与技术栈

## 结论

当前最优路线是：**保留 Vite + React，升级为 TypeScript + 内容驱动 + 状态机式游戏核心**。

不建议现在直接迁移到 Next.js。这个 MVP 的主要复杂度在游戏内容、状态流、动画和资源生成，而不是 SSR、API routes 或复杂路由。换到 Next.js 会增加目录和部署心智负担，但不会直接解决扩卡、图片生成和游戏逻辑分散的问题。

## 推荐技术栈

### 当前阶段

- Vite: 保持快速启动和静态部署。
- React: 保持现有组件投资。
- TypeScript: 让卡牌、游戏状态、动作和资源 manifest 有类型约束。
- Zod: 校验卡牌 JSON/TS 数据，避免运行时才发现缺字段。
- Vitest: 测纯规则、抽卡、胜负、schema。
- Testing Library: 测核心交互。
- Playwright: 测完整游戏流程和视觉冒烟。
- GSAP: 保留，但从组件中抽成可测试的 animation adapter。

### 可选升级

当项目出现下面需求时，再考虑 Next.js/Remix：

- 卡牌编辑后台
- 用户登录
- 云端存档
- 服务端图片生成任务
- 多人房间 API
- 分享页 SEO

如果优先做多人而不是后台，候选方案是：

- Vite + PartyKit: 轻量实时房间。
- Vite + Supabase Realtime: 登录、数据库和实时同步一体化。
- Next.js + Supabase: 当后台、账号、分享页也变重要时。

## 当前代码观察

当前项目有两套游戏状态模型：

- `src/App.jsx` 中用 `useState` 维护实际运行的游戏状态。
- `src/utils/gameReducer.js` 和 `src/logic/gameLogic.js` 中已有 reducer/pure logic 的雏形。

建议后续统一到一个 `domain/game` 模块，由 UI 只 dispatch actions，不直接计算分数、抽卡、切回合。

## 目标分层

```txt
src/
  app/                 React shell, screens, providers
  domain/              pure rules, state machine, selectors
  content/             decks, cards, intro data
  i18n/                localization resources
  styles/              visual system and feature CSS
  utils/               generic helpers only
```

### app 层

只处理：

- 当前 screen
- 事件绑定
- 展示组件组合
- i18n 读取
- 调用 domain selectors

不处理：

- 抽卡规则
- 分数规则
- 胜负判断
- AI 命中率
- 图片路径推导

### domain/game 层

负责：

- 游戏状态类型
- action 类型
- reducer 或 state machine
- 抽卡
- AI 模拟
- 计分
- 胜负判断
- 剩余牌计数

### domain/cards 层

负责：

- 卡牌 schema
- deck schema
- 卡牌注册表
- 按 deck/side/tag/difficulty 查询
- 资源路径和 manifest 校验

### animation 层

负责：

- 卡牌抽出 timeline
- 分数弹跳 timeline
- deck wobble timeline
- screen transition tokens

组件只传 DOM ref 和动作参数，不直接拼 GSAP 细节。

## 目标目录

```txt
src/
  app/
    App.tsx
    screens/
      HomeScreen.tsx
      DeckSelectScreen.tsx
      IntroScreen.tsx
      TeamSelectScreen.tsx
      ScoreSelectScreen.tsx
      LoadingScreen.tsx
      GameScreen.tsx
      ResultScreen.tsx
    components/
      Header.tsx
      CardView.tsx
      DeckLibrary.tsx
      ScorePanel.tsx
      OptionGrid.tsx
  domain/
    game/
      types.ts
      actions.ts
      reducer.ts
      rules.ts
      ai.ts
      selectors.ts
      constants.ts
    cards/
      schema.ts
      registry.ts
      selectors.ts
      image-manifest.ts
    animation/
      timelines.ts
      motion-tokens.ts
  content/
    decks/
      index.ts
    cards/
      food.ts
      gesture.ts
      culture.ts
      custom.ts
      sports.ts
      politics.ts
    intro/
      slides.ts
  i18n/
  styles/
scripts/
  generate-card-images.mjs
  validate-content.mjs
  build-card-manifest.mjs
```

## Migration 顺序

1. 把卡牌和卡组类型迁到 TypeScript。
2. 用 Zod 校验现有数据。
3. 合并 `App.jsx`、`utils/gameReducer.js`、`logic/gameLogic.js` 的状态逻辑。
4. 把抽卡、AI、计分、胜负判断移动到 `domain/game`。
5. 把 GSAP timeline 从 `GameScreen` 抽到 `domain/animation` 或 `app/animation`。
6. 改造图片生成脚本为 manifest 驱动。
7. 加 Playwright 冒烟测试：从首页到结算。
