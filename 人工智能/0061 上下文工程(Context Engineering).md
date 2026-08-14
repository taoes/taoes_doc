**上下文工程（Context Engineering）**，是继提示词工程之后，更进阶、更系统化的一门学问。如果说提示词工程是教你怎么“说话”，那上下文工程就是教你怎么为AI构建一个“完整的工作环境”——包括历史记忆、实时信息、人格设定、工具权限等。

它的核心原理可以概括为一句话：**在有限的上下文窗口内，塞入最有助于当前任务完成的信息，并让信息以最优结构呈现。**

---

## 🧠 上下文工程的核心理念

传统观点认为，上下文就是“对话历史”。但在现代Agent系统中，上下文是一个**多维度的动态信息空间**：

```
┌─────────────────────────────────────────────────────────┐
│                    上下文空间                             │
├─────────────────────────────────────────────────────────┤
│  1. 系统指令 (System Prompt)                             │
│     - 角色设定、行为准则、输出格式                    │
│  2. 对话历史 (Conversation History)                   │
│     - 多轮对话、用户情绪、未完成的意图                │
│  3. 任务状态 (Task State)                            │
│     - 当前进行到哪一步、已完成子任务                  │
│  4. 外部知识 (External Knowledge)                    │
│     - RAG检索的文档、数据库查询结果                  │
│  5. 工具上下文 (Tool Context)                        │
│     - API Schema、可用工具列表、调用权限              │
│  6. 用户画像 (User Profile)                          │
│     - 偏好、历史行为、身份信息                       │
│  7. 环境信息 (Environment)                          │
│     - 时间、地点、设备、当前场景                     │
└─────────────────────────────────────────────────────────┘
```

---

## 🔬 上下文工程的原理

### 1. **注意力机制的局限性**

Transformer架构的核心是注意力机制，但它有一个关键弱点：**它对所有token的注意力权重是均匀分配的**。当上下文超过一定长度（如32K、128K tokens），模型对中间部分的"记忆力"会显著下降。

这就是著名的 **"Lost-in-the-Middle"（中间丢失）现象**：

```
高注意力 ████████████████████  ← 开头 (高权重)
低注意力 ████                  ← 中间 (易被忽略)
高注意力 ████████████████████  ← 结尾 (高权重)
```

**启示**：关键信息要放在开头或结尾，不要埋在中间。

---

### 2. **信息密度与压缩**

大模型的上下文窗口有限（即便128K也不够用），我们需要对信息进行压缩。

```python
# 信息压缩对比

# ❌ 未压缩（占用大量token）
full_document = """
客户张先生，男，35岁，北京人，从事IT行业，于2023年11月15日...
（5000字的客户档案）
"""

# ✅ 压缩后（保留核心信息）
compressed_context = """
用户画像：{age:35, location:北京, industry:IT}
关键事件：2023-11-15 首次购买，订单号#ORD123
"""
```

---

### 3. **渐进式上下文加载**

不是一次性把所有信息都塞进去，而是**按需加载**：

```python
class ContextManager:
    def __init__(self):
        self.base_context = "系统角色设定..."      # 始终加载
        self.session_context = []                  # 动态加载
    
    def build_context(self, user_input, step):
        context = self.base_context
        
        # 按需加载
        if step == "意图识别":
            context += self.load_intent_knowledge()
        elif step == "工具调用":
            context += self.load_tool_schemas()
        
        # 只加载最近N轮对话
        context += self.get_recent_history(5)
        
        # 加载用户画像（如果有必要）
        if self.need_user_profile():
            context += self.load_user_profile()
        
        return context
```

---

## 🎯 上下文工程的最佳实践

### 实践1：分层上下文架构

```
第一层：系统级上下文（固定）
├── 角色定义："你是一个智能客服助手"
├── 行为准则："不得泄露用户隐私"
└── 输出规范："始终以JSON格式返回"

第二层：会话级上下文（动态）
├── 当前意图：[BookFlight, QueryWeather]
├── 已完成步骤：已查询到航班信息
├── 待处理任务：需要用户确认价格
└── 对话状态：waiting_for_confirmation

第三层：用户级上下文（按需加载）
├── 用户ID：U12345
├── 历史订单：最近3笔
├── 偏好设置：偏好国航、靠窗座位
└── 信用等级：黄金会员

第四层：知识级上下文（RAG检索）
├── 航班政策文档（片段）
├── 天气数据（实时API结果）
└── 竞品价格对比（按需检索）
```

---

### 实践2：动态Token预算管理

