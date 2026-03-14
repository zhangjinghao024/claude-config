# format-qa

对项目问答（projectIntroQa.json）中指定项目和行动的答案做排版优化，**内容一字不改，只做格式润色**。

## 触发场景

用户说"帮我优化问答格式"、"答案排版优化"、"帮我润色问答格式"等时触发。

## 常量（执行前确认）

```
QA_JSON_PATH = /Users/zhangjinghao/Desktop/project/js_oj_node_backEnd/data/projectIntroQa.json
TEMP_SCRIPT   = /Users/zhangjinghao/Desktop/project/format_qa_temp.js
```

## 执行步骤

### 第一步：确定范围

从用户的请求里提取：
- **项目 key**：在 `qaMap` 的顶层 key 中匹配（如 `qunar-train-ticket`、`train-ai-node`）
- **行动 index**：用户说"行动N"→ `actionIndex = N - 1`（行动1 = 0，行动2 = 1，以此类推）；未指定则处理全部

如果 key 不确定，先运行：
```bash
node -e "const d=JSON.parse(require('fs').readFileSync('<QA_JSON_PATH>'));console.log(Object.keys(d.qaMap));"
```

### 第二步：扫描现状

用下面这一条命令扫描目标范围内所有 QA，**全程复用此命令，只换 filter 条件**：

```bash
node -e "
const fs=require('fs');
const d=JSON.parse(fs.readFileSync('<QA_JSON_PATH>'));
const all=d.qaMap['<PROJECT_KEY>'];
const items=all.filter(qa=><ACTION_FILTER>);   // 全部时用: qa=>true
items.forEach((qa,i)=>{
  const a=qa.answer;
  const fences=(a.match(/\`\`\`/g)||[]).length;
  const h3=(a.match(/^###/gm)||[]).length;
  const hasTriangle=a.includes('▎');
  // padding 检测：只在非代码块区域查找长空格
  const nonCode=a.replace(/\`\`\`[\s\S]*?\`\`\`/g,'');
  const hasPad=!!nonCode.match(/ {10,}/);
  console.log('QA'+(i+1),'id:',qa.id,
    '| 围栏:',fences/2,'| ###:',h3,
    '| ▎:',hasTriangle,'| pad:',hasPad);
  console.log('Q:',qa.question);
  console.log('A(前200):',a.trim().slice(0,200).replace(/\n/g,'↵'));
  console.log();
});
"
```

> `<ACTION_FILTER>` 示例：`qa=>qa.actionIndex===1`

### 第三步：判断哪些答案需要优化

| 问题 | 判断标准 | 处理方式 |
|------|---------|---------|
| `▎` + padding 空格 | 含 `▎` 字符或非代码块区域有 10+ 连续空格 | 去掉 `▎`，剥离行首缩进，重排段落 |
| 代码没有围栏 | 含 `if (`、`const `、`function`、`//` 等，但无 ` ``` ` | 加 ` ```js ` 围栏 |
| 缺少区段标题 | 有 `---` 分隔但下方是裸文本 | 加 `### 标题` |
| ASCII 树/表格图 | 字符画结构（`├─`、`└─`、`┌─`） | 用 ` ``` ` 包裹 |
| 引用对话 | `"面试官问题？"` 形式 | 改为 `> "..."` 引用块 |

**跳过条件**：已有 `###` 和代码围栏、内容简短（1-2 句）、答案为空。

### 第四步：写临时脚本并执行

> 答案内容较长时，**必须写临时文件**，不要用内联 `node -e`（反引号嵌套和转义极易出错）。

```js
// format_qa_temp.js
const fs = require('fs');
const path = '<QA_JSON_PATH>';
const data = JSON.parse(fs.readFileSync(path, 'utf8'));

const answers = {
  '<qa-id-1>': `格式化后的内容`,
  '<qa-id-2>': `格式化后的内容`,
  // 只写需要修改的 id，跳过的不列
};

data.qaMap['<PROJECT_KEY>'].forEach(qa => {
  if (answers[qa.id]) qa.answer = answers[qa.id];
});
data.updatedAt = new Date().toISOString();
fs.writeFileSync(path, JSON.stringify(data, null, 2), 'utf8');
console.log('✅ 写入', Object.keys(answers).length, '条');
```

```bash
node <TEMP_SCRIPT> && rm <TEMP_SCRIPT>
```

### 第五步：验证（复用第二步命令）

直接复用第二步的扫描命令，确认：
- `▎: false`，`pad: false`（非代码块区域无 padding）
- 有代码内容的 QA，`围栏 > 0`
- 有多段落的 QA，`### > 0`

---

## 格式化规则速查

| 规则 | 操作 |
|------|------|
| 区段标题 | `---` 后的标题行 → `### 标题` |
| 代码片段 | JS/TS 加 ` ```js `，命令行加 ` ```bash `，纯文本/ASCII 图加 ` ``` ` |
| 行内代码 | 函数名、变量名、文件路径 → 反引号包裹 |
| 列表标签 | `- 第一，xxx：` → `- **第一，xxx：**` |
| `▎` 段落 | 去掉 `▎` 前缀，保留后面的正文，重新对齐段落 |

## 注意事项

- **内容零修改**：不增删任何文字，只改格式符号
- **代码块内缩进不算 padding**：只有非代码块区域的长空格才需要处理
- **不强加结构**：短答案不需要加标题，保持原样
- **临时文件用完即删**：执行完 `rm <TEMP_SCRIPT>`
