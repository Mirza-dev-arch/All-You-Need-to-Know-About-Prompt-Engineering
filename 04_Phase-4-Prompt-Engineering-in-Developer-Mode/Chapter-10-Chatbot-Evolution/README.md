# Chapter-10-Chatbot-Evolution
→ From simple chatbots to stateful, memory-aware, production-ready agents.

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








