# Chapter-3-Memory-Management-Multi-turn-Chats

Now that you can call the OpenAI API and control response parameters, the next critical skill is **memory management**. Real-world chatbots, agents, and assistants must remember previous messages across multiple turns.

This chapter explains why LLMs are stateless, how conversation history actually works, and how to maintain context effectively in code.

---

## Why Chat History Matters

Consider two cases.

### Case 1 — Without History** -- Like we did in the past lecture on *API calling*

```python
messages = [
   {"role": "system", "content": "You are a friendly chatbot."},
   {"role": "user", "content": "What is my name?"} 
  
  ]
```
The model receives no earlier context.

**Result**:  *"I don't know your name."*


### Case 2 — With Full History

```python
messages = [
   {"role": "system", "content": "You are a friendly chatbot."},
   {"role": "user", "content": "Hi, my name is Mirza"},
   {"role": "assistant", "content": "Nice to meet you, Mirza."},
   {"role": "user", "content": "What is my name?"}  
   
   ]
```

Now the model sees:

The user introduced their name earlier
**Response**:  *"Your name is **Mirza**."*
The model did not remember — it simply read the earlier message again.

---

## How LLMs Handle & Remember Conversations (Multi-Turn Chat Explained)

**Previous conversation ≠ automatically remembered**

<img src="../../assets/images/phase4-conversation-history.png"
width="25%"
align="left"
style="margin-right: 25px; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);"
alt="Conversation history diagram showing how full message history is sent with every API call">

Large language models (GPT, Claude, LLaMA, Grok, etc.) are stateless by design. They do not retain any information between separate API calls or terminal sessions.
Every time you send a request, the model sees only the exact messages you include in that request.

**Key Rule:**
If you want the model to remember names, previous answers, assigned roles, tasks in progress, or any context, you must send the full relevant history as part of every new API call.
This is exactly how production chat systems (ChatGPT, Claude, custom agents) create the illusion of memory.

---

## How the Model Actually Sees a Conversation
Internally, the model receives a single combined sequence of text tokens.

### Conversation Flow:
```markdown
                     System message (optional – sets personality/rules)
                                          ↓
                                    User message 1
                                          ↓
                                  Assistant reply 1
                                          ↓
                                   User message 2
                                          ↓
                                  Assistant reply 2
                                          ↓
                                    User message 3
                                          ↓
                               ... (all previous turns)
                                          ↓
                Current user message ← model predicts next assistant reply here
                
```
You are responsible for maintaining and sending the entire conversation history with every new request. The model reads the whole sequence from top to bottom and generates only the next assistant message.

---

## Ways Developers Make LLMs “Remember” Things

There are several proven patterns developers use to maintain conversation state. We’ll start with the most educational approaches used in open-source models (LLaMA family) before moving to production-grade techniques in later sections.

---

### Method 1: Manual Tagging (LLaMA-2 / LLaMA-3 Style – Most Educational)

Many open-source chat models (LLaMA-2, LLaMA-3, Mistral, etc.) expect **special instruction tags** to distinguish between system instructions, user input, and assistant responses.

**Key tags used by these models:**
- `<s>` — Start of a message
- `</s>` — End of a message
- `[INST]` — Start of an instruction (user prompt)
- `[/INST]` — End of an instruction

#### Example: Building a Multi-turn Chat Manually

```python
# Turn 1
prompt_1 = "What are fun activities I can do this weekend?"
response_1 = llama(prompt_1)          # Assume model replies: "Hiking, cycling, yoga, movies."

# Turn 2 – Manually construct full history with tags
full_chat_prompt = f"""
<s>[INST] {prompt_1} [/INST]
{response_1}
</s>
<s>[INST] Which of these would be good for my health? [/INST] 
"""

response_2 = llama(full_chat_prompt, add_inst=False)   # Disable auto-tagging
```

**Result:** The model now sees the complete conversation and answers correctly, for example:
“Hiking and cycling are excellent for cardiovascular health…”

**When to use this method:**

* Learning how chat formatting actually works under the hood
* Working with older LLaMA-2 models or custom fine-tunes
* Needing maximum low-level control
* Educational projects and experimentation

---
### Method 2: Using a Helper Function (llama_chat / chat wrapper)

Most modern libraries and courses provide a helper function that automatically builds the correctly tagged prompt for you — eliminating manual string formatting errors.

#### Example pattern (very common in tutorials):

