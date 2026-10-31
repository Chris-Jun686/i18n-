# 第5章：《文本管理与存档》

## 5.1 关于文本管理与存档

游戏里给玩家看的每一句话、每个按钮名和道具名，都不要直接写在程序里，而是集中记在一张清单上，程序只记一个固定编号，显示时按编号去清单里取当前语言的文字。一句话要整句记，数字和名字留空由程序填，不能把句子拆开拼。存档时也只记物品、任务等的编号和数量，不记玩家看到的文字，这样换语言后旧存档仍能显示正确语言。

## 5.2 规则

### 5.2.1 文本外置

- **R5-01** 所有玩家可见文本都必须放进文本表，包括界面、任务、道具名、技能描述、按钮、错误提示、字幕、成就和系统提示。
- **R5-02** 代码、UI 预制体和脚本里不得出现玩家可见的文字，只能引用字符串 ID。
- **R5-03** 文本表是唯一的文本来源，修改文案只改文本表，不改代码。
- **R5-04** UI 预制体里的默认占位文字也要替换成字符串 ID，否则容易漏翻。前作 #13 英文界面出现中文，就是这类问题。

### 5.2.2 字符串 ID 命名

- **R5-05** ID 格式为“模块_对象_字段_状态”，全部用大写英文和下划线，例如 `ATTR_EMOTION_GAINED`、`QUEST_001_OBJECTIVE_01`。
- **R5-06** 不得用无意义的编号（如 `text1`、`str_023`）或译文内容作为 ID。
- **R5-07** 已发布的 ID 不得改名或复用，新增文本必须新建 ID。
- **R5-08** 含义和使用场景都相同的文本共用一个 ID；中文相同但场景不同的，要分开建 ID，并在上下文栏写明区别。

### 5.2.3 不拼接句子

- **R5-09** 玩家可见的句子必须整句放进文本表。
- **R5-10** 不得用字符串相加、前后缀拼接、截取、条件判断或替换词序等方式生成玩家可见的文字。

### 5.2.4 变量与占位符

- **R5-11** 数量、名字、时间等动态内容用命名占位符传入，如 `{count}`、`{daughter_name}`，不用 `{0}` 这类编号。
- **R5-12** 代码按占位符名称传参，不假设占位符的顺序，让译者可以自由调整占位符在句中的位置。
- **R5-13** 如果传入的是玩家可见的名称（如物品名），先按当前语言查文本表，不能传入写死的中文。
- **R5-14** 在上下文栏写明每个占位符的含义和示例值；源文里不写死示例数值。
- **R5-15** 单复数、性别等语法变化不在代码里判断，具体做法见第 6 章。

### 5.2.5 不需要翻译的文本

- **R5-16** 调试日志、协议字段、正则表达式、文件路径等玩家看不到的文字，不放进文本表，也不翻译。  
  例：代码里的报错日志“存档加载失败:path=xxx”只给程序员看，留在代码里即可。如果放进文本表，译者会白白花时间去翻；协议字段或文件路径被翻译了，还会导致程序出错。
- **R5-17** 上述内部文字可以拼接，但拼接结果不得显示给玩家。
- **R5-18** 玩家自己输入的内容（如给女儿起的名字）单独保存原文，标记为不翻译，不与系统物品名、任务名混在一起。
- **R5-19** 玩家输入的内容可以作为占位符传入已翻译的句子，但本身不进文本表。例：玩家给女儿起名“小月”，切换成英文后界面仍然显示“小月”，只是被放进已翻译的句子里：`{daughter_name} gained 3 Charisma` 显示为“小月 gained 3 Charisma”。

### 5.2.6 存档

- **R5-20** 存档只保存物品、任务等的 ID 和数量，不保存显示文字。
- **R5-21** 读取存档时，按当前语言查文本表显示名称，这样玩家切换语言后，旧存档也能正确显示。

## 5.3 前作发生了什么

### 5.3.1 文本表与统一表述

**问题编号-#23** </br>
<img width="600" height="300" alt="LQA-023" src="https://github.com/user-attachments/assets/01733f2d-9d1a-48dd-9c2d-0e5efee83fe8" />


> 属性提升用的是 “increased by 5%” 这种句式，和#28、#30 的表述都不同。三处含义相同，却有三种写法，说明各自单独写在了不同地方</br>
> 对应规则 R5-03、R5-08

