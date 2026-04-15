# Chapter 2: API Calling 🔌



Now that you understand how to control AI responses with parameters like **temperature**, **top_p**, **max_tokens**, and **stop sequences**, it's time to move into real **developer mode**.

In developer mode, you control everything through code. You can set parameters directly in the API call and build powerful prompts using **message roles** (system + user + assistant). This approach becomes essential when connecting the model to tools, functions, external data, databases, or building full applications.

We will use the official **OpenAI Python library** (`openai`) to interact with various AI models for text, code, images, speech, and more. 

---

**[← Back to Chapter 1](../Chapter-1-Controlling-AI-Response-Parameters/README.md)** | **[Next Topic →](#)** | **[↑ Back to Top](#chapter-2-api-calling-)** | **[Back to Phase 4 Main Page](../README.md)**

---

## Models You Can Interact With

The OpenAI library allows you to work with different types of specialized models.

| Model Type     | Example Models                  | Purpose                                      |
|----------------|---------------------------------|----------------------------------------------|
| Text AI        | GPT-4o, GPT-4.5, GPT-4o-mini   | Chatbots, writing, knowledge retrieval, reasoning |
| Image AI       | DALL·E 3                        | AI-generated images and illustrations        |
| Speech AI      | Whisper-1                       | Speech-to-text transcription                 |
| Code AI        | GPT-4o (with code focus)        | Code generation, debugging, explanation      |
| Embeddings     | text-embedding-ada-002 / text-embedding-3-large | Search, AI memory, recommendations, RAG      |

---

## OpenAI Library: Key Functions

Here are the most important functions you'll use frequently:

| Function                                 | Purpose                                      | Typical Model Used          | When to Use                                      |
|------------------------------------------|----------------------------------------------|-----------------------------|--------------------------------------------------|
| `client.chat.completions.create()`       | Generates conversational AI responses        | gpt-4o, gpt-4.5             | Chatbots, content writing, Q&A, agents           |
| `client.images.generate()`               | Text-to-image generation                     | dall-e-3                    | Visual content, concept art, branding            |
| `client.audio.transcriptions.create()`   | Speech-to-text transcription                 | whisper-1                   | Meetings, podcasts, voice notes                  |
| `client.embeddings.create()`             | Creates vector embeddings                    | text-embedding-3-large      | Semantic search, memory, recommendation systems  |

---

## Installation & Setup

**Step 1: Install & set up OpenAI library**

   ```bash
   pip install openai
   ```


**Step 2: Get your API key**:
     - Go to platform.openai.com
     - Log in → API keys → Create new secret key
     - Copy it (starts with sk-)

**Step 3: Setting up API key**

Set up the client (recommended way):

 ```Python
from openai import OpenAI
import os

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY")) # Paste your API Key here
 ```

**Security Tip: Never hardcode your API key. Use environment variables or a .env file (We'll see the Use of the .env file in the upcoming topics)**

---

## Basic API Calling
Here's a complete, modern example using the current best practices:

 ```Python

from openai import OpenAI
import os

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))   # ← Replace with your key
prompt_text = input("Ask anything: ")     # Define task, i.e., "Explain how neural networks work in simple terms."

# Calling OpenAI's API

response = openai.ChatCompletion.create ( 
    model="gpt-4",                       # OpenAI has different models (e.g., GPT-3.5, GPT-4), and this ensures we use GPT-4
    

  """ messages is a list of dictionaries, each representing a message.
"role": "user" → Indicates that this message comes from the user.
"content": prompt_text → The actual text input that the user provides """  
    
    messages=[ { "role" :  "user" ,  "content" :  prompt_text } ]   

)

# 'response' is a dictionary that contains the AI's generated response. 'Choices' is a list that contains possible completions (responses). [0] accesses the first index (and usually only) response. The 'message' contains the AI's message. 'content' extracts the actual text response from GPT-4

print (response["choices"][0]["message"]["content"])

 
""" Or a safer version:
ai_message = response.get("choices", [{}])[0].get("message", {}).get("content", "No response")
print(ai_message)  """

 ```

---
## Using System and User Roles

** Without system message (simple one-off questions)**:

 ```Python
messages = [ {"role": "user", "content": prompt _text } ]
 ```

- AI generates a response without any preset instructions. The model defaults to a general AI assistant but may not always align with your preferred tone.
- Less control over response behavior.
- If you just need a simple one-time answer, Open-ended Q&A (Casual Use), or Direct API Testing (Simple Prompts) you can use it 

**With system message (recommended for consistent behaviour)**:

 ```Python
messages = [
               {"role": "system", "content": prompt_text1 },  # system: Sets the behavior and personality of the AI (instructions)
               {"role": "user", "content": prompt_text2 }  # user: Your actual query or input
]

# Pro Tip: A well-written system prompt combined with API parameters often produces better results than a long user prompt alone
 ```

- The system message sets the behavior, personality, or rules for the assistant.
- It guides the AI’s responses throughout the conversation.
- More control over response style and behavior.
- Helps define the AI’s character or expertise.

You can make certain changes in this base template for different use cases as per the requirement

---

## Combining Parameters with API Calls
You can apply everything you learned in Chapter 1 directly to code:

 ```Python
messages = [ {"role": "user", "content": prompt _text } ],

# THERE IS NO NECESSITY TO USE ALL  OF THEM. YOU CAN USE ANY SINGLE ONE SPECIFICALLY AS NEEDED. Mostly,  “temperature” IS USED

    temperature=0.7,      # Controls randomness (0 = predictable, 1 = creative)  also mostly used then other parameter
    max_tokens=300,       # Limits response length (shorter = more focused)
    top_p=0.9,            # Controls diversity of response  
    stop=["\n\n"]         # clean stop


```
---

## Step-by-Step Implementation Guideline:

- Install the `openai` library and set up your API key securely.
- Initialize the OpenAI client once (reuse it across your application).
- Decide your goal (chat, code, creative, factual, etc.).
- Craft a clear system prompt for behavior + user prompt for the task.
- Choose appropriate parameters (temperature, max_tokens, etc.) based on the use case.
- Make the API call and handle the response (response.choices[0].message.content).
- Add error handling, retries, and logging for production use.

**Best Practice: Start simple with a good system prompt and low temperature. Iterate by adjusting parameters and testing different models.**

---

**Example of complete code with System Role & Parameters:**

 ```Python
from openai import OpenAI
import os

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))   # ← Replace with your key

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a senior Python developer and helpful coding mentor. Always explain code clearly with examples."},
        {"role": "user", "content": "How do I implement a retry mechanism with exponential backoff?"} ]
    temperature=0.7,      
    max_tokens=300,      
    top_p=0.9,           
    stop=["\n\n"]         
)

ai_message = response.get("choices", [{}])[0].get("message", {}).get("content", "No response")

print(ai_message) 

 ```

---
Phase 4 of "All You Need to Know About Prompt Engineering" — Portfolio Project by Mirza (BS AI Student, Karachi)




