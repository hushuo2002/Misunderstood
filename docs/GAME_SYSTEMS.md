# 游戏系统、状态、动画与数值规则

## Screen Flow

当前主流程：

```txt
home
  -> deckSelect
  -> intro
  -> teamSelect
  -> scoreSelect
  -> loading
  -> game
  -> result
```

房间入口当前是占位流程：

```txt
home -> room -> deckSelect
```

## 关键状态

当前实际运行状态主要在 `src/App.jsx` 中维护：

- `screen`: 当前页面。
- `selectedDeckId`: 当前选择的卡组。
- `introIndex`: 世界观导入页索引。
- `selectedTeam`: 玩家阵营。
- `selectedWinScore`: 胜利分数。
- `loadingDeckId`: loading 阶段锁定的卡组。
- `loadingTeam`: loading 阶段锁定的玩家阵营。
- `loadingFirstTeam`: 随机得到的先手阵营。
- `game`: 对局状态。

`game` 对象包含：

- `deckId`
- `playerTeam`
- `winScore`
- `score.earth`
- `score.moon`
- `round`
- `currentTeam`
- `currentMoonCard`
- `currentEarthCard`
- `usedMoonIds`
- `usedEarthIds`
- `selectedOption`
- `revealed`
- `winner`
- `showResultPopup`
- `moonShowResultPopup`
- `aiPendingResult`
- `moonCardsRemaining`
- `earthCardsRemaining`
- `aiAutoTriggered`

## 当前对局模型

地球组和月球组互相答对方文化题：

- Earth team 回答 Moon cards。
- Moon team 回答 Earth cards。

`currentTeam === "earth"` 时，页面展示 `currentMoonCard`。  
`currentTeam === "moon"` 时，页面展示 `currentEarthCard`。

## 抽卡规则

当前函数：

- `drawMoonCard(deckId, usedIds)`
- `drawEarthCard(deckId, usedIds)`

规则：

1. 如果 `deckId === "all"`，从全部对应 side 卡池抽。
2. 否则只从指定 deck 抽。
3. 优先排除 `usedIds`。
4. 如果未使用卡池为空，则回退到完整卡池。
5. 使用 `Math.random()` 随机抽取。

重构建议：

- 引入可注入 RNG，方便测试和回放。
- `usedIds` 更新只由 reducer/state machine 负责。
- 卡池为空时应显式返回 exhausted 状态，而不是静默回到完整卡池，除非设计上允许循环牌库。

## 开局规则

当前开局：

1. 玩家选择 deck。
2. 玩家选择阵营。
3. 玩家选择 `winScore`，可选 3/5/7/9 或 1-20 自定义。
4. `Math.random() < 0.5` 决定先手。
5. 抽初始 `moonCard` 和 `earthCard`。
6. 根据先手扣掉对应牌库剩余数量。

注意：

- 代码里 `CARDS_PER_SIDE = 12`，但当前每个单独 deck 的实际卡数少于 12。短期展示可接受，正式规则应让剩余数量来自实际卡池长度。
- `WIN_SCORE = 5` 仍存在，但实际对局使用玩家选择的 `selectedWinScore`。

## 答题与计分

玩家或 AI 做出 decision：

```txt
selectedOption === currentCard.answerIndex -> correct
correct -> 当前 team +1 分
```

当前玩家答题：

- 只能在 `game.currentTeam === game.playerTeam` 时答。
- 已 reveal 后不能重复答。
- 答完立即 reveal。
- Earth 成功弹 `earth-success`，失败弹 `earth-miss`。

当前 AI 答题：

- 在 UI 中等待约 3000ms 后触发。
- `simulateAiTurn` 随机选择 0-3。
- 实际正确率约 25%，不是文档旧版里写的 65%。
- AI 答完写入 `aiPendingResult` 和 `moonShowResultPopup`。

建议：

- 把 AI 正确率做成常量或 difficulty-based 策略。
- 如果想让 AI 看起来像对手，推荐基础正确率 50%-65%。
- 如果想强调“月球也会误解地球”，推荐根据 deck/difficulty 计算。

## 回合切换

当前 `handleNextTurn` 规则：

1. 如果当前还没 reveal，不允许进入下一轮。
2. 如果任一方达到 `winScore`，进入 result。
3. 如果任一牌库剩余数为 0，进入 result。
4. 否则切换 `currentTeam`。
5. 抽下一张 Moon card 和 Earth card。
6. 根据刚完成回合的 team 扣对应剩余牌数。
7. 清空选项、reveal、popup、AI pending。

注意：

