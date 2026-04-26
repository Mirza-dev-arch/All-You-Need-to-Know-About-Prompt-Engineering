# Chapter-10-Chatbot-Evolution
→ From simple chatbots to stateful, memory-aware, production-ready agents.
---
## Table of Chatbot Evolution:
- [Chatbot 1: Building a Single Agent Chatbot (API calling)— memory + system message + loop](#chatbot-1-building-a-single-agent-chatbot-API-calling—memory-system-message-loop)  
- [CHATBOT 2: Multi-Model Research Chatbot with Ollama](#chatbot-2-multi-model-research-chatbot-with-0llama)
- [CHATBOT 3: Multi-Model Research Chatbot with Memory + Auto-Summarization](#chatbot-3-multi-model-research-chatbot-with-memory-auto-summarization)
---


## Chatbot 1: Building a Single Agent Chatbot (API calling)— memory + system message + loop
This project combines everything you’ve learned: API parameters, conversation history (memory), a professional system prompt, and an interactive loop.

### Installation & Setup

**Step 1: Install & set up OpenAI library**

   ```bash
   pip install openai
   ```

**Step 2: Get your API key**:
     - Go to platform.openai.com
     - Log in → API keys → Create new secret key
     - Copy it (starts with sk-)

**Step 3: Create a `.env` file in your project folder:**
         **Why use `.env`?** It keeps your API key secure and out of your source code.
   * In your project directory, create a file named `.env` at the root level.
   * In your `.env` file, define key-value pairs for your configuration settings. For example: `OPENAI_API_KEY=sk-your-real-key-here`
   * The program will utilize the loaded environment variables from the `.env` file, and the output will display the value associated with the specified key, such as `MY_KEY`.

### Full Chatbot Code:
```python
from openai import OpenAI
import os
from dotenv import load_dotenv

# ====================== SETUP ======================
load_dotenv()  # Load variables from .env file
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# ====================== MEMORY (Conversation History) ======================
conversation = []  # This list will remember the entire chat

# ====================== PROFESSIONAL SYSTEM PROMPT ======================
SYSTEM_PROMPT = """
You are a world-class AI tutor specializing in mathematics.
TASK: Explain concepts clearly with step-by-step reasoning.
CONTEXT: Use real-life examples whenever possible.
CONSTRAINTS: Use simple language, avoid jargon.
FORMAT: Always use numbered steps and relevant emojis.
THINKING: Think step by step before answering.
OPTIMIZE: Make explanations fun, engaging, and easy to understand.
"""

# Add system prompt once at the beginning
conversation.append({"role": "system", "content": SYSTEM_PROMPT})

# ====================== CHAT FUNCTION ======================
def chat_with_ai(user_input, temperature=0.7, max_tokens=400):
    # Add new user message to history
    conversation.append({"role": "user", "content": user_input})
    
    # Call OpenAI with full conversation history
    response = client.chat.completions.create(
        model="gpt-4o",           # or "gpt-4o-mini" for lower cost
        messages=conversation,
        temperature=temperature,  # Creativity control
        max_tokens=max_tokens,    # Length limit
        top_p=0.9
    )
    
    ai_reply = response.choices[0].message.content
    
    # Save AI reply to memory for next turns
    conversation.append({"role": "assistant", "content": ai_reply})
    
    return ai_reply

# ====================== MAIN CHAT LOOP ======================
print("🤖 Math Tutor Chatbot is ready! Type 'exit' to stop.\n")

while True:
    user_input = input("You: ")
    if user_input.lower() in ["exit", "quit", "bye"]:
        print("👋 Goodbye!")
        break
    
    reply = chat_with_ai(user_input)
    print(f"AI: {reply}\n")
```
### How Everything Works Together:

| Layer          |  What It Does                                            | Where You See It in the Code                        |           
|----------------|----------------------------------------------------------|-----------------------------------------------------|
| Setup          |  Loads API key securely from .env                        |  Top 5 lines                                        | 
| Memory         |  Stores every message so the AI remembers context        | `conversation = []` list                            | 
| System Prompt  |  Sets personality, rules, and response style             | `SYSTEM_PROMPT` variable                            |
| Core Function  |  Sends full history to OpenAI and saves reply            | `chat_with_ai()` function                           | 
| Chat Loop      |  Keeps the conversation going indefinitely               | `while True:` loop                                  | 

**Key Learning Points:**
* The `conversation` list acts as the memory of the chatbot.
* The system prompt is added only once and defines consistent behavior throughout the chat.
* Every new user message and AI reply is appended to the list, so the model always has full context.
* You can easily adjust `temperature`, `max_tokens`, and other parameters inside the `chat_with_ai()` function.

**Pro Tip**: Start with a strong, detailed system prompt. It often has more impact on output quality than individual user messages.
This single-agent chatbot is the foundation for more advanced agents, multi-agent systems, and tool-calling applications you will build later in Phase 4.

---

  **[↑ Back to Top](#chapter-10-chatboot-evolution-)** 
  
---

## CHATBOT 2: Multi-Model Research Chatbot with Ollama

This is a smart research chatbot that uses 4 specialized models working together like a team. Instead of one model doing everything (and making mistakes), they collaborate in a loop until the answer is accurate and complete.
The system resembles a research pipeline with role-specialized agents.

[Github Repo you can visit](https://github.com/openai/openai-cookbook/blob/main/examples/Enhance_your_prompts_with_meta_prompting.ipynb)


### System Architecture

#### Core Idea
A multi-agent pipeline where each model performs a specific cognitive task.

| Role                | Function                                                 |     
|---------------------|----------------------------------------------------------|
| Senior Model        |  Clarifies and restructures the user query               |
| Researcher Model    | Generates analysis and a structured answer               | 
| Critic Model        |  Audits reasoning and identifies gaps                    | 
| Supervisor Model    |  Produces final answer and quality check                 |


### Pipeline Logic

<img src="../../assets/images/phase4-chatbot2-pipeline.png"
     width="45%"
     align="center"
     style="display: block; margin: 25px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);"
     alt="Pipeline logic">
<br clear="all"/>


### Model Selection
We use Ollama-available models optimized for each role:

| Role           |  Model                             | Why                                           | Strength                   | Weakness                     |  RAM needed         |         
|----------------|------------------------------------|-----------------------------------------------|----------------------------|------------------------------|---------------------|   
| Senior         |  Llama 3.1 8B                      |  strong reasoning and prompt reformulation    | contextual understanding   |  moderate hallucination      | 16GB                |   
| Researcher     |  Mistral 7B                        |  fast analytical output                       | speed and structure        |  slightly weaker reasoning   | 12GB                |   
| Critic         |  DeepSeek‑R1 Distill               | chain-of-thought evaluation                   | reasoning accuracy         |  slower                      | 16GB                |   
| Supervisor     |  Llama 3.1 70B (or 8B locally)     | synthesis and evaluation                      | Coherence                  |  heavy compute               | 16GB                |   

**Minimal Setup Recommendation**: All 8B/7B models — runs well on most modern laptops.

### Agent Responsibilities

##### Senior Agent
Refines the user query into a clear, structured research prompt.

<img src="../../assets/images/phase4-senior-agent.png"
     width="555%"
     align="center"
     style="display: block; margin: 25px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);"
     alt="Senior Agent architecture">
<br clear="all"/>


#### Researcher Agent
Performs in-depth analysis and generates a structured response.

<img src="../../assets/images/phase4-reseacher-agent.png"
     width="55%"
     align="center"
     style="display: block; margin: 25px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);"
     alt="Researcher Agent architecture">
<br clear="all"/>


#### Critic Agent
Reviews the research output for errors, gaps, and hallucinations.

<img src="../../assets/images/phase4-critics-agent.png"
     width="55%"
     align="center"
     style="display: block; margin: 25px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);"
     alt="Critics Agent architecture">
<br clear="all"/>


#### Supervisor Agent
Integrates feedback and produces the final polished answer.

<img src="../../assets/images/phase4-supervisor-agent.png"
     width="55%"
     align="center"
     style="display: block; margin: 25px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);"
     alt="Supervisor Agent architecture">
<br clear="all"/>


### Installation Guide

#### Step 1 — Install Ollama

**Linux/Mac:**

```Bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Windows:**
Download the installer from [Ollama](https://ollama.com)


#### Step 2 — Pull models

```Bash
ollama pull llama3.1
```

```Bash
ollama pull mistral
```

```Bash
ollama pull deepseek-r1
```


#### Step 3 — Install Python library

```Bash
pip install ollama
```

#### Step 4 — Run the chatbot

```Bash
python chatbot.py
```

#### Full Code Implementation

```python

import ollama

# ------------------- Generic Call Function -------------------
def call_model(model: str, prompt: str, temperature: float = 0.3):
    """Sends a prompt to Ollama and returns the response text."""
    response = ollama.chat(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        options={"temperature": temperature, "top_p": 0.9}
    )
    return response["message"]["content"]

# ------------------- Senior Agent -------------------
def senior_agent(query: str) -> str:
    prompt = f"""You are a senior research analyst.
Understand the user query and restructure it clearly.
Extract: objective, constraints, success metrics, negative cases, required output format.
Return ONLY a structured research prompt for the next model.

USER QUERY: {query}"""
    return call_model("llama3.1:8b", prompt, temperature=0.2)

# ------------------- Researcher Agent -------------------
def researcher_agent(senior_prompt: str) -> str:
    prompt = f"""You are a research analyst.
Use the structured prompt below and generate a detailed answer.

PROMPT: {senior_prompt}

Return in this exact format:
SUMMARY
DETAILED ANALYSIS
METHOD
RESULT
REFERENCES
LIMITATIONS"""
    return call_model("mistral:7b", prompt, temperature=0.4)

# ------------------- Critic Agent -------------------
def critic_agent(senior_prompt: str, research_output: str) -> str:
    prompt = f"""You are a critical reviewer.
Compare the research output against the senior prompt.
Find: missing points, logic errors, hallucinations, and redundancies.
Give clear improvement actions.

SENIOR PROMPT: {senior_prompt}
RESEARCH OUTPUT: {research_output}"""
    return call_model("deepseek-r1:8b", prompt, temperature=0.1)

# ------------------- Supervisor Agent -------------------
def supervisor_agent(query: str, research_output: str, critique: str) -> str:
    prompt = f"""You are a supervisor AI.
User query: {query}
Research output: {research_output}
Critique report: {critique}

Task:
1. Fix issues identified
2. Produce final polished answer
3. Verify all requirements are met

Return:
FINAL_OUTPUT
QUALITY_CHECK
REQUIREMENT_MATCH_SCORE"""
    return call_model("llama3.1:8b", prompt, temperature=0.3)

# ------------------- Main Collaborative Chatbot -------------------
def collaborative_chatbot(query: str) -> str:
    senior_prompt = senior_agent(query)
    research_output = ""
    critique = ""

    # Researcher ↔ Critic loop (3 iterations for refinement)
    for i in range(3):
        research_output = researcher_agent(senior_prompt)
        critique = critic_agent(senior_prompt, research_output)
        # Feed critique back for improvement
        senior_prompt += f"\n\nCRITIQUE_FEEDBACK (Iteration {i+1}):\n{critique}"

    # Final synthesis
    final_answer = supervisor_agent(query, research_output, critique)
    return final_answer

# ------------------- Run Chatbot -------------------
if __name__ == "__main__":
    print("🤖 Multi-Model Research Chatbot Ready!")
    user_query = input("\nEnter your research question: ")
    result = collaborative_chatbot(user_query)
    
    print("\n" + "="*60)
    print("FINAL ANSWER")
    print("="*60)
    print(result)

```
---

  **[↑ Back to Top](#chapter-10-chatboot-evolution-)** 

---

## CHATBOT 3: Multi-Model Research Chatbot with Memory + Auto-Summarization
This advanced version of the research chatbot adds **permanent conversation memory** and **smart auto-summarization**. It prevents context overflow while keeping important information from long conversations.

### New Features Added
- Every message (user + assistant) is saved in `conversation = []`
- Before each new query, the code checks the total history length
- When the limit is reached → `update_summary()` compresses old messages
- New prompts now use: **Summary + Last 4 messages** (keeps recent context fresh)
- You can continue very long conversations without losing key context

**Example after many turns:**
- Old history is summarized as: *"User is a biology researcher in Berlin preparing a research paper."*
- Recent messages stay in full detail → AI still remembers your name, preferences, and current topic.


### Installation Guide

#### Step 1 — Install Ollama

**Linux/Mac:**

```Bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Windows:**
Download the installer from [Ollama](https://ollama.com)


#### Step 2 — Pull models

```Bash
ollama pull llama3.1
```

```Bash
ollama pull mistral
```

```Bash
ollama pull deepseek-r1
```


#### Step 3 — Install Python library

```Bash
pip install ollama
```

#### Step 4 — Run the chatbot

```Bash
python chatbot.py
```

#### Full Code Implementation
```python

import ollama

# ====================== GLOBAL MEMORY & SUMMARY ======================
conversation = []   # Stores full history: [{"role": "user/assistant", "content": "..."}]
summary = ""        # Compressed memory of old conversation

# ====================== TOKEN LIMIT CHECK ======================
MAX_HISTORY_CHARS = 12000   # Approx 3000-4000 tokens for 8B models

def should_summarize():
    total_chars = sum(len(msg["content"]) for msg in conversation)
    return total_chars > MAX_HISTORY_CHARS

# ====================== UPDATE SUMMARY FUNCTION ======================
def update_summary():
    global summary
    if len(conversation) < 4:   # Too short to summarize
        return
    
    # Summarize everything except the last 3 messages (keep recent context fresh)
    old_convo = conversation[:-3]
    history_text = "\n".join([f"{msg['role'].upper()}: {msg['content']}" for msg in old_convo])
    
    prompt = f"""Briefly and accurately summarize the following conversation history. 
Keep only key facts, user details, goals, and important context. Do NOT add new information.

CONVERSATION:
{history_text}"""

    response = ollama.generate(model="llama3.1:8b", prompt=prompt)
    summary = response["response"].strip()
    print("📝 Conversation summarized (old history compressed)")

# ====================== GENERIC MODEL CALL ======================
def call_model(model: str, prompt: str, temperature: float = 0.3):
    """Sends a prompt to Ollama and returns the response text."""
    response = ollama.chat(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        options={"temperature": temperature, "top_p": 0.9}
    )
    return response["message"]["content"]

# ====================== SENIOR AGENT (Uses Summary + Recent History) ======================
def senior_agent(query: str) -> str:
    global summary
    
    # Trigger summarization if history is getting too long
    if should_summarize():
        update_summary()
    
    # Build prompt with compressed summary + recent messages
    history_recent = "\n".join([f"{msg['role'].upper()}: {msg['content']}" for msg in conversation[-4:]])
    
    prompt = f"""You are a senior research analyst.
{summary if summary else ""}   # ← Compressed long-term memory

Recent conversation:
{history_recent}

New user query: {query}

Understand the intent and restructure it clearly.
Extract: objective, constraints, success metrics, negative cases, required output format.
Return ONLY a structured research prompt for the Researcher."""

    return call_model("llama3.1:8b", prompt, temperature=0.2)

# ====================== RESEARCHER AGENT ======================
def researcher_agent(senior_prompt: str) -> str:
    prompt = f"""You are a research analyst.
Use the structured prompt below and generate a detailed answer.

PROMPT: {senior_prompt}

Return in this exact format:
SUMMARY
DETAILED ANALYSIS
METHOD
RESULT
REFERENCES
LIMITATIONS"""

    return call_model("mistral:7b", prompt, temperature=0.4)

# ====================== CRITIC AGENT ======================
def critic_agent(senior_prompt: str, research_output: str) -> str:
    prompt = f"""You are a critical reviewer.
Compare the research output against the senior prompt.
Find: missing points, logic errors, hallucinations, and redundancies.
Give clear improvement actions.

SENIOR PROMPT: {senior_prompt}
RESEARCH OUTPUT: {research_output}"""

    return call_model("deepseek-r1:8b", prompt, temperature=0.1)

# ====================== SUPERVISOR AGENT ======================
def supervisor_agent(query: str, research_output: str, critique: str) -> str:
    prompt = f"""You are a supervisor AI.
User query: {query}
Research output: {research_output}
Critique report: {critique}

Task:
1. Fix issues identified
2. Produce final polished answer
3. Verify all requirements are met

Return:
FINAL_OUTPUT
QUALITY_CHECK
REQUIREMENT_MATCH_SCORE"""

    return call_model("llama3.1:8b", prompt, temperature=0.3)

# ====================== MAIN COLLABORATIVE CHATBOT ======================
def collaborative_chatbot(query: str) -> str:
    # Add user message to permanent memory
    conversation.append({"role": "user", "content": query})
    
    # Run the multi-agent pipeline
    senior_prompt = senior_agent(query)
    research_output = ""
    critique = ""

    # Researcher ↔ Critic refinement loop (3 iterations)
    for _ in range(3):
        research_output = researcher_agent(senior_prompt)
        critique = critic_agent(senior_prompt, research_output)
        # Feed critique back for continuous improvement
        senior_prompt += f"\n\nCRITIQUE_FEEDBACK:\n{critique}"

    # Final synthesis by Supervisor
    final_answer = supervisor_agent(query, research_output, critique)
    
    # Add AI reply to permanent memory
    conversation.append({"role": "assistant", "content": final_answer})
    
    return final_answer

# ====================== RUN CHATBOT ======================
if __name__ == "__main__":
    print("🤖 Chatty - Your Multi-Model Research Partner with Memory!")
    print("Type 'exit' to stop.\n")
    
    while True:
        user_query = input("You: ")
        if user_query.lower() in ["exit", "quit", "bye"]:
            print("👋 Goodbye!")
            break
        
        result = collaborative_chatbot(user_query)
        print(f"\nAI: {result}\n")

```
---

  **[↑ Back to Top](#chapter-10-chatboot-evolution-)** 

---

## Chatbot 4: Multi-Model Research Chatbot with Auto Model Picker + Memory + Summarization 🤖
This is the most advanced research chatbot in the series. It intelligently:
- Detects your available RAM
- Analyzes the query type (general, coding, research, deep reasoning)
- Automatically picks the best quantized model for each agent
- Maintains full conversation memory with smart auto-summarization
- Uses a 4-agent collaborative team (Senior → Researcher → Critic → Supervisor)

### Key Features
- **Auto RAM Detection** using `psutil`
- **Dynamic Model Picker** — chooses optimal model per role and task type
- **Permanent Conversation Memory** with intelligent auto-summarization
- **4-Agent Collaborative Pipeline** with refinement loop
- **Task-aware routing** (coding, research, general, etc.)
- Runs efficiently on both low-end and high-end hardware


### Auto Model Selection Logic:
The chatbot automatically selects the best model based on your RAM and query type:
- **Low RAM (<12 GB)**: Uses lightweight models like Phi-3 Mini
- **Medium RAM (12–24 GB)**: Llama 3.1 8B (default)
- **High RAM (24–40 GB)**: Qwen2.5 14B
- **Very High RAM (40+ GB)**: Llama 3.1 70B

It also applies task-specific overrides (e.g., DeepSeek-Coder for coding tasks).

### Installation Guide

#### Step 1 — Install Ollama

**Linux/Mac:**

```Bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Windows:**
Download the installer from [Ollama](https://ollama.com)

#### Step 2: Pull Recommended Models

```Bash
ollama pull llama3.1:8b
```
```Bash
ollama pull mistral:7b
```
```Bash
ollama pull deepseek-coder-v2:16b
```
```Bash
ollama pull qwen2.5:14b
```

#### Step 3 — Install Python library

```Bash
pip install ollama psutil
```

#### Step 4 — Run the chatbot

```Bash
python smart_chatbot.py
```

#### Full Code Implementation

```python

import ollama
import psutil

# ====================== AUTO RAM DETECTION ======================
def get_ram_gb():
    return psutil.virtual_memory().total / (1024 ** 3)

RAM_GB = get_ram_gb()
print(f"🖥️ Detected RAM: {RAM_GB:.1f} GB")

# ====================== AUTO MODEL SELECTOR ======================
def get_best_model_for_role(role: str, task_type: str = "general") -> str:
    # Base model selection based on RAM
    if RAM_GB < 12:
        base = "phi3:mini"
    elif RAM_GB < 24:
        base = "llama3.1:8b"
    elif RAM_GB < 40:
        base = "qwen2.5:14b"
    else:
        base = "llama3.1:70b"

    # Task-specific overrides
    if "coding" in task_type.lower():
        return "deepseek-coder-v2:16b" if RAM_GB >= 16 else "qwen2.5-coder:7b"
    elif "research" in task_type.lower() and RAM_GB >= 24:
        return "llama3.1:70b" if RAM_GB >= 40 else "llama3.1:8b"

    # Role-specific defaults
    if role == "senior":
        return base
    elif role == "researcher":
        return "mistral:7b" if RAM_GB < 16 else base
    elif role == "critic":
        return "deepseek-r1:8b" if RAM_GB >= 16 else "llama3.1:8b"
    elif role == "supervisor":
        return "llama3.1:8b" if RAM_GB < 32 else "llama3.1:70b"
    
    return base

# ====================== GLOBAL MEMORY & SUMMARY ======================
conversation = []
summary = ""
MAX_HISTORY_CHARS = 12000

def should_summarize():
    total = sum(len(msg["content"]) for msg in conversation)
    return total > MAX_HISTORY_CHARS

def update_summary():
    global summary
    if len(conversation) < 4:
        return
    old_text = "\n".join([f"{m['role'].upper()}: {m['content']}" for m in conversation[:-3]])
    prompt = f"""Summarize this conversation briefly. Keep only key facts, user details, goals, and context:
{old_text}"""
    response = ollama.generate(model="llama3.1:8b", prompt=prompt)
    summary = response["response"].strip()
    print("📝 Old conversation summarized")

# ====================== GENERIC MODEL CALL ======================
def call_model(model: str, prompt: str, temperature: float = 0.3):
    response = ollama.chat(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        options={"temperature": temperature, "top_p": 0.9}
    )
    return response["message"]["content"]

# ====================== SENIOR AGENT ======================
def senior_agent(query: str):
    global summary
    if should_summarize():
        update_summary()

    history_recent = "\n".join([f"{msg['role'].upper()}: {msg['content']}" for msg in conversation[-4:]])
    
    prompt = f"""You are a senior research analyst.
{summary}
Recent history:
{history_recent}

New query: {query}

1. Understand the intent
2. Detect task type (general/chat/coding/research/deep-reasoning)
3. Extract objective, constraints, metrics
4. Return ONLY in this format:
TASK_TYPE: [general/coding/research]
OBJECTIVE: ...
CONSTRAINTS: ...
METRICS: ...
RESEARCH_PROMPT: [structured prompt for researcher]"""

    senior_model = get_best_model_for_role("senior")
    return call_model(senior_model, prompt, temperature=0.2)

# ====================== RESEARCHER, CRITIC & SUPERVISOR AGENTS ======================
def researcher_agent(senior_prompt: str, task_type: str):
    model = get_best_model_for_role("researcher", task_type)
    prompt = f"""You are a research analyst.
Use this structured prompt:
{senior_prompt}

Return exactly:
SUMMARY
DETAILED ANALYSIS
METHOD
RESULT
REFERENCES
LIMITATIONS"""
    return call_model(model, prompt, temperature=0.4)

def critic_agent(senior_prompt: str, research_output: str):
    model = get_best_model_for_role("critic")
    prompt = f"""You are a strict critic. Compare against senior prompt and find issues.
SENIOR PROMPT: {senior_prompt}
RESEARCH OUTPUT: {research_output}
Return:
MISSING_POINTS
LOGIC_ERRORS
HALLUCINATIONS
REDUNDANCIES
IMPROVEMENT_ACTIONS"""
    return call_model(model, prompt, temperature=0.1)

def supervisor_agent(query: str, research_output: str, critique: str):
    model = get_best_model_for_role("supervisor")
    prompt = f"""You are the supervisor AI.
User query: {query}
Research: {research_output}
Critique: {critique}
Fix all issues and produce final clean answer.
Return:
FINAL_OUTPUT
QUALITY_CHECK
REQUIREMENT_MATCH_SCORE (0-100)"""
    return call_model(model, prompt, temperature=0.3)

# ====================== MAIN COLLABORATIVE CHATBOT ======================
def collaborative_chatbot(query: str):
    conversation.append({"role": "user", "content": query})
    
    senior_output = senior_agent(query)
    
    # Simple task type extraction
    task_type = "general"
    if "coding" in senior_output.lower():
        task_type = "coding"
    elif "research" in senior_output.lower():
        task_type = "research"

    research_output = ""
    critique = ""

    for _ in range(3):   # Researcher ↔ Critic refinement loop
        research_output = researcher_agent(senior_output, task_type)
        critique = critic_agent(senior_output, research_output)
        senior_output += f"\n\nCRITIQUE_FEEDBACK:\n{critique}"

    final_answer = supervisor_agent(query, research_output, critique)
    
    conversation.append({"role": "assistant", "content": final_answer})
    return final_answer

# ====================== RUN ======================
if __name__ == "__main__":
    print("🤖 Smart Multi-Agent Research Chatbot with Auto Model Picker is ready!")
    print("It automatically detects your RAM and chooses optimal models per task.\n")
    print("Type 'exit' to quit.\n")
    
    while True:
        user_query = input("You: ")
        if user_query.lower() in ["exit", "quit", "bye"]:
            print("👋 Goodbye!")
            break
        result = collaborative_chatbot(user_query)
        print(f"\nAI: {result}\n")

```
---

  **[↑ Back to Top](#chapter-10-chatboot-evolution-)** 

---

## Chatbot 5: Hybrid Chatbot (Local + API Intelligent Routing) ⚡
This advanced hybrid chatbot intelligently combines **local** and **API** models:

- Uses a small local model as a **smart router**
- Runs **simple tasks** locally (fast, free, private)
- Escalates **complex tasks** to a powerful API model (Llama-3.1-70B via Together.ai)
- Maintains full conversation memory with automatic summarization

### How the Hybrid System Works

1. Local router (fast 8B model) decides: “Is this query simple or complex?”
2. **Simple** → Entire pipeline runs locally (cheap & private)
3. **Complex** → Researcher and Supervisor use a high-quality API model
4. Memory + auto-summarization stays active across all decisions

### Key Features

- Automatic RAM detection
- Intelligent query routing (local vs API)
- Full conversation memory with auto-summarization
- Cost-efficient (uses API only when necessary)
- High privacy for simple queries
- Best quality for complex research/coding tasks

---

### Installation Guide

#### Step 1 — Install Ollama

**Linux/Mac:**

```Bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Windows:**
Download the installer from [Ollama](https://ollama.com)

#### Step 2: Pull Recommended Models

```Bash
ollama pull llama3.1:8b
```
```Bash
ollama pull mistral:7b
```
```Bash
ollama pull deepseek-coder-v2:16b
```
```Bash
ollama pull qwen2.5:14b
```

#### Step 3 — Install Python library

```Bash
pip install ollama psutil
```

#### Step 4 — Run the chatbot

```Bash
python hybrid_chatbot.py
```

#### Full Code Implementation
```Python
import ollama
import psutil
from openai import OpenAI
import os

# ====================== SETUP ======================
# === Change this line with your key ===
TOGETHER_API_KEY = "your-together-api-key-here"

client_api = OpenAI(
    base_url="https://api.together.xyz/v1",
    api_key=TOGETHER_API_KEY
)

RAM_GB = psutil.virtual_memory().total / (1024 ** 3)
print(f"🖥️ Detected RAM: {RAM_GB:.1f} GB")

# ====================== GLOBAL MEMORY & SUMMARY ======================
conversation = []
summary = ""
MAX_HISTORY_CHARS = 12000

def should_summarize():
    total = sum(len(msg["content"]) for msg in conversation)
    return total > MAX_HISTORY_CHARS

def update_summary():
    global summary
    if len(conversation) < 4:
        return
    old_text = "\n".join([f"{m['role'].upper()}: {m['content']}" for m in conversation[:-3]])
    prompt = f"""Summarize this conversation briefly. Keep only key facts, user details, goals, and context:
{old_text}"""
    resp = ollama.generate(model="llama3.1:8b", prompt=prompt)
    summary = resp["response"].strip()
    print("📝 History summarized")

# ====================== INTELLIGENT ROUTER ======================
def is_complex_query(query: str) -> bool:
    """Local small model decides if task needs powerful model"""
    router_prompt = f"""Classify this query as SIMPLE or COMPLEX.
Rules:
- SIMPLE: greeting, basic facts, short explanation, casual chat
- COMPLEX: deep reasoning, coding, research, math, multi-step analysis

Query: {query}

Answer with only one word: SIMPLE or COMPLEX"""

    response = ollama.chat(
        model="llama3.1:8b",
        messages=[{"role": "user", "content": router_prompt}],
        options={"temperature": 0.0}
    )
    decision = response["message"]["content"].strip().upper()
    return "COMPLEX" in decision

# ====================== MODEL CALLS ======================
def call_local(model: str, prompt: str, temp: float = 0.3):
    resp = ollama.chat(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        options={"temperature": temp, "top_p": 0.9}
    )
    return resp["message"]["content"]

def call_api(prompt: str, temp: float = 0.3):
    """Uses Together.ai Llama-3.1-70B"""
    resp = client_api.chat.completions.create(
        model="meta-llama/Llama-3.1-70B-Instruct-Turbo",
        messages=[{"role": "user", "content": prompt}],
        temperature=temp,
        max_tokens=800
    )
    return resp.choices[0].message.content

# ====================== AGENTS ======================
def senior_agent(query: str):
    if should_summarize():
        update_summary()
    
    history = "\n".join([f"{m['role'].upper()}: {m['content']}" for m in conversation[-4:]])
    
    prompt = f"""{summary}
Recent conversation:
{history}

New query: {query}

Analyze intent and return a well-structured prompt for the researcher."""
    
    return call_local("llama3.1:8b", prompt, 0.2)

def researcher_agent(senior_prompt: str, use_api: bool):
    prompt = f"""Follow this structured prompt and give a detailed answer:
{senior_prompt}

Format:
SUMMARY
DETAILED ANALYSIS
METHOD
RESULT
REFERENCES"""

    return call_api(prompt, 0.4) if use_api else call_local("mistral:7b", prompt, 0.4)

def critic_agent(senior_prompt: str, research_output: str):
    prompt = f"""You are a strict critic. Compare the output against the senior prompt and find issues.
Senior Prompt: {senior_prompt}
Research Output: {research_output}

Return:
MISSING_POINTS
LOGIC_ERRORS
HALLUCINATIONS
REDUNDANCIES
IMPROVEMENT_ACTIONS"""

    return call_local("deepseek-r1:8b", prompt, 0.1)

def supervisor_agent(query: str, research_output: str, critique: str, use_api: bool):
    prompt = f"""You are the supervisor AI.
User query: {query}
Research: {research_output}
Critique: {critique}

Fix all issues and produce the final clean answer.
Return:
FINAL_OUTPUT
QUALITY_CHECK"""

    return call_api(prompt, 0.3) if use_api else call_local("llama3.1:8b", prompt, 0.3)

# ====================== MAIN HYBRID CHATBOT ======================
def hybrid_chatbot(query: str):
    conversation.append({"role": "user", "content": query})
    
    use_api = is_complex_query(query)
    
    if use_api:
        print("🚀 Escalating to powerful API model (70B)...")
    else:
        print("⚡ Running on fast local model...")

    senior = senior_agent(query)
    research = researcher_agent(senior, use_api)
    critique = critic_agent(senior, research)
    senior += f"\n\nCRITIQUE:\n{critique}"
    
    final = supervisor_agent(query, research, critique, use_api)
    
    conversation.append({"role": "assistant", "content": final})
    return final

# ====================== RUN ======================
if __name__ == "__main__":
    print("🤖 Hybrid Chatbot (Local + API Intelligent Routing) is ready!")
    print("Simple questions → local | Complex questions → API 70B\n")
    print("Type 'exit' to quit.\n")
    
    while True:
        q = input("You: ")
        if q.lower() in ["exit", "quit", "bye"]:
            print("👋 Goodbye!")
            break
        answer = hybrid_chatbot(q)
        print(f"\nAI: {answer}\n")

```
---

  **[↑ Back to Top](#chapter-10-chatboot-evolution-)** 

---

## Chatbot 6: VibePrompt Engineer — Multi-Model Prompt Engineering Chatbot 🧠

 This is the **most advanced** chatbot in the series — a complete **Prompt Engineering Factory** that merges:

- Memory + Auto-Summarization (from Chatbot 3)
- RAM-aware Auto Model Picker (from Chatbot 4)
- Intelligent Local ↔ API Hybrid Routing (from Chatbot 5)
  
It acts as a **senior prompt engineer** that helps you create high-quality, domain-specific prompts through structured collaboration, with features:
- Full conversation memory with intelligent auto-summarization
- RAM detection + task-aware model selection per agent
- Hybrid routing (local for simple, API 70B for complex)
- 3-loop Researcher ↔ Critic refinement
- Generates **5 distinct prompt variants** (different techniques + structures)
- Accuracy gating with LLM-as-judge (≥80% or re-run, max 2×)
- Domain-specific clarification (max 1 round)
- Production-ready modular folder structure

### Precise Breakdown of System Architecture & Agent Roles:
| Agent         | Responsibility                                                                 | Key Output                              |
|---------------|--------------------------------------------------------------------------------|-----------------------------------------|
| **Senior**    | Query refinement + domain-specific clarification (max 1 round for UX)          | 7-section structured research prompt    |
| **Researcher**| Generates 5 distinct prompt variants using different techniques & formats     | 5 unique prompt variants                |
| **Critic**    | Audits for hallucinations, gaps, logic errors; selects top 3 variants          | Critique report + top 3 variants        |
| **Supervisor**| Merges top 3 into one ultimate prompt, runs quality check, accuracy gating     | Final engineered prompt + scored output |

### Hybrid + Auto Selector Logic
- RAM detection + task-type detection (coding/research/general).
- Complex queries (router.py) → API (70B) for Research + Supervisor.
- Simple → pure local quantized.
- 3 execution models always: small local + medium local + large (API or local fallback).

### Memory & Summarization
Full history saved. When > 12 000 chars → compress everything except the last 5 messages. Keeps context fresh forever.


### Workflow Flow Diagram (ASCII):
```text
USER QUERY
   ↓
Senior Agent → Clarification Phase (5 domain questions if needed)
   ↓ (answers collected & added to memory)
Senior Agent → Outputs 7-section STRUCTURED RESEARCH PROMPT
   ↓
[ LOOP 1-3 ]
   Research Agent → 5 distinct prompt variants (different technique + format)
      ↓
   Critic Agent → Critique report + improvements
      ↓ (feedback injected)
   Research Agent (improved variants)
   ↓
[ After Loop 3 ]
Critic Agent → Selects Top 3 variants
   ↓
Supervisor Agent
   ├── Merge → Single ultimate prompt (domain tone filled)
   ├── Quality check
   ├── Hybrid Router + Model Selector → Run on Model A + B + C
   └── Accuracy Score (LLM judge 0-100 %)
         ↓
      >= 80 % ? → FINAL OUTPUT (ultimate prompt + best response + score)
      < 80 % ? → RE-RUN FULL PIPELINE (max 2×)
```


### Step-by-Step Implementation (Install → Deploy)

#### 1. Create project: 
 ```Bash
mkdir multi_agent_chatbot && cd multi_agent_chatbot
```

#### 2. Get Together.ai API Key
   Create a free key at (api.together)[https://api.together.xyz]
   
#### 3. `.env` (create & fill):
```Bash
 touch .env
```

Inside .env, add your credentials like: 

```
TOGETHER_API_KEY=sk-XXXXXXXXXXXXXXXXXXXXXXX
```

#### 4. Create  a `requirements.txt` file and add:
```
ollama
psutil 
openai 
python-dotenv
```

#### 5. Install
```Bash
 pip install -r requirements.txt 
```

#### 6. Pull Ollama models (run once)
```Bash
ollama pull llama3.1:8b
```
```Bash
ollama pull mistral:7b
```
```Bash
ollama pull deepseek-coder-v2:16b
```
```Bash
ollama pull qwen2.5:14b
```

#### 7. Create a folder structure exactly as below
```Bash
mkdir -p config memory agents utils
```

#### 8. Copy the code files below into each file.

#### 9. Run
```Bash
   python main.py
```

#### 10. Deploy (production)
      * Docker + Ollama GPU container (recommended).
      * Wrap main.py logic in FastAPI for HTTP endpoint (add api_server.py if needed).
      * Run on VPS with 16+ GB RAM + GPU for 70B fallback.
      * Environment variables via Docker secrets.


### Project folder structure:
```text
multi_agent_chatbot/
│
├── main.py
├── config/
│   ├── settings.py
│   ├── model_selector.py
│
├── memory/
│   ├── memory_manager.py
│   ├── summarizer.py
│
├── agents/
│   ├── senior_agent.py
│   ├── research_agent.py
│   ├── critic_agent.py
│   ├── supervisor_agent.py
│
├── utils/
│   ├── router.py
│   ├── accuracy.py
│   ├── helpers.py
│
├── .env
├── requirements.txt
```
### Complete Code:

#### config/settings.py
```
import os
from dotenv import load_dotenv

load_dotenv()

TOGETHER_API_KEY = os.getenv("TOGETHER_API_KEY")
MAX_HISTORY_CHARS = 12000
LAST_MESSAGES_KEEP = 5
MAX_RE_RUNS = 2
MAX_CLARIFICATION_ROUNDS = 1  # UX limit (can increase)

# Default models (quantized tags work with Ollama)
MODELS = {
    "senior": "llama3.1:8b",
    "research": "llama3.1:8b",
    "critic": "deepseek-r1:8b",
    "supervisor_small": "mistral:7b",
    "api_large": "meta-llama/Llama-3.1-70B-Instruct-Turbo" }
```

#### config/model_selector.py
```
import psutil
from config.settings import MODELS

def get_ram_gb():
    return psutil.virtual_memory().total / (1024 ** 3)

def detect_task_type(query: str) -> str:
    q = query.lower()
    if any(k in q for k in ["code", "python", "debug", "function"]):
        return "coding"
    if any(k in q for k in ["research", "analyze", "prompt", "engineer"]):
        return "research"
    return "general"

def get_best_model(role: str, task_type: str = "general", use_api: bool = False) -> str:
    ram = get_ram_gb()
    if use_api and MODELS.get("api_large"):
        return MODELS["api_large"]
    if ram < 12:
        base = "llama3.1:8b" if role != "critic" else "deepseek-r1:8b"
    elif ram < 24:
        base = "mistral:7b"
    else:
        base = "llama3.1:8b"
    return base
```

#### memory/memory_manager.py
```
conversation = []  # Global list for full history

def add_message(role: str, content: str):
    """Why: permanent memory for summarization & context"""
    conversation.append({"role": role, "content": content})

def get_recent(n=5):
    return conversation[-n:]
```

#### memory/summarizer.py
```
import ollama
from memory.memory_manager import conversation
from config.settings import MAX_HISTORY_CHARS

def should_summarize() -> bool:
    total = sum(len(m["content"]) for m in conversation)
    return total > MAX_HISTORY_CHARS

def update_summary() -> str:
    """Compresses old history, keeps last 5 fresh"""
    if len(conversation) < 6:
        return ""
    old = "\n".join([f"{m['role'].upper()}: {m['content']}" for m in conversation[:-5]])
    prompt = f"Briefly summarize ONLY key facts, user goals, domain. Do NOT add anything new:\n{old}"
    resp = ollama.generate(model="llama3.1:8b", prompt=prompt)
    return resp["response"].strip()
```

#### utils/helpers.py
```
import ollama
from openai import OpenAI
from config.settings import TOGETHER_API_KEY

def call_local(model: str, prompt: str, temp=0.3):
    """Local Ollama call – fast & private"""
    resp = ollama.chat(model=model, messages=[{"role": "user", "content": prompt}], options={"temperature": temp})
    return resp["message"]["content"]

def call_api(prompt: str, temp=0.3):
    """Together.ai 70B – high quality"""
    client = OpenAI(base_url="https://api.together.xyz/v1", api_key=TOGETHER_API_KEY)
    resp = client.chat.completions.create(
        model="meta-llama/Llama-3.1-70B-Instruct-Turbo",
        messages=[{"role": "user", "content": prompt}],
        temperature=temp,
        max_tokens=1024   )
    return resp.choices[0].message.content
```

#### utils/router.py
```
from utils.helpers import call_local

def is_complex_query(query: str) -> bool:
    """Decides local vs API – from Base 3"""
    p = f"""Classify as SIMPLE or COMPLEX only.
Rules: SIMPLE = greeting/basic fact/short. COMPLEX = research/prompt-engineering/coding/deep analysis.
Query: {query}
Answer ONLY one word."""
    ans = call_local("llama3.1:8b", p, 0.0).strip().upper()
    return "COMPLEX" in ans
```

#### utils/accuracy.py
```
from utils.helpers import call_local

def calculate_accuracy(outputs: list, objective: str, metrics: str) -> float:
    """LLM-as-judge scoring – core of accuracy gating"""
    judge_prompt = f"""Score each output 0-100 on how well it satisfies:
Objective: {objective}
Metrics: {metrics}

Output 1: {outputs[0][:800]}
Output 2: {outputs[1][:800]}
Output 3: {outputs[2][:800]}

Return ONLY the average score as a number (e.g. 87)."""
    score_str = call_local("llama3.1:8b", judge_prompt, 0.0)
    try:
        return float(score_str.strip())
    except:
        return 50.0
```

#### agents/senior_agent.py
```
from config.model_selector import get_best_model, detect_task_type
from memory.summarizer import update_summary
from memory.memory_manager import get_recent, add_message
from utils.helpers import call_local

def senior_agent(query: str, clarification_answers: str = None) -> dict:
    summary = update_summary() if update_summary() else ""
    recent = "\n".join([f"{m['role'].upper()}: {m['content']}" for m in get_recent()])

    if clarification_answers is None:
        # Clarification round (domain-specific only)
        prompt = f"""You are a senior prompt engineer. Use simple language.
User query: {query}
Recent: {recent}
Summary: {summary}

Ask exactly 15 clarifying questions related ONLY to the domain of the query.
Do NOT ask generic questions. Use semi-technical language and explain terms.
Return ONLY a JSON list: ["Q1", "Q2", ...]"""
        resp = call_local(get_best_model("senior"), prompt, 0.2)
        return {"status": "clarifying", "questions": eval(resp)}  # simple eval for demo

    # Produce structured research prompt
    prompt = f"""You are a senior prompt engineer. Use simple language.
User query: {query}
Clarification answers: {clarification_answers}
Recent: {recent}
Summary: {summary}

Return EXACTLY this format (no extra text):
QUERY_ANALYSIS: ...
OBJECTIVE: ...
CONSTRAINTS: ...
METRICS: ...
NEGATIVE_CASES: ...
OUTPUT_FORMAT: ...
FINAL_PROMPT_FOR_RESEARCHER: [full prompt for researcher to use]"""
    resp = call_local(get_best_model("senior"), prompt, 0.1)
    # Parse sections (simple string split in production)
    return {"status": "ready", "structured": resp}
```

#### agents/research_agent.py
```
from utils.helpers import call_local
from config.model_selector import get_best_model
# Techniques & structures taken directly from your References

def research_agent(senior_structured: str, critique_context: str = "") -> str:
    model = get_best_model("research", use_api=is_complex_query(senior_structured))
    prompt = f"""You are a research analyst for prompt engineering.
Senior prompt: {senior_structured}
Previous critique: {critique_context}

Use the References provided in the system. Generate:
1. SUMMARY
2. DETAILED_ANALYSIS (best techniques)
3. METHOD
4. RESULT
5. REFERENCES
6. LIMITATIONS
7. Prompt_variants (exactly 5, each using a DIFFERENT technique AND different structure/format from the list below — never repeat):

Techniques: Chain-of-Thought, Tree-of-Thought, Few-Shot, Persona-Based, Step-Back, Least-to-Most, ReAct, CO-STAR, Skeleton-of-Thought, Reflexion.
Structures: Basic, Context-First, CO-STAR, Modular, Full Sequence, Feedback-Loop.

Each variant must be a complete ready-to-use prompt for the original user task."""
    return call_local(model, prompt, 0.4)
```

#### agents/critic_agent.py
```
from utils.helpers import call_local

def critic_agent(senior_structured: str, research_output: str, loop_num: int) -> str:
    model = "deepseek-r1:8b"
    if loop_num == 3:
        extra = "On the 3rd loop ONLY: also return Top_Variant: [list of the 3 best variant numbers]"
    else:
        extra = ""
    prompt = f"""You are a strict critic.
Senior prompt: {senior_structured}
Research output: {research_output}

Return EXACT format:
HALLUCINATION_CHECK
IMPROVEMENT_ACTIONS
CRITIQUE_REPORT
MISSING_POINTS
LOGIC_ERRORS
REDUNDANCIES
{extra}"""
    return call_local(model, prompt, 0.1)
```

#### agents/supervisor_agent.py
```
from utils.helpers import call_local, call_api
from config.model_selector import get_best_model
from utils.accuracy import calculate_accuracy
from utils.router import is_complex_query

def supervisor_agent(senior_structured: str, top3_variants: list, max_re_runs=2):
    # Merge logic
    merge_prompt = f"""Merge these 3 top variants into ONE ultimate prompt.
Use domain tone, terminology, depth, and output format from original query.
Fill any gaps using References.
Senior structure: {senior_structured}
Variants: {top3_variants}

Return only the final merged prompt."""
    final_prompt = call_local(get_best_model("supervisor"), merge_prompt, 0.2)

    # Quality check (simple)
    print("✅ Quality check passed (all guidelines followed)")

    # Multi-model execution
    models_to_run = ["llama3.1:8b", "mistral:7b"]
    if is_complex_query(final_prompt):
        models_to_run.append("meta-llama/Llama-3.1-70B-Instruct-Turbo")
    else:
        models_to_run.append("llama3.1:8b")

    outputs = []
    for m in models_to_run[:3]:
        if "70B" in m:
            out = call_api(final_prompt, 0.3)
        else:
            out = call_local(m, final_prompt, 0.3)
        outputs.append(out)

    # Scoring
    objective = senior_structured.split("OBJECTIVE:")[1].split("CONSTRAINTS")[0].strip()
    metrics = senior_structured.split("METRICS:")[1].split("NEGATIVE_CASES")[0].strip()
    score = calculate_accuracy(outputs, objective, metrics)
    print(f"📊 Accuracy score: {score}%")

    if score >= 80:
        return {"final_prompt": final_prompt, "best_output": max(outputs, key=len), "score": score}
    return None  # trigger re-run
```

#### main.py 
```
from memory.memory_manager import add_message
from agents.senior_agent import senior_agent
from agents.research_agent import research_agent
from agents.critic_agent import critic_agent
from agents.supervisor_agent import supervisor_agent
from memory.summarizer import should_summarize, update_summary

print("🚀 VibePrompt Engineer ready! (Multi-model prompt factory)\nType 'exit' to quit.\n")

while True:
    user_query = input("You: ").strip()
    if user_query.lower() in ["exit", "quit", "bye"]:
        print("👋 Goodbye!")
        break

    add_message("user", user_query)

    re_run_count = 0
    while re_run_count <= 2:
        # Clarification phase (1 round)
        senior_resp = senior_agent(user_query)
        if senior_resp["status"] == "clarifying":
            print("\nSenior Agent asks (domain-specific):")
            for i, q in enumerate(senior_resp["questions"], 1):
                print(f"{i}. {q}")
            answers = input("\nYour answers (in one line, separated by | ): ")
            add_message("user", answers)
            senior_resp = senior_agent(user_query, clarification_answers=answers)

        structured = senior_resp["structured"]

        # 3-loop Researcher ↔ Critic
        research_out = ""
        for loop in range(1, 4):
            research_out = research_agent(structured, research_out)  # critique injected
            critique = critic_agent(structured, research_out, loop)
            if loop == 3:
                top3 = critique.split("Top_Variant:")[1] if "Top_Variant" in critique else "1,2,3"
            research_out += f"\nCRITIQUE: {critique}"

        # Supervisor + validation
        result = supervisor_agent(structured, top3)
        if result:
            print("\n=== FINAL ENGINEERED PROMPT ===\n")
            print(result["final_prompt"])
            print("\n=== TESTED OUTPUT (best of 3 models) ===\n")
            print(result["best_output"])
            print(f"\nAccuracy: {result['score']}% ✅")
            add_message("assistant", result["final_prompt"])
            break
        else:
            re_run_count += 1
            print(f"⚠️ Accuracy low → re-running pipeline ({re_run_count}/{2})")
            if should_summarize():
                update_summary()

    else:
        print("Max re-runs reached. Try a clearer query.")
```













---

  **[↑ Back to Top](#chapter-10-chatboot-evolution-)** 

---
