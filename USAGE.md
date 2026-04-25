# 跨平台部署指南

> 这份文档详细说明如何在不同的 AI 平台上使用这个 skill。每个平台都有针对性的最佳实践。

## 平台支持矩阵

| 平台 | 支持等级 | 集成方式 |
|---|---|---|
| Claude.ai (Web/Desktop) | ⭐⭐⭐⭐⭐ | 原生 Skill 上传 |
| Claude API | ⭐⭐⭐⭐⭐ | System Prompt + 文件引用 |
| Cursor | ⭐⭐⭐⭐ | `.cursorrules` 文件 |
| GitHub Copilot | ⭐⭐⭐ | Custom Instructions |
| Codex / OpenAI Agents | ⭐⭐⭐⭐ | System Prompt |
| ChatGPT (GPTs) | ⭐⭐⭐⭐ | Instructions + Knowledge |
| Cline (VS Code) | ⭐⭐⭐⭐ | Custom Instructions |
| Gemini | ⭐⭐⭐ | Gem Instructions |
| 国产模型(豆包、Kimi、文心一言等) | ⭐⭐⭐ | 智能体配置 |

---

## 1. Claude.ai (Web 或 Desktop)

### 方法 A:直接上传 Skill 文件夹(推荐)

1. 打开 claude.ai → Settings → Capabilities → Skills
2. 点击 "Upload Skill"
3. 选择本项目根目录(包含 `SKILL.md` 的那个目录)
4. 启用这个 skill

之后任何对话中,只要你提到投资判断、问股票、提到段永平等关键词,Claude 会自动调用这个 skill。

### 方法 B:打包成 .skill 文件

如果你希望分享给别人:

```bash
# 在项目根目录运行(假设 Anthropic 的 skill packager 已安装)
zip -r duan-yongping-skill.skill .
```

接收方就可以直接上传这个 `.skill` 文件。

---

## 2. Claude API / Claude Code

### 在 Claude Code(终端工具)中

1. 把整个 skill 文件夹放在你的工作目录里(例如 `~/.claude/skills/duan-yongping-skill/`)
2. Claude Code 会自动识别 skill 目录中的 SKILL.md
3. 在 prompt 中提到投资相关问题时自动触发

### 通过 Anthropic API 直接调用

```python
import anthropic

# 读取 SKILL.md 作为 system prompt
with open('duan-yongping-skill/SKILL.md', 'r', encoding='utf-8') as f:
    skill_content = f.read()

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=4096,
    system=skill_content,
    messages=[
        {"role": "user", "content": "用段永平方法论分析贵州茅台,当前股价 1500 元"}
    ]
)

print(response.content[0].text)
```

如需深度分析,可以把 references 文件夹里的 case 文档也作为 context 传入。

---

## 3. Cursor

### 配置 .cursorrules

在你的项目根目录(或全局 `~/.cursor/rules`)创建 `.cursorrules` 文件,把 `SKILL.md` 全文粘贴进去。

或者更简洁的做法——在 `.cursorrules` 里引用:

```
你是一个 AI 投资助手,严格按照 docs/duan-yongping-skill/SKILL.md 的方法论工作。

当用户问到任何投资判断、公司分析、股票估值的问题时,请按照该文档中定义的 7 步决策流程进行分析。

参考文档:
- docs/duan-yongping-skill/references/case-apple-2011.md - 苹果案例
- docs/duan-yongping-skill/references/case-moutai-2013.md - 茅台案例
- docs/duan-yongping-skill/references/case-pdd-special.md - PDD案例
- docs/duan-yongping-skill/references/duan-quotes.md - 经典语录

输出必须包含:7步分析、综合结论、诚实提醒。
```

### 配合使用 @ 引用

在 Cursor 对话中可以直接 `@duan-yongping-skill/SKILL.md` 把方法论拉入 context。

---

## 4. ChatGPT / OpenAI

### 创建 Custom GPT

1. 打开 ChatGPT → Explore GPTs → Create
2. 在 Configure 中:
   - **Name**: 段永平投资助手 (或你喜欢的名字)
   - **Description**: 用段永平的价值投资方法论帮你分析公司、判断标的
   - **Instructions**: 把 `SKILL.md` 全文粘贴
   - **Knowledge**: 上传 `references/` 和 `examples/` 下的所有 .md 文件
3. 关闭 Web Browsing(段永平方法论不依赖外部搜索)
4. 关闭 DALL-E(不需要画图)
5. 保留 Code Interpreter(可以做财务计算)

### 直接使用 ChatGPT(无 GPTs 订阅)

打开新对话,第一条消息发:

```
我希望你严格按照以下方法论分析投资问题。在我后续提出任何投资判断时,你必须按这个 7 步流程分析。

[粘贴 SKILL.md 全文]

请确认你已经理解。
```

ChatGPT 确认后,你就可以正常提问了。

---

## 5. Codex / OpenAI Agents SDK

```python
from openai import OpenAI

client = OpenAI()

with open('duan-yongping-skill/SKILL.md', 'r', encoding='utf-8') as f:
    skill_content = f.read()

response = client.chat.completions.create(
    model="gpt-4o",  # 或其他模型
    messages=[
        {"role": "system", "content": skill_content},
        {"role": "user", "content": "用段永平方法论分析一下英伟达"}
    ]
)

print(response.choices[0].message.content)
```

