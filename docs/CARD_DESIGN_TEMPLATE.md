# 卡牌问题设计表

用于队员填写新卡牌内容。填写完成后，可以按行转换成 `src/data/cards/*.js` 里的卡牌对象。

## 填写规则

- `side`:
  - `moon`: 地球组回答的月球文化题。
  - `earth`: 月球组回答的地球文化题。
- `deck`: 只能填 `gesture`、`culture`、`custom`、`food`、`sports`、`politics`。
- `answerIndex`: 正确选项索引，从 0 开始。
  - `0` = 选项A
  - `1` = 选项B
  - `2` = 选项C
  - `3` = 选项D
- `difficulty`: 建议填 `1`、`2`、`3`。
- `image`: 默认用 `/assets/cards/{id}.png`。
- `imagePrompt`: 只描述画面，不要出现文字、水印、logo，也不要把正确答案写成画面里的可读文本。

## 可复制表格

| id | side | deck | title | scenario | optionA | optionB | optionC | optionD | answerIndex | explanation | deeperMeaning | difficulty | tags | image | imagePrompt |
|---|---|---|---|---|---|---|---|---|---:|---|---|---:|---|---|---|
| food-003 | moon | food |  |  |  |  |  |  |  |  |  | 1 | resource,risk | /assets/cards/food-003.png | Anime card illustration, lunar habitat food culture scene, no readable text, no watermark. |
| earth-food-003 | earth | food |  |  |  |  |  |  |  |  |  | 1 | social,food | /assets/cards/earth-food-003.png | Anime card illustration, Earth food sharing scene observed by lunar residents, no readable text, no watermark. |
| gesture-003 | moon | gesture |  |  |  |  |  |  |  |  |  | 1 | communication,body | /assets/cards/gesture-003.png | Anime card illustration, lunar low-gravity body language misunderstanding, no readable text, no watermark. |
| earth-gesture-003 | earth | gesture |  |  |  |  |  |  |  |  |  | 1 | communication,greeting | /assets/cards/earth-gesture-003.png | Anime card illustration, Earth greeting gesture misunderstood by lunar visitor, no readable text, no watermark. |

## 示例完整行

| id | side | deck | title | scenario | optionA | optionB | optionC | optionD | answerIndex | explanation | deeperMeaning | difficulty | tags | image | imagePrompt |
|---|---|---|---|---|---|---|---|---|---:|---|---|---:|---|---|---|
| custom-003 | moon | custom | 月夜静默 | 月球基地进入长夜周期前，所有人把公共频道静音三分钟。 | 通讯系统故障 | 一种共同降噪和心理准备仪式 | 对地球访客的抗议 | 正在进行秘密投票 | 1 | 长夜周期会改变能源、作息和心理压力，公共频道静音是进入低干扰模式前的集体确认。 | 在月球，环境周期不是背景，而是会直接改变共同体节奏的压力源。仪式常常同时承担情绪管理和风险管理。 | 2 | time,risk,ritual | /assets/cards/custom-003.png | Anime card illustration, lunar habitat entering long night cycle, residents quietly dimming communication panels in a solemn shared ritual, retrofuturistic archive style, no readable text, no watermark. |

## Prompt 建议

推荐结构：

```txt
Anime card illustration, [主体场景], [关键人物动作], [环境线索], retrofuturistic space archive style, no readable text, no watermark, no logo.
```

好的 prompt 会描述：

- 场景地点：月球基地、地球餐厅、会议舱、运动场、气闸等。
- 人物关系：地球访客、月球居民、同事、主持人、观众等。
- 关键动作：递水、停顿、注视、分食、调暗窗景等。
- 环境线索：低重力、水培、氧气阀、头盔、终端、地球窗景等。

不要写：

- 画面里出现中文/英文答案。
- “正确答案是……”
- 具体品牌、logo、水印。
