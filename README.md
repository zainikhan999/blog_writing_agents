# 🧠 AutoGen Blogpost Generator

This project uses [AutoGen](https://github.com/microsoft/autogen) to create an intelligent multi-agent workflow for generating and refining blog posts. The system includes:

- ✍️ **Writer Agent** — writes concise blog posts  
- 🧐 **Critic Agent** — gives feedback to improve quality  
- 🔍 **Nested Reviewers**:
  - SEO Reviewer
  - Legal Reviewer
  - Ethics Reviewer
  - Meta Reviewer — aggregates all feedback and sends to writer

---

## 📦 Installation

```bash
pip install autogen
```

---

## 🔐 API Key Setup

### On Google Colab:

```python
from google.colab import userdata
deep_seek = userdata.get('DEEP_SEEK_API')
```

### On Local Machine:

```python
deep_seek = "your_openrouter_api_key_here"
```

---

## ⚙️ LLM Configuration

```python
llm_config = {
    "config_list": [
        {
            "model": "deepseek/deepseek-r1",
            "api_key": deep_seek,
            "base_url": "https://openrouter.ai/api/v1",
            "price": [0, 0],
        }
    ],
    "temperature": 0.7,
    "max_tokens": 1000
}
```

---

## 🤖 Agent Initialization

### ✍️ Writer Agent

```python
writer = autogen.AssistantAgent(
    name="writer",
    system_message="You are a writer. You write engaging and concise blogposts (with title) on given topics. You must polish your writing based on the feedback you receive and give a refined version. Only return your final work without additional explanation.",
    llm_config=llm_config
)
```

### 🧐 Critic Agent

```python
critic = autogen.AssistantAgent(
    name="critic",
    system_message="You are a writing critic. You offer brief, constructive feedback on blog posts to improve clarity and quality.",
    llm_config=llm_config
)
```

---

## 🔍 Nested Reviewer Agents

### 📈 SEO Reviewer

```python
seo = autogen.AssistantAgent(
    name="seo",
    system_message="You are an SEO specialist. You review blogposts and suggest optimizations to improve search engine visibility.",
    llm_config=llm_config
)
```

### ⚖️ Legal Reviewer

```python
legal = autogen.AssistantAgent(
    name="legal",
    system_message="You are a legal expert. You review blogposts to ensure they do not contain legal issues or liabilities.",
    llm_config=llm_config
)
```

### 🧭 Ethics Reviewer

```python
ethics = autogen.AssistantAgent(
    name="ethics",
    system_message="You are an ethics reviewer. You review blogposts for any ethical concerns, bias, or offensive content.",
    llm_config=llm_config
)
```

---

## 🧠 Meta Reviewer

```python
meta_reviewer = autogen.GroupChat(
    agents=[seo, legal, ethics],
    messages=[],
    max_round=1
)

meta_reviewer_manager = autogen.GroupChatManager(
    groupchat=meta_reviewer,
    llm_config=llm_config,
    system_message="You gather feedback from SEO, Legal, and Ethics reviewers. Summarize the suggestions clearly for the writer."
)
```

---

## 🔄 Group Chat: Writer and Critic

```python
group = autogen.GroupChat(
    agents=[writer, critic],
    messages=[],
    max_round=3
)

manager = autogen.GroupChatManager(
    groupchat=group,
    llm_config=llm_config
)
```

---

## 🚀 Start Blog Generation

```python
manager.initiate_chat(
    sender=critic,
    message="Write a blog post on 'How AI is Transforming Healthcare'."
)
```

---

## ✅ Final Integration with Nested Feedback

Once the writer finishes and the critic gives feedback, pass the result through `meta_reviewer_manager` to ensure full SEO, legal, and ethical compliance.

```python
meta_reviewer_manager.initiate_chat(
    sender=seo,
    message="Please review the latest version of the blogpost."
)
```

---

## 📄 Output

You will receive:

- Finalized, refined blog post from the writer
- Detailed review summary from SEO, legal, and ethics reviewers
- All within a fully autonomous agent chat loop

---

## 🔧 Notes

- You can swap `deepseek/deepseek-r1` with any other OpenRouter-compatible LLM like `openai/gpt-4`, `google/gemini-pro`, etc.
- Adjust temperature and max_tokens to control creativity and verbosity.
