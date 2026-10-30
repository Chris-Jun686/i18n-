
# 文本管理与存档

## 1. 关于文本管理与存档

游戏里给玩家看的每一句话、每个按钮名和道具名，都不要直接写在程序里，而是集中记在一张清单上，程序只记一个固定编号，显示时按编号去清单里取当前语言的文字。一句话要整句记，数字和名字留空由程序填，不能把句子拆开拼。存档时也只记物品、任务等的编号和数量，不记玩家看到的文字，这样换语言后旧存档仍能显示正确语言。

## 2. 前作纰漏

### 2.1 文本表与统一表述

> *以下 3 处同样表达属性提升的文本出现 3 种不同表述。*

**问题编号-23** </br>
<img width="600" height="300" alt="LQA-023" src="https://github.com/user-attachments/assets/62f394ed-f964-4bd0-b8ab-6cf120603f3d" />

**问题编号-28** </br>
<img width="600" height="300" alt="LQA-026" src="https://github.com/user-attachments/assets/4faa8ec1-c4ea-497c-a876-51bdb3edc40a" />


**问题编号-30** </br>
<img width="600" height="300" alt="LQA-028" src="https://github.com/user-attachments/assets/f927be14-96e5-4c12-bb6d-9cc6b15f67f6" />

### 2.2 句子拼接

> *以下句子由于拼接出现句中错误大写、缺少空格等问题。*

**问题编号-22** </br>
<img width="600" height="300" alt="LQA-020" src="https://github.com/user-attachments/assets/78e6c8f9-b846-45f1-bbe8-54fa463d7517" />

**问题编号-26** </br>
<img width="600" height="300" alt="LQA-024" src="https://github.com/user-attachments/assets/7f092f1e-8129-4ef5-aaa9-d22fa08c9de8" />

## 3. 规范

1. 所有玩家可见文本，包括 UI、任务、道具名、技能描述、错误提示、按钮、字幕、成就、系统提示等，必须抽取到文本表；文本表是唯一文本来源，代码、UI 预制体、脚本中不得出现玩家可见自然语言字面量，只能引用字符串 ID；修改文案只改文本表，不改代码。不得把示例数值写死进源文，并备注占位符含义和示例值。

2. 字符串 ID 使用稳定、可读的格式：`模块_对象_字段_状态`，例如 `ATTR_EMOTION_GAINED`、`QUEST_001_OBJECTIVE_01`；不得使用 `text1`、`str_023` 或译文内容作为 ID。已发布 ID 不得改名或复用，新增文本必须新增 ID；同一句话在多处使用时必须引用同一个字符串 ID，不得复制多份文本。

3. 玩家可见句子必须整句放入文本表，使用完整句子模板和命名占位符；禁止用字符串相加、条件判断、前后缀拼接、截取、替换词序等方式生成或修改玩家可见自然语言；占位符顺序必须允许翻译人员调整，代码必须按占位符名称传参，不得假设占位符顺序。

4. 数量、名字、时间、类型等动态内容必须通过占位符传入；占位符中的名称若玩家可见，也必须用文本 ID 查当前语言，不得传硬编码中文；单复数、性别、选择必须使用 ICU MessageFormat 或本地化平台等价机制，不得在代码里写 `count == 1 ? "message" : "messages"`。

5. 拼接后只用于内部日志或协议字段的内容，不得直接显示给玩家；内部调试日志、协议字段、正则表达式、文件路径等非玩家文本不进入文本表、不翻译；非玩家文本即使包含中文，也不得为了“统一”而放进文本表并送翻。

6. 玩家自定义名称如需保存，必须单独字段保存原文，并明确不翻译，不得与系统物品名、任务名混用；玩家自定义名称可以作为占位符传入已翻译的句子模板，但名称本身不得作为文本表条目被翻译。

## 4. 正确与错误示范

### 4.1 文本表、字符串 ID 与动态数值

**示例：属性提升提示**

- 中文原文：情绪增加了3（其中 `3` 为动态数值，实际显示值由运行时传入）
- 错误写法一：把整句连同数值写死在代码里。

```csharp
emotionText.text = "情绪增加了3";
```

- 错误写法二：把固定文案和动态数值用字符串相加拼起来。

```csharp
emotionText.text = "情绪增加了" + count;
```

- 正确写法：

文本表：

| 字符串 ID | 源文 | 译文 | 上下文/备注 | 长度限制 | 截图/场景 |
|---|---|---|---|---|---|
| `ATTR_EMOTION_GAINED` | 情绪增加了{count} | Emotion increased by {count} | 属性提升提示，角色属性面板显示；`{count}` 为本次增加的情绪值，整数，示例值 3 | 源文 20 字符；译文 40 字符 | 角色属性面板 |

代码：

```csharp
emotionText.text = Localization.Get("ATTR_EMOTION_GAINED", count);
```

其中 `count` 为运行时传入的数值，例如 3。文本表保存完整句子模板，翻译人员可调整 `{count}` 在句中的位置；代码只按占位符名称传参，不拼接句子。

### 4.2 不拼接句子

**示例 2：飞鸽传书与单复数**

- 中文原文：你还有 5 封飞鸽传书
- 错误写法：

```csharp
msg = "You have " + count + " new " + (count == 1 ? "letter" : "letters");
```

- 正确写法：

中文文本表：

```text
MSG_NEW_LETTERS = 你还有{count}封飞鸽传书
```

代码：

```csharp
Localization.Get("MSG_NEW_LETTERS", count);
```

英文单复数的处理见第6章。

### 4.3 存档只存 ID、不存文字

**示例：背包物品**

- 中文原文：铁剑
- 错误写法：存档保存显示文字。

```json
{ "itemName": "铁剑", "count": 1 }
```

- 正确写法：存档保存物品 ID 和数量，显示时查文本表。

```json
{ "itemId": "ITEM_SWORD_IRON", "count": 1 }
```

显示时：

```csharp
itemName.text = Localization.Get("ITEM_SWORD_IRON_NAME");
```

或通过物品配置把 `ITEM_SWORD_IRON` 映射到名称文本 ID `ITEM_SWORD_IRON_NAME`，再查当前语言文本。这样切换到英文后，旧存档会显示 `Iron Sword`，不会继续显示“铁剑”。