如果使用 OpenAI Agents,可以把 SKILL.md 和 references 作为 agent 的 instructions + knowledge files。

---

## 6. Gemini (Google)

### 在 Gemini 网页版

如果有 Gemini Advanced + Gems 功能:
1. 创建一个新 Gem
2. Instructions 粘贴 `SKILL.md`
3. (可选)上传 references 文件作为 knowledge

### Gemini API

```python
import google.generativeai as genai

with open('duan-yongping-skill/SKILL.md', 'r', encoding='utf-8') as f:
    skill_content = f.read()

model = genai.GenerativeModel(
    'gemini-2.5-pro',
    system_instruction=skill_content
)

response = model.generate_content("用段永平方法论分析腾讯")
print(response.text)
```

---

## 7. 国内模型(豆包、Kimi、文心一言、智谱 GLM 等)

### 通用方法:智能体/Agent 配置

绝大多数国内大模型平台都提供"智能体"或"Agent"配置功能:

1. 创建新智能体
2. 在"角色设定"或"系统提示词"中粘贴 `SKILL.md` 全文
3. 在"知识库"中(如果支持)上传 references 和 examples
4. 保存后即可使用

### 字节豆包 / Coze

1. 进入 Coze 平台
2. 创建 Bot
3. 角色设定填写 `SKILL.md` 中的核心内容
4. 上传知识库:references/ 和 examples/ 下的所有文件
5. 发布后即可对话

### Kimi(月之暗面)

Kimi 支持长上下文,可以直接:
1. 在新对话中先发送 `SKILL.md` 全文
2. 然后正常提问

或者通过 Kimi 的智能体功能配置系统提示。

---

## 8. Cline / Continue / Roo Cline (VS Code 插件)

### Cline

在 VS Code 设置中找到 Cline 的 Custom Instructions,粘贴 `SKILL.md`。

或者更优:把整个项目放在你的工作区,在 prompt 中说"参考 `duan-yongping-skill/SKILL.md` 的方法论分析..."。

### Continue

在 `~/.continue/config.json` 中添加 system message,引用 SKILL.md 的内容。

---

## 9. 通用最佳实践

### 系统提示注入位置

不论平台:
- **如果支持 system prompt**:把 `SKILL.md` 放在 system 位置,效果最好
- **如果只支持 user message**:把 `SKILL.md` 放在第一条 user message,然后等 AI 确认理解,再提问
- **如果支持知识库 / RAG**:references 和 examples 走知识库,SKILL.md 走 instructions

### Context 长度问题

`SKILL.md` 大约 5000 字,加上 references 总共可能超过 30000 字。如果模型 context window 较小:
- **优先级 1**:确保 `SKILL.md` 全文在 context 中
- **优先级 2**:references 按需引入(用户问到苹果时才引入苹果案例)
- **优先级 3**:examples 仅在用户要求"展示完整分析"时引入

### 触发关键词

为了让 AI 主动调用方法论,鼓励用户在提问时包含这些关键词:
- "段永平 / 大道"
- "护城河 / 本分 / 能力圈"
- "用价值投资方法分析"
- "这家公司值不值得买"
- "这是不是好生意"

### 输出风格调教

如果 AI 输出过于啰嗦或偏离方法论:
- 提醒它"严格按 SKILL.md 的 7 步流程输出"
- 要求它"不要省略任何一步"
- 要求它"在结论前明确指出卡在哪些步骤"

---

## 常见问题

### Q1: AI 会"扮演段永平"对我说话吗?

**不会,也不应该**。这个 skill 的设计是让 AI 用段永平的方法论分析问题,**而不是模仿段永平的语气说话**。冒充真实人物可能误导你的决策。

### Q2: AI 会推荐我买什么股票吗?

**不会**。这个 skill 严格遵守"分析而不决策"的边界。AI 会告诉你某只股票通过/不通过段永平的 7 步流程,但最终决策权在你。

### Q3: 如果我不同意 AI 的分析怎么办?

**很好,这正是设计意图**。这个 skill 的目标是强迫你回答正确的问题,不是给你一个权威答案。如果你不同意,**说出你不同意的具体步骤和理由**——这就是真正的独立思考。

### Q4: 这个 skill 适合所有市场吗?

**主要针对优质成熟公司的价值投资判断**,适用于 A 股、港股、美股的大型企业。**不适用于**:
- 早期初创公司(应该用风投思维)
- 技术快速迭代行业(超出段永平能力圈)
- 周期股(段永平方法论倾向拒绝)
- 短期交易 / 量化交易(完全不同的逻辑)

### Q5: 我可以修改这个 skill 吗?

**当然可以,License 是 MIT**。建议的修改方向:
- 加入更多案例(网易、腾讯、可口可乐等)
- 针对特定市场的本地化(比如 A 股的特殊情况)
- 加入你自己的方法论扩展(标记清楚是你的扩展,而非段永平原意)

如果修改了,欢迎提 PR 让其他人也能用上。
