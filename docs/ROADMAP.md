# 开发路线图

## Phase 0: 文档与方向统一

- [x] 明确继续单人优先。
- [x] 明确短期不迁移到重框架。
- [x] 记录当前游戏逻辑、动画和数值规则。
- [x] 设计目标技术栈和目标目录。
- [x] 设计内容与图片生成流水线。

## Phase 1: 工程化内容系统

- [ ] 引入 TypeScript。
- [ ] 定义 `GameCard`、`Deck`、`GameState` 类型。
- [ ] 用 Zod 校验卡牌和 deck。
- [ ] 给卡牌增加 `side`、`tags`、`art` 字段。
- [ ] 写 `content:validate` 脚本。
- [ ] 把图片路径改为由 id 或 manifest 推导。

## Phase 2: 游戏核心重构

- [ ] 合并 `App.jsx`、`utils/gameReducer.js`、`logic/gameLogic.js` 的重复逻辑。
- [ ] 建立单一 reducer 或 state machine。
- [ ] 把抽卡、计分、胜负、AI 决策移到 `domain/game`。
- [ ] 让 UI 只 dispatch action。
- [ ] 引入可注入 RNG，支持测试和回放。
- [ ] 修正牌库剩余数量为真实卡池长度。

## Phase 3: 动画系统整理

- [ ] 把卡牌抽取 GSAP timeline 从 `GameScreen` 抽离。
- [ ] 用 ref 替代 DOM selector。
- [ ] 建立 motion tokens。
- [ ] 支持 reduced motion。
- [ ] 移除生产环境 console log。
- [ ] 加 Playwright 视觉冒烟检查。

## Phase 4: 图片生成流水线

- [ ] 重写 `generate-images.mjs` 为参数化脚本。
- [ ] 支持 `--deck`、`--card`、`--only-missing`、`--dry-run`、`--force`。
- [ ] 生成 manifest。
- [ ] 记录 prompt hash。
- [ ] 增加失败重试和速率控制。
- [ ] 生成卡牌预览页或 contact sheet。

## Phase 5: 内容扩展

- [ ] 每个 deck 至少 12 张 Moon cards。
- [ ] 每个 deck 至少 12 张 Earth cards。
- [ ] 增加 difficulty 3 卡。
- [ ] 给卡牌加 tags。
- [ ] 增加医疗、教育、家庭、法律等新卡组。
- [ ] 优化中英文翻译一致性。

## Phase 6: 多人原型

只有当单人内容系统稳定后再做。

- [ ] 决定实时方案：PartyKit、Supabase Realtime 或 Next.js backend。
- [ ] 房间码。
- [ ] 阵营加入。
- [ ] 回合同步。
- [ ] 组内投票。
- [ ] 掉线恢复。

## 技术债优先级

高优先级：

- 游戏状态统一。
- 卡牌 schema 校验。
- 图片生成可增量运行。
- 牌库数量和真实卡池一致。

中优先级：

- 动画 timeline 抽离。
- i18n 与内容 schema 对齐。
- Playwright 冒烟。

低优先级：

- 换框架。
- 后台。
- 多人。
