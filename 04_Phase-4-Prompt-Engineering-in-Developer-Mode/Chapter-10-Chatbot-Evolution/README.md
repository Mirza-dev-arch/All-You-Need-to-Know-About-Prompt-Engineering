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


### Auto Model Selection Logic

The chatbot automatically selects the best model based on your RAM and query type:

- **Low RAM (<12 GB)**: Uses lightweight models like Phi-3 Mini
- **Medium RAM (12–24 GB)**: Llama 3.1 8B (default)
- **High RAM (24–40 GB)**: Qwen2.5 14B
- **Very High RAM (40+ GB)**: Llama 3.1 70B

It also applies task-specific overrides (e.g., DeepSeek-Coder for coding tasks).

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