**问题编号-#28** </br>
<img width="600" height="300" alt="LQA-026" src="https://github.com/user-attachments/assets/4faa8ec1-c4ea-497c-a876-51bdb3edc40a" />

> “Affection 3 increased” 把数值夹在属性名和动词中间，英文读起来不自然。</br>
> 对应规则 R5-08、R5-12

**问题编号-#30** </br>
<img width="600" height="300" alt="LQA-028" src="https://github.com/user-attachments/assets/f927be14-96e5-4c12-bb6d-9cc6b15f67f6" />

> “Charisma increased2” 的数值直接贴在单词后面，缺少空格，说明由拼接导致。既属于表述不统一，也属于拼接问题。</br>
> 对应规则 R5-08、R5-10

### 5.3.2 句子拼接

**问题编号-#22** </br>
<img width="600" height="300" alt="LQA-020" src="https://github.com/user-attachments/assets/78e6c8f9-b846-45f1-bbe8-54fa463d7517" />

> “chance to Obtain double dishes” 句中出现大写，说明前后两段文字分开存放后拼在了一起。</br>
> 对应规则 R5-09、R5-10

**问题编号-#26** </br>
<img width="600" height="300" alt="LQA-024" src="https://github.com/user-attachments/assets/7f092f1e-8129-4ef5-aaa9-d22fa08c9de8" />

> “purchasing itemsfrom the store” 两个单词之间缺少空格，同样是拼接导致。</br>
> 对应规则 R5-09、R5-10

## 5.4 正确与错误示范

### 5.4.1 文本表、字符串 ID 与动态数值

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

> 其中Localization.Get为演示用伪代码； `count` 为运行时传入的数值，例如 3。文本表保存完整句子模板，翻译人员可调整 `{count}` 在句中的位置；代码只按占位符名称传参，不拼接句子。

### 5.4.2 不拼接句子

**示例：飞鸽传书**

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

> 英文单复数的处理见第6章。

### 5.4.3 存档只存 ID，不存文字

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

> **使用Unity引擎时需注意**

**用 String Table 管所有可见文本**

* 在 `Window → Asset Management → Localization Tables` 中创建 `String Table Collection`，每个 key 就是本章的字符串 ID。</br>

* 代码和预制体里只引用 key，不写文字；UI 上的 `Text`/`TextMeshProUGUI` 组件可绑定 `LocalizedString`，由它按当前 Locale 从 String Table 取值。</br>

**用 Smart Strings 处理占位符与语法变化**

* Smart Strings 能够一句话保留完整模板，支持命名占位符，替代 `{0}`、`{1}` 这类数字索引，对译者更友好。同时程序按名字填内容，按数量、性别等自动选对说法，避免把句子拆开硬拼。

**存档只存 ID，显示时查 String Table**

* 存档里保存 `"itemId": "ITEM_SWORD_IRON"`，读档时用 `Localization.Get("ITEM_SWORD_IRON_NAME")` 或通过物品配置映射到名称 key，再按当前语言取文本。

**不要在存档中保存任何 `LocalizedString` 的当前取值**

* 存档一旦存了文字，切语言后旧档就可能出现混杂语言。

**玩家输入单独保存，不进 String Table**

* 玩家给女儿起的名字存为原始字符串，不放入任何 String Table，也不标记翻译；需要嵌入系统句子时，把它作为 Smart String 的 `{daughter_name}` 参数传入即可。</br>

* **内部调试日志、协议字段、文件路径**同样不进 String Table——String Table 的每一行都是给译者的，放进去只会浪费翻译资源。

## 5.5 检查清单

☐ 在代码和 UI 预制体里搜索，找不到玩家可见的中文。</br>

☐ 每个字符串 ID 都符合命名格式，且没有重复。</br>

☐ 没有用加号或拼接生成玩家可见的句子。</br>

☐ 每个占位符都有名字，并在上下文栏写明含义和示例值。</br>

☐ 存档文件里只有 ID 和数值，没有显示文字。</br>

☐ 玩家起的名字单独保存，并标记为不翻译。

## 参考文献

* IGDA本地化兴趣小组《游戏本地化最佳实践》*[Best Practices for Game Localization](https://igda.org/resources-archive/best-practices-for-game-localization/)*
* Unity引擎本地化文档 **[Unity文档](https://docs.unity3d.com/Packages/com.unity.localization@1.4/manual/LocalizationSettings.html)**
* 微软软件本地化博客 **[微软博客](https://xtm.ai/blog/software-localization)**