```python
class TokenBudgetManager:
    def __init__(self, max_tokens=128000):
        self.budget = max_tokens
        self.allocations = {}
    
    def allocate(self, component, priority, min_tokens, max_tokens):
        """根据优先级分配token"""
        self.allocations[component] = {
            'priority': priority,  # 1-10
            'min': min_tokens,
            'max': max_tokens,
            'allocated': min_tokens
        }
    
    def optimize(self, total_available):
        """在总预算内优化分配"""
        # 高优先级组件获得更多token
        sorted_components = sorted(
            self.allocations.items(),
            key=lambda x: x[1]['priority'],
            reverse=True
        )
        
        remaining = total_available
        for comp, config in sorted_components:
            allocated = min(config['max'], remaining)
            config['allocated'] = allocated
            remaining -= allocated
            if remaining <= 0:
                break
```

---

### 实践3：上下文摘要与压缩

**关键问题**：多轮对话后，上下文越来越长，但大部分信息已无用。

**解决方案**：使用摘要SubAgent定期压缩历史。

```python
class ContextSummarizer:
    def __init__(self, llm):
        self.llm = llm
    
    def summarize_history(self, history, max_length=1000):
        """将长历史压缩为摘要"""
        if len(history) < 10:  # 少于10轮不压缩
            return history
        
        prompt = f"""
        请将以下对话历史压缩为简洁摘要，保留：
        1. 用户的核心需求和意图
        2. 已完成的步骤
        3. 尚未解决的问题
        4. 重要实体（订单号、日期等）
        
        对话历史：
        {history}
        
        压缩后摘要：
        """
        return self.llm.generate(prompt)
    
    def sliding_window(self, history, window_size=5):
        """滑动窗口：保留最近N轮 + 历史摘要"""
        return {
            'summary': self.summarize_history(history[:-window_size]),
            'recent': history[-window_size:]
        }
```

---

### 实践4：结构化上下文注入

将上下文以结构化格式（如JSON、XML）注入，比自然语言更高效。

```python
# 使用XML标签构建上下文
context = f"""
    <system>
    你是智能客服助手，始终礼貌、专业。
    </system>
    
    <user_profile>
    {json.dumps(user_profile)}
    </user_profile>
    
    <conversation_history>
    {json.dumps(recent_history)}
    </conversation_history>
    
    <task_state>
    {json.dumps(task_state)}
    </task_state>
    
    <available_tools>
    {json.dumps(tool_schemas)}
    </available_tools>
    
    <current_input>
    {user_input}
    </current_input>
"""
```

---

### 实践5：小样本示例的动态选择（RAG for Few-shot）

不是把所有示例都塞进去，而是根据当前问题**检索最相关的示例**。

```python
class ExampleRetriever:
    def __init__(self, vector_db):
        self.vector_db = vector_db
    
    def retrieve_relevant_examples(self, query, top_k=3):
        """根据当前问题检索最相关的示例"""
        query_vector = self.embed(query)
        results = self.vector_db.search(query_vector, top_k)
        return [r['example'] for r in results]
    
    def build_few_shot_context(self, user_input):
        examples = self.retrieve_relevant_examples(user_input)
        context = "以下是相关示例：\n"
        for i, ex in enumerate(examples):
            context += f"示例{i+1}：{ex['input']} → {ex['output']}\n"
        return context
```

---

### 实践6：上下文过期策略

不是所有信息都永久有效，需要设定**时效性**。

```python
class ContextExpiry:
    def __init__(self):
        self.context_items = []
    
    def add_item(self, key, value, ttl_seconds=3600):
        self.context_items.append({
            'key': key,
            'value': value,
            'expires_at': time.time() + ttl_seconds
        })
    
    def get_valid_context(self):
        now = time.time()
        return [
            item['value'] 
            for item in self.context_items 
            if item['expires_at'] > now
        ]
```

---

## 🚀 高级技巧：多模态上下文

现代Agent越来越多地处理文本以外的信息：

```python
class MultimodalContext:
    def __init__(self):
        self.text_context = ""
        self.image_context = []  # 图片URL或base64
        self.audio_context = []  # 语音转录文本
        self.video_context = []  # 关键帧描述
    
    def build_prompt(self):
        """
        构建包含多模态信息的提示词
        图片和视频内容通过描述文本注入
        """
        context = self.text_context
        
        if self.image_context:
            context += "\n\n【用户发送的图片描述】\n"
            for img in self.image_context:
                context += f"- {img['description']}\n"
        
        return context
```

