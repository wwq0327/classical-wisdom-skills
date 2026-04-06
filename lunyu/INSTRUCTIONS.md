# 论语 JSON 生成规则

## 源文件
`/Users/walt/codes/lunyu/论语.txt` — 包含《论语》原文，共 499 条

## 输出文件
`/Users/walt/codes/lunyu/.claude/skills/lunyu/resources/论语.json`

## JSON 格式（每条）

```json
{
  "id": "lunyu_001",
  "chapter": "学而第一",
  "original": "子曰："学而时习之，不亦说乎？有朋自远方来，不亦乐乎？人不知，而不愠，不亦君子乎？"",
  "keywords": ["学习", "朋友", "君子"],
  "interpretation": {
    "problem_type": "学习/人际/心态",
    "applicable_scenario": "学完知识后及时复习巩固；远方的朋友特意来访；不被理解时保持平和心态",
    "example": "职场新人参加培训后主动练习并分享给同事",
    "action": "建立复习习惯；主动维护友情；对误解保持豁达",
    "misconception": "误以为学习只是为了考试；忽视朋辈互助的价值"
  }
}
```

## 关键规则

### 1. 引号使用
- **内层对话用中文弯引号**：`""`（U+201C 和 U+201D）
- **JSON 字符串边界用 ASCII 双引号**：`"`
- **示例**：`"子曰："学而时习之，不亦说乎？""`
  - 外层 ASCII `"` 是 JSON 字符串边界
  - 内层 `""` 是中文对话引号

### 2. 五要素（interpretation）
每条必须包含完整五要素，且**必须根据本条原文的具体含义**生成，不得使用通用模板：

| 字段 | 说明 | 示例 |
|------|------|------|
| problem_type | 问题类型 | 学习/人际/心态 |
| applicable_scenario | 适用场景 | **必须具体到本条讨论的核心主题**，不能泛泛而谈 |
| example | 具体例子 | **必须与本条含义直接相关**，不是通用社交例子 |
| action | 行动建议 | **必须体现本条传达的具体道理**，不是"真诚待人"等套话 |
| misconception | 常见误区 | **必须指出本条容易产生的误解**，不是泛化的人生哲理 |

**禁止规则**：
- ❌ 禁止使用通用模板（如"日常生活中待人接物的基本原则"、"与人交往时保持真诚和尊重"）
- ❌ 禁止 applicable_scenario/example/action/misconception 在不同条目中重复相同内容
- ❌ 禁止为所有条目使用相同的五要素模板
- ❌ 每个条目的解读必须反映该条《论语》的具体含义

**正确做法**：先理解本条《论语》在说什么，再生成对应的场景、例子、行动建议。

### 3. 总条目
- **必须恰好 499 条**
- **ID 连续**：lunyu_001 到 lunyu_499
- 按论语.txt 原文顺序生成

### 4. keywords
- 每条 **3 个不重复的关键词**
- **禁止使用占位符或通用标签**如"为人处世"作为关键词
- 关键词必须与本条内容直接相关
- 可用标签类型：学习、朋友、君子、诚信、孝道、人际、心态、工作、家庭、礼、仁、义、君子与小人、修养、治国、为人处世、智慧、勇气、忠诚、悌道、天命、礼仪、政治、教育、反省、友情、诚信、恭敬、谦虚、谨慎、中庸、刚柔、性情、学习态度、为人、为学、礼仪等

**关键词生成规则**：
- 先分析本条原文的核心主题
- 提取 3 个最相关的关键词
- 关键词之间应互不重复
- 如果原文讨论"君子vs小人"，关键词可以是["君子", "小人", "义"]
- 如果原文讨论"学习"，关键词可以是["学习", "复习", "智慧"]

### 5. 调性要求
- ✅ 有趣、幽默、积极
- ❌ 恶搞、搞笑、恶趣味

### 6. 质量红线（禁止）
以下情况视为不合格，必须重新生成：
- 三个关键词完全相同（如 ["为人处世", "为人处世", "为人处世"]）
- 三个关键词中有重复
- interpretation 五要素与本条原文含义无关
- 多条目使用相同的 applicable_scenario / example / action / misconception

### 7. 生成流程
1. 读取 `/Users/walt/codes/lunyu/论语.txt` 原文
2. **逐条理解**本条《论语》的含义（不要批量套模板）
3. 根据本条含义生成：
   - 3 个与内容相关的关键词（不重复）
   - 针对本条含义的 interpretation（五要素各不同）
4. 输出 JSON

## 验证命令

```bash
# 1. 验证 JSON 格式正确
python3 -m json.tool /Users/walt/codes/lunyu/.claude/skills/lunyu/resources/论语.json > /dev/null && echo "JSON valid"

# 2. 验证条目数量
python3 -c "import json; data=json.load(open('/Users/walt/codes/lunyu/.claude/skills/lunyu/resources/论语.json')); print(f'条目数: {len(data)}')"

# 3. 验证 ID 连续
python3 -c "import json; data=json.load(open('/Users/walt/codes/lunyu/.claude/skills/lunyu/resources/论语.json')); ids=[int(d['id'].split('_')[1]) for d in data]; print('ID连续' if ids==list(range(1,len(data)+1)) else 'ID不连续')"

# 4. 验证关键词无重复
python3 -c "
import json
data = json.load(open('/Users/walt/codes/lunyu/.claude/skills/lunyu/resources/论语.json'))
dupes = []
for e in data:
    kws = e['keywords']
    if len(kws) != len(set(kws)):
        dupes.append(e['id'])
if dupes:
    print(f'关键词重复: {dupes}')
else:
    print('关键词无重复')
"

# 5. 验证关键词数量
python3 -c "
import json
data = json.load(open('/Users/walt/codes/lunyu/.claude/skills/lunyu/resources/论语.json'))
bad = [e['id'] for e in data if len(e['keywords']) != 3]
if bad:
    print(f'关键词数量异常: {bad}')
else:
    print('关键词数量正确(3个)')
"
```

## 执行步骤

1. 读取 `/Users/walt/codes/lunyu/论语.txt` 原文
2. **逐条理解**每条《论语》的含义（不是批量套模板）
3. 为每条生成：
   - 3 个与本条含义相关的关键词（不重复）
   - 针对本条含义的 interpretation（五要素）
4. 合并为单个 JSON 文件
5. 运行验证命令确认正确