- 当前每次切换都会同时抽 `nextMoonCard` 和 `nextEarthCard`，即使下一回合只展示其中一张。
- `usedMoonIds` 和 `usedEarthIds` 在多个位置更新，容易重复记录。

建议：

- 每个 turn 只抽当前 team 需要的 card。
- 把 `DRAW_CARD`、`ANSWER_CARD`、`ADVANCE_TURN`、`FINISH_GAME` 拆成明确 action。
- 不在组件里直接改 `usedIds`。

## 胜负规则

当前实际逻辑：

- 任一方分数达到 `winScore`，结算。
- 或任一方牌库剩余数为 0，结算。
- `earth > moon` 则 Earth 胜。
- `moon > earth` 则 Moon 胜。
- 相等则 tie。

历史文档中“双方各 12 张，抽完后结算”和“先到 5 分”曾同时出现。当前建议统一为：

- 默认短局：先到 `winScore`。
- Deck endurance 模式：抽完结算。
- 两种模式作为配置，不要混在同一套规则里。

## 动画系统

当前使用 GSAP。

### Screen transition

`navigateTo` 中用 `isTransitioning` 和 `heroAnimClass` 控制 400ms 页面切换。

### Home

- 首屏有 orbit rings、Earth/Moon visual、data stream。
- 返回 home 时有 `entering` class。
- 使用 idle callback 预加载 deck cover。

### Card draw

`GameScreen` 中直接创建 GSAP timeline：

1. 读取 source deck 的 DOM rect。
2. 把主卡片设置到 deck 中心附近。
3. `scale: 0.4`, `rotateY: 180`, `opacity: 0`。
4. 飞到视口中心，放大到 `1.3`。
5. 稍作停留放大到 `1.35`。
6. 翻面到 `rotateY: 0`，scale 到 `1.2`。
7. 回到布局位置，scale 到 `1`。

### Deck wobble

`useDeckWobble(currentTeam, cardId)`：

- 卡牌 id 变化时触发。
- 根据 current team 找对应 deck stack。
- `rotate -3 -> 3`，0.1s，repeat 3，yoyo。

### Score animation

`useScoreAnimation(showResultPopup, moonShowResultPopup, score)`：

- 有 popup 时找到对应 score 数字。
- `scale 1 -> 1.4 -> 1`。

建议：

- 把 DOM selector 改成 ref。
- Timeline 参数集中到 `motion-tokens.ts`。
- 给 prefers-reduced-motion 加降级。
- 去掉生产环境 console log。

## 数值常量

当前常量：

- `CARDS_PER_SIDE = 12`
- `WIN_SCORE = 5`
- AI 选项随机范围：4 个选项
- AI 触发延迟：3000ms
- 页面切换延迟：400ms
- score 可选值：3/5/7/9
- 自定义 score 范围：1-20

建议集中成：

```ts
export const GAME_BALANCE = {
  defaultWinScore: 5,
  scoreOptions: [3, 5, 7, 9],
  customScoreMin: 1,
  customScoreMax: 20,
  aiThinkDelayMs: 3000,
  screenTransitionMs: 400,
};

## 未确认的视觉问题

### DeckSelect 页面底部按钮处有矩形硬边缘

**现象**：DeckSelect 页面的"接入档案"按钮上方，deck 卡片的青色光效与底部渐变之间出现明显的矩形边缘，看着像是光效被某个不透明层截断。

**排查记录**：

1. **deck-card::before** (`_home.css:30-37`)
   - `z-index: 1`，gradient 从 `transparent 30%` 到 `rgba(7,11,18,0.95) 100%`
   - 曾尝试将 z-index 改为 -1 让 box-shadow 光效透出，但未完全解决

2. **footer-actions gradient** (`_shared.css:730`)
   - 使用 `linear-gradient(to top, rgba(7, 11, 18, 0.98) 60%, transparent)` 硬边缘
   - 已改为柔和渐变 `0%→30%→70%→100%`

3. **媒体查询中的 footer-actions** (`styles.css:59, 346`)
   - `@media (max-width: 900px)` 和 `@media (max-width: 620px)` 内也有相同的硬边缘渐变
   - 已同步改为柔和渐变

4. **--bg CSS 变量** (`_base.css:11`)
   - `--bg: #070b12`
   - 用户发现注释掉 `--bg` 变量后问题消失，疑似与 `var(--bg)` 在媒体查询中的使用有关

**待验证**：
- 将 `var(--bg)` 替换为硬编码颜色，或确认媒体查询中渐变是否完全生效
- 检查是否有其他 CSS 变量或层叠上下文导致矩形边缘
```