---

## 📊 上下文工程的效果对比

| 上下文管理方式          | 准确率 | 延迟 | Token消耗 |
|:----------------------- |:------ |:---- |:--------- |
| **全量历史**            | 85%    | 3.2s | 高        |
| **最近N轮（固定窗口）** | 78%    | 1.8s | 中        |
| **摘要+最近窗口**       | 82%    | 2.1s | 中        |
| **分层+动态加载**       | 89%    | 2.4s | 低        |
| **分层+RAG检索**        | 92%    | 2.8s | 低        |

---

## ⚠️ 常见陷阱与解决方案

| 陷阱                 | 表现                   | 解决方案                  |
|:-------------------- |:---------------------- |:------------------------- |
| **信息过载**         | AI无法聚焦关键信息     | 分层结构+优先级排序       |
| **关键信息埋在中间** | AI忽略重要指令         | 重复关键信息（开头+结尾） |
| **Token浪费**        | 塞入无用历史           | 定期摘要+滑动窗口         |
| **上下文断裂**       | 指代不明（"它"是谁？） | 显式指代消解+实体追踪     |
| **时效性丢失**       | 用了过时的信息         | 设置TTL+实时校验          |

---

## 🛠️ 落地工具箱

### 1. **上下文管理器（Python示例）**

```python
class ContextManager:
    def __init__(self, llm, max_tokens=128000):
        self.llm = llm
        self.max_tokens = max_tokens
        self.system_prompt = ""
        self.history = []
        self.state = {}
        self.user_profile = {}
        self.knowledge_base = {}
    
    def set_system_prompt(self, prompt):
        self.system_prompt = prompt
    
    def add_user_message(self, message):
        self.history.append(('user', message))
        # 自动触发上下文优化
        self.optimize_context()
    
    def add_assistant_message(self, message):
        self.history.append(('assistant', message))
    
    def set_user_profile(self, profile):
        self.user_profile = profile
    
    def add_knowledge(self, key, value):
        self.knowledge_base[key] = value
    
    def build(self):
        """构建最终上下文"""
        context = self.system_prompt + "\n\n"
        
        # 添加用户画像（如果有）
        if self.user_profile:
            context += f"用户信息：{json.dumps(self.user_profile)}\n\n"
        
        # 添加当前状态
        if self.state:
            context += f"当前状态：{json.dumps(self.state)}\n\n"
        
        # 添加对话历史（优化后的）
        context += self._format_history()
        
        # 添加相关知识
        if self.knowledge_base:
            context += f"相关信息：{json.dumps(self.knowledge_base)}\n\n"
        
        # Token预算检查
        self._check_budget(context)
        
        return context
    
    def _format_history(self):
        """格式化历史，保持最近轮次细节"""
        if len(self.history) <= 10:
            return "\n".join([f"{role}: {msg}" for role, msg in self.history])
        else:
            # 10轮以上：压缩早期历史
            summary = self._summarize_early_history()
            recent = self.history[-5:]
            return f"历史摘要：{summary}\n\n最近对话：\n" + \
                   "\n".join([f"{role}: {msg}" for role, msg in recent])
```

---

### 2. **在意图识别中的应用**

```python
class IntentContextBuilder:
    """针对意图识别的上下文构建器"""
    
    def build(self, user_input, conversation_history, user_profile):
        return {
            "system": "你是意图识别专家，根据上下文准确识别用户意图",
            
            "context_hint": "如果用户使用了指代词'它'、'那个'，请结合对话历史确定指向",
            
            "history_summary": self._summarize_entity_tracking(conversation_history),
            # 只提取与实体相关的历史，而非完整对话
            
            "user_profile": {
                "常用意图": user_profile.get('frequent_intents', []),
                "历史实体": user_profile.get('historical_entities', {})
            },
            
            "current_input": user_input,
            
            "intent_list": ["BookFlight", "QueryWeather", "CancelOrder", "QueryOrder"],
            
            "output_format": '{"intent": "...", "entities": {...}}'
        }
```

---

## 💡 总结与核心原则

1. **开头结尾效应**：关键信息放开头和结尾，而非中间
2. **先压缩，再加载**：定期摘要，滑动窗口
3. **按需检索**：用RAG动态加载相关信息，而不是全量塞入
4. **结构化优于自然语言**：用JSON/XML提供明确结构
5. **分层管理**：系统级固定、会话级动态、用户级按需
6. **时效性管理**：过期信息及时清理
7. **Token预算意识**：始终监控和控制token消耗
