# 内容与图片生成流水线

## 目标

让新增卡牌变成稳定流程：

1. 写卡牌内容。
2. 运行校验。
3. 生成缺失图片。
4. 预览。
5. 进入游戏。

不应该为了新增卡牌去改 `App.jsx`、游戏规则或图片路径工具。

## 当前状态

当前脚本：

```bash
npm run generate:images
```

实际执行：

```bash
node scripts/generate-images.mjs
```

能力：

- 读取 `APINEBULA_API_KEY`。
- 从 `src/data/index.js` 读取 decks、cards、earthCards。
- 生成 deck cover 和 card image。
- 已存在 PNG 时跳过。
- 失败时记录 failed。

限制：

- 不能按 deck/card 过滤。
- 没有 dry run。
- 没有 manifest。
- 没有 prompt 版本。
- 没有重试和速率控制。

当前已新增：

- `npm run content:validate` 校验 deck、Moon cards、Earth cards 和 image job 数量。
- 图片生成脚本复用同一个 image job builder，避免校验和生成各自维护一套数据拼装逻辑。

## 推荐命令

目标脚本：

```bash
npm run content:validate
npm run content:manifest
npm run generate:images -- --only-missing
npm run generate:images -- --deck food
npm run generate:images -- --card food-001
npm run generate:images -- --dry-run
npm run generate:images -- --force --card food-001
```

## 目标脚本结构

```txt
scripts/
  validate-content.mjs
  build-card-manifest.mjs
  generate-card-images.mjs
  lib/
    load-content.mjs
    apinebula-client.mjs
    prompt-builder.mjs
    asset-paths.mjs
```

## Manifest

推荐生成：

```json
{
  "generatedAt": "2026-05-24T00:00:00.000Z",
  "promptVersion": "space-archive-v1",
  "assets": [
    {
      "id": "food-001",
      "kind": "card",
      "deckId": "food",
      "side": "moon",
      "output": "public/assets/cards/food-001.png",
      "promptHash": "..."
    }
  ]
}
```

作用：

- 判断图片是否由最新 prompt 生成。
- 支持只重生成过期图片。
- 记录失败任务。
- 方便后续接入缩略图或预览页。

## Prompt 模板

不要在每张卡里重复全部风格词。推荐三层 prompt：

### Global style

```txt
Retrofuturistic space archive terminal game art, 1950s-60s space race poster influence, polished anime card illustration, clean shapes, cinematic lighting, no readable text, no watermark, no logo.
```

### Deck style

```txt
food: lunar cafeteria, hydroponics, water conservation, fermentation tanks, clean habitat lighting
gesture: EVA suits, tactile signals, low-gravity body language, helmet reflections
politics: oxygen valves, council chamber, habitat maps, serious governance atmosphere
```

### Card scene

只描述这一张卡的画面，不泄露正确答案文字。

## 输出路径

推荐固定：

```txt
public/assets/cards/{cardId}.png
public/assets/decks/{deckId}.png
```

如果生成多版本：

```txt
public/assets/generated/cards/{promptVersion}/{cardId}.png
```

但游戏运行时仍通过 manifest 映射到当前版本。

## 安全规则

- 使用 `APINEBULA_API_KEY` 环境变量。
- `.env` 不提交。
- `.env.example` 只写变量名。
- 生成脚本不能打印完整 key。
- 失败日志不能包含敏感请求头。

## 建议的新增卡流程

```bash
# 1. add or edit content
npm run content:validate

# 2. inspect planned jobs
npm run generate:images -- --dry-run --only-missing

# 3. generate only missing assets
npm run generate:images -- --only-missing

# 4. build and test
npm run build
npm test
npm run content:validate
```

## 卡面图像质量标准

- 不出现可读文字。
- 不出现 logo 或水印。
- 主体清晰，卡牌缩小后仍能看懂。
- 颜色符合 deck 主题，但保持整体“太空档案终端”统一。
- 画面表达场景，不直接把答案画得过于直白。
- Earth cards 和 Moon cards 应有不同环境线索。