```python
from utils import llama_chat   # Helper function provided by the course/library

prompts = [
    "What are fun weekend activities?",
    "Which of these would be good for my health?"
]

responses = []

# First turn
resp1 = llama(prompts[0])
responses.append(resp1)

# Second turn – helper automatically adds history with correct tags
resp2 = llama_chat(prompts, responses)
print(resp2)   # Model now has full context
```
**What the helper does internally:**
It constructs the same tagged structure shown in Method 1, but automatically handles `<s>`, `</s>`, `[INST]`, and `[/INST]` tags.

**When to use this method:**

* Following structured courses or official documentation
* Avoiding manual string formatting mistakes
* Rapid prototyping and learning
* When working with libraries that provide `chat()` or `chat_completion()` wrappers

---

**Pro Tip:
Manual tagging (Method 1) teaches you the underlying mechanics. Helper functions (Method 2) save time in real projects. Both approaches follow the same core principle: you must send the full conversation history with every new request.**

---

### Method 3: Storing Messages in a List (Most Popular Production Method – OpenAI Style)

This is the **industry standard** for almost every real chatbot that works today — including ChatGPT, Grok, Claude web/app interfaces, and most custom production applications. 

You maintain **one growing list** of messages that represents the entire conversation history.

### Structure of Messages

Each message is a dictionary with exactly two fields:
- **`role`**: Who sent the message (`system`, `user`, or `assistant`)
- **`content`**: The actual text of the message

**Possible roles:**
- **`system`** — Sets behavior, personality, and rules (usually at the beginning)
- **`user`** — Human input / questions
- **`assistant`** — Model’s previous responses

#### Implementation Example: 

**Simple Interactive Loop:** 
```python
conversation = [
    {"role": "system", "content": "You are a helpful and friendly chatbot."}
]

while True:
    user_input = input("You: ")
    
    # Add user message to history
    conversation.append({
        "role": "user",
        "content": user_input
    })
    
    # Send full conversation to the model
    reply = chat(conversation)          # Your chat function
    
    print("AI:", reply)
    
    # Save the assistant's reply to history
    conversation.append({
        "role": "assistant",
        "content": reply
    })

```
    
**Reusable Function Approach:**
```python
def chat_with_memory(conversation, new_user_message, model="gpt-4o", temperature=0.7):
    """
    Append new user message, call the model with full history,
    Then append the assistant's reply to the conversation.
    """
    # Add the new user input
    conversation.append({
        "role": "user",
        "content": new_user_message
    })
    
    # Call the model with the complete conversation history
    response = client.chat.completions.create(
        model=model,
        messages=conversation,
        temperature=temperature
    )
    
    reply = response.choices[0].message.content
    
    # Save the assistant's reply to maintain memory
    conversation.append({
        "role": "assistant",
        "content": reply
    })
    
    return reply
```

**Object-Oriented Approach (Recommended for Larger Projects):**
```python
class Conversation:
    def __init__(self, system_prompt="You are a helpful assistant."):
        self.messages = []
        if system_prompt:
            self.messages.append({
                "role": "system", 
                "content": system_prompt
            })
    
    def generate(self, user_question, model="gpt-4o", temperature=0.7):
        # Append the new user question
        self.messages.append({
            "role": "user", 
            "content": user_question
        })
        
        # Call the model with full conversation history
        response = client.chat.completions.create(
            model=model,
            messages=self.messages,
            temperature=temperature
        )
        
        reply = response.choices[0].message.content
        
        # Append the assistant's response to maintain memory
        self.messages.append({
            "role": "assistant", 
            "content": reply
        })
        
        return reply
```

**Usage Example:**
```python
chat = Conversation("You are a friendly AI tutor specializing in prompt engineering.")
print(chat.generate("What is few-shot prompting?"))
print(chat.generate("Can you give me an example?"))   # Remembers previous context
```

**Explanation (Analogy):**
Think of the messages list as a notebook that records the entire conversation. Every time you talk to the model, you show it the full notebook so it can continue the conversation naturally.

**Key Advantages of this Method:**

* Simple and clean
* Works with OpenAI, Anthropic, Grok, Ollama, and most providers
* Easy to debug and inspect conversation history
* Foundation for more advanced memory techniques (summarization, vector memory, etc.)

---

> 💡 **Want to see this in a working chatbot?**  
> **[🤖 Open Full Single-Agent Chatbot Implementation](../Chapter-10-Chatbot-Evolution/README.md)**

---




















