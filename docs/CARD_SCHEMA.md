# 卡牌数据结构

## 目标

卡牌数据应该支持三件事：

1. 游戏运行时稳定读取。
2. 内容创作者容易新增和校验。
3. 图片生成脚本能直接读取 prompt 和目标路径。

## 推荐 TypeScript 类型

```ts
export type DeckId =
  | "all"
  | "gesture"
  | "culture"
  | "custom"
  | "food"
  | "sports"
  | "politics";

export type CardSide = "moon" | "earth";

export type CardDifficulty = 1 | 2 | 3;

export interface GameCard {
  id: string;
  deckId: Exclude<DeckId, "all">;
  side: CardSide;
  title: string;
  scenario: string;
  options: [string, string, string, string];
  answerIndex: 0 | 1 | 2 | 3;
  explanation: string;
  deeperMeaning: string;
  difficulty: CardDifficulty;
  tags: string[];
  art: {
    image: string;
    prompt: string;
    negativePrompt?: string;
    stylePreset?: string;
  };
}
```

## 当前兼容字段

当前代码中卡牌字段是：

```ts
{
  id: string;
  deck: string;
  title: string;
  scenario: string;
  options: string[];
  answerIndex: number;
  explanation: string;
  deeperMeaning: string;
  difficulty: number;
  image: string;
  imagePrompt: string;
}
```

迁移时可以先做 adapter：

```ts
function normalizeLegacyCard(card): GameCard {
  return {
    id: card.id,
    deckId: card.deck,
    side: card.id.startsWith("earth-") ? "earth" : "moon",
    title: card.title,
    scenario: card.scenario,
    options: card.options,
    answerIndex: card.answerIndex,
    explanation: card.explanation,
    deeperMeaning: card.deeperMeaning,
    difficulty: card.difficulty,
    tags: [],
    art: {
      image: card.image,
      prompt: card.imagePrompt,
    },
  };
}
```

## Deck Schema

```ts
export interface Deck {
  id: DeckId;
  name: string;
  subtitle: string;
  description: string;
  difficulty: "入门" | "标准" | "进阶";
  themeColor: string;
  art: {
    coverImage: string;
    coverPrompt: string;
    stylePreset?: string;
  };
}
```

## ID 规则

推荐：

```txt
moon cards:  {deck}-{number}
earth cards: earth-{deck}-{number}
```

示例：

```txt
food-001
food-002
earth-food-001
earth-food-002
```

规则：

- `id` 全局唯一。
- deck 内编号三位数递增。
- 不在 id 中加入标题，避免改标题导致资源路径变化。
- 图片路径默认由 id 推导：`/assets/cards/{id}.png`。

## 必填校验

每张卡必须满足：

- `id` 唯一。
- `deckId` 存在于 deck registry。
- `side` 是 `earth` 或 `moon`。
- `options.length === 4`。
- `answerIndex` 在 0-3。
- `title/scenario/explanation/deeperMeaning` 非空。
- `art.prompt` 非空。
- `art.image` 指向 `public/assets/cards/` 下的 PNG。

## 内容质量标准

每张卡都要有：

- 明确误解点：玩家为什么会误判。
- 正确解释：这件事在当地文化中是什么意思。
- 环境根因：重力、水、空气、空间、时间、风险或制度如何塑造这个习俗。
- 错误选项：不能只是离谱，要像真实误解。
- 图片 prompt：描述场景而不是描述答案。

队员填写新题目时，优先使用 [卡牌问题设计表](CARD_DESIGN_TEMPLATE.md)，或直接导入 [CSV 模板](card-design-template.csv) 到表格软件。

## i18n 策略

推荐长期结构：

```txt
content/cards/food.ts              canonical ids and mechanics
i18n/locales/zh/cards/food.json    zh display text
i18n/locales/en/cards/food.json    en display text
```

游戏逻辑只读取 id、deckId、side、difficulty、answerIndex、image。  
UI 文案通过 id 从 i18n 中读取。
