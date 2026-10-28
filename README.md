## 1. 文本

**文本统一放进文本表，不要硬编码** </br>

硬编码指将玩家能看到的文字直接写在程序代码、UI 预制体、脚本里，例如：

```csharp
showMessage("你获得了铁剑");

itemName.text = "铁剑";
```

这样的问题在于每改一种语言，都要改代码；翻译无法统一管理；同一句话在多处重复；容易漏翻；无法做长度检查、上下文说明、术语统一；代码和翻译版本容易不一致。

“统一放进文本表”：**所有玩家可见文本都抽取到一个独立的文本资源表里，代码只引用字符串 ID。** 文本表可以是 CSV、Excel、JSON、数据库或本地化平台，通常至少包含：

| 字段 | 示例 |
|---|---|
| 字符串 ID | `ITEM_SWORD_IRON_NAME` |
| 源文 | 铁剑 |
| 译文 | Iron Sword |
| 上下文/备注 | 物品名称，背包列表显示 |
| 长度限制 | 20 字符 |
| 截图/场景 | 背包界面 |

代码里只写 ID：

```csharp
showMessage(Localization.Get("ITEM_SWORD_IRON_NAME"));

itemName.text = Localization.Get("ITEM_SWORD_IRON_NAME");
```

这样切换语言时，代码不变，只换文本表。文本表成为**唯一文本来源**。
> 注意：玩家可见文本、UI、任务、道具名、技能描述、错误提示、按钮、字幕、成就等都应进文本表；内部调试日志、协议字段、正则表达式、文件路径等通常不属于玩家文本。

字符串 ID 命名建议稳定、可读、不复用、不包含译文内容，例如：

```text
模块_对象_字段_状态
ITEM_SWORD_IRON_NAME
UI_BUTTON_CONFIRM
QUEST_001_OBJECTIVE_01
ERROR_NETWORK_TIMEOUT
```

不要用 `text1`、`str_023`，也不要把中文或英文原文当 ID。ID 一旦发布尽量不要改，避免存档、配置、埋点引用失效。

## 2. 句子

**不要拼接句子** </br>
“拼接句子”指代码把一句话拆成多个片段，再用字符串相加、格式化或条件判断拼成玩家可见句子。例如：

```csharp
msg = "你获得了" + count + "个" + itemName;
```

或者：

```csharp
msg = "You have " + count + " new " + (count == 1 ? "message" : "messages");
```

这种做法对本地化很危险，因为不同语言的**语序、语法、复数、性别、格变化、量词、空格、标点**都不同。中文可以“你获得了3个铁剑”，英文可能是 `You got 3 Iron Swords`，日语、俄语、阿拉伯语语序和复数规则又不同。如果把“你获得了”“个”“铁剑”拆开翻译，翻译人员无法看到完整句子，译员无法调整语序，运行时拼出来就会语法错误或很生硬。

更隐蔽的拼接还包括：

```csharp
"New " + itemType + " available"
"Player " + name + " has joined"
"剩余" + time + "秒"
```

这些都应改成**完整句子模板 + 占位符**，让翻译人员能调整整个句子的语序。例如：

```text
MSG_ITEM_GAINED = 你获得了{count}个{itemName}
```

英文文本表：

```text
MSG_ITEM_GAINED = You got {count} {itemName}.
```

代码只传参数：

```csharp
Localization.Get("MSG_ITEM_GAINED", count, itemName);
```

如果涉及单复数、性别、选择，最好使用 ICU MessageFormat 这类本地化格式，而不是在代码里写 </br>
`if (count == 1)`：

```text
{count, plural,
  =0 {You have no new messages.}
  one {You have # new message.}
  other {You have # new messages.}
}
```

中文可能不需要复数，但目标语言需要。代码不应假设“单数加 s”这种规则。

> 禁止用字符串相加、条件拼接、前后缀拼接来生成玩家可见自然语言。动态内容必须使用完整句子模板和命名占位符；占位符顺序由翻译人员可调整；复数、性别、格变化使用本地化消息格式处理。代码不得对译文做 `+ "s"`、截取、替换词序等语言相关操作。

## 3. 存档

**存档只存 ID、不存文字。** 例如背包存档应存：

```json
{ "itemId": "ITEM_SWORD_IRON", "count": 1 }
```

不要存：

```json
{ "itemName": "铁剑", "count": 1 }
```

因为玩家切换到英文后，显示时应通过 `ITEM_SWORD_IRON` 查当前语言文本表得到 `Iron Sword`。如果存档里存了“铁剑”，切换语言后旧存档仍显示“铁剑”，就会出错。存档存 ID，文本表存文字，显示时动态查表，才能保证语言切换、文本修订和术语更新一致。
