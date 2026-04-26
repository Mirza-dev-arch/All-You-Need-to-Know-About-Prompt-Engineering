# Chapter 4: Local LLMs & Running Models on Your Machine 💻

---

## Table of Contents
- [Introduction to LLaMA Models – Running LLMs Locally](#introduction-to-llama-models--running-llms-locally)
- [What is LLaMA?](#what-is-llama)
- [Why LLaMA Changed Everything](#why-llama-changed-everything)
- [LLaMA Family Overview (2023–2026)](#llama-family-overview-2023-2026)
- [Three Main Types of LLaMA Models](#three-main-types-of-llama-models)
- [Ways to Access LLaMA Models](#ways-to-access-llama-models)

  
**[← Back to Chapter 3](../Chapter-3-Memory-Management-Multi-turn-Chats/README.md)** | **[Next Section →](#)** | **[↑ Back to Top](#chapter-4-local-llms--running-models-on-your-machine-)** | **[Back to Phase 4 Main Page](../README.md)**

---

## Introduction to LLaMA Models – Running LLMs Locally

LLaMA is one of the most influential **open-weight** large language model families ever released. It made powerful AI accessible to everyone — not just companies with huge server farms.

---

## What is LLaMA?

**LLaMA** stands for **Large Language Model Meta AI**. It is a family of open-weight models created by Meta AI.

LLaMA models excel at:
- Text generation
- Answering questions
- Summarizing documents
- Helping write or understand code
- Having natural conversations

### Key Difference from ChatGPT / Gemini / Claude

→ You can **download the actual model files (weights)** and run them **on your own computer**.

**What are “weights”?**  
In simple terms, weights are the learned numbers inside the model that decide how strongly different words and ideas connect to each other.  

Downloading weights = downloading the **“brain”** of the AI.

### Open-Weight vs Closed Models

<img src="../../assets/images/phase4-openvs-closed-weight.png"
     width="75%"
     align="center"
     style="display: block; margin: 25px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);"
     alt="Open-weight vs Closed-weight models comparison - LLaMA vs proprietary models like GPT and Claude">
<br clear="all"/>

---

## Why LLaMA Changed Everything

Before LLaMA (early 2023), almost all strong models were locked behind paid APIs.

After LLaMA:

- Millions of people started running powerful AI **locally** on their laptops
- Researchers could study and improve how large models really work
- Startups built custom models without paying huge API bills
- Communities created thousands of fine-tuned versions (Alpaca, Vicuna, WizardLM, Orca, etc.)

---

## LLaMA Family Overview (2023–2026)

| Generation   | Year      | Key Sizes                  | Context Window     | Notes / Improvements                     |
|--------------|-----------|----------------------------|--------------------|------------------------------------------|
| LLaMA 1      | 2023      | 7B, 13B, 30B, 65B          | ~2k–4k             | First shock to the industry              |
| LLaMA 2      | 2023      | 7B, 13B, 70B               | 4k                 | Better license, improved chat versions   |
| LLaMA 3      | 2024      | 8B, 70B                    | 8k → 128k          | Much better reasoning capabilities       |
| LLaMA 3.1    | 2024      | 8B, 70B, 405B              | 128k               | Very strong open model                   |
| LLaMA 3.2    | 2025      | 1B, 3B, 11B, 90B (vision)  | 128k+              | Small + multimodal (vision support)      |
| LLaMA 4      | 2025–26   | Various (rumored)          | 1M+?               | Expected to be even stronger             |

---

## Three Main Types of LLaMA Models

<img src="../../assets/images/phase4-types-of-llama-model.png"
     width="75%"
     align="center"
     style="display: block; margin: 25px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);"
     alt="Three main types of LLaMA models - Base, Chat, and Fine-tuned variants">
<br clear="all"/>

---

## Ways to Access LLaMA Models

LLaMA models (Meta’s open-weight family) can be used in three main ways, each offering different trade-offs in **ease of use**, **cost**, **privacy**, **speed**, and **control**.

Here are the three common approaches, ranked from **easiest** to **most advanced**:

---

### 1. Hosted API (Recommended for Most People)

You send prompts to someone else’s server and receive responses — exactly like using ChatGPT.

| Provider          | Typical Cost (8B model)       | Strengths                          | Best For                              |
|-------------------|-------------------------------|------------------------------------|---------------------------------------|
| Together.ai       | ~$0.06–0.20 / 1M tokens       | Fast, many Llama variants, easy API| Beginners, prototyping, production    |
| Groq              | Very low latency              | Extremely fast inference           | Real-time apps, chatbots              |
| Fireworks.ai      | Competitive pricing           | Good Llama 3.1 & 3.2 support       | Speed + cost balance                  |
| Amazon Bedrock    | Pay-per-use                   | Enterprise-grade, compliance       | Companies needing SOC2 / audit logs   |
| Google Vertex AI  | Higher cost                   | Very reliable, integrated with GCP | Google Cloud users                    |
| Microsoft Azure   | Enterprise pricing            | Good for Windows / Azure shops     | Large organizations                   |

#### Quick Code Example (Together.ai style – most popular in 2026)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.together.xyz/v1",
    api_key="your-together-api-key"
)

response = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct-Turbo",
    messages=[{"role": "user", "content": "Write a short poem about Karachi at night."}],
    temperature=0.7,
    max_tokens=200
)

print(response.choices[0].message.content)
```
#### Why most developers start here:

* No GPU required
* Instant setup (just an API key)
* Access to the newest LLaMA variants within days of release
* Easy to switch between models


### 2. Cloud Deployment (You Host It)
You rent GPU servers and run the model yourself.

| Platform          | Typical Cost (A100/H100)      | Best For                           | Difficulty                            |
|-------------------|-------------------------------|------------------------------------|---------------------------------------|
| RunPod            | $0.4–1.2/hour                 | Fast setup,many GPU types          |Easy                                   |
| Vast.ai           | Often cheapest                | Peer-to-peer GPUs                  | Medium                                |
| Lambda Labs       | $1.1–2.5/hour                 | "Reliable, good support"           | Easy                                  |
| AWS SageMaker     | Higher                        | "Enterprise, auto-scaling"         | Hard                                  |
| Google Cloud      | High                          | TPUs + good integration            | Medium                                |

#### Typical Workflow:

* Rent a GPU instance (A100 40GB or H100)
* Install Ollama / vLLM / llama.cpp
* Pull the model (e.g., ollama run llama3.1:70b)
* Expose an OpenAI-compatible API endpoint
* Point your code to your own server

#### Best for:

* Full data privacy
* Running custom fine-tuned versions
* Very high volume usage (becomes cheaper than APIs after ~10–20M tokens/day)
  
### 3. Local Installation (Run on Your Own Computer)
Download and run models completely offline on your laptop or desktop.

| Tool              | Ease of Use                   | Speed        |  Model Support               | Best For                              |
|-------------------|-------------------------------|--------------|------------------------------|---------------------------------------|
| Ollama            | ★★★★★                      | Fast         | Almost all GGUF model        | "Beginners, daily use "               |
| LM Studio         | ★★★★★                      | Fast         | GGUF + Hugging Face          |  GUI lovers                           |
| GPT4All           | ★★★★                        | Good        | GGUF                          | Simple desktop app                    |
| llama.cpp         | ★★★                         | Fastest      |  GGUF (most efficient)       | Maximum speed & low RAM                |
| KoboldCPP         | ★★★★                        | Good         | GGUF + extras                | Roleplay & creative writing           |

#### Hardware Requirements

| Model Size         | RAM / VRAM Needed      | Realistic Speed (tokens/sec)       | Laptop?              |
|-------------------|-------------------------|------------------------------------|----------------------|
| 3B–8B             | 8–16 GB                 | 40–120 t/s                         | Yes                  |
| 13B–34B           | 20–40 GB                |  20–60 t/s                         | Possible             |
| 70B               | 48–80 GB                | 8–30 t/s (quantized)               | Hard                 |
| 405B              | 300+ GB                 | Very slow                          | No (Server required) |
 
#### Quick Start with Ollama

```Bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh
```

```Bash
# Download and chat immediately
ollama run llama3.1:8b
# or
ollama run llama3.1:70b
```

#### Python Usage:

```Python
import ollama

response = ollama.chat(
    model='llama3.1:8b',
    messages=[{'role': 'user', 'content': 'Tell me a short story about a cat in Karachi.'}]
)

print(response['message']['content'])
```

### Quick Decision Guide

| Goal                                    | Best Choice                     | Why                          |
|-----------------------------------------|---------------------------------|------------------------------|
| Just want to try Llama quickly          | Ollama + llama3.1:8b            | "Zero setup, instant"        |
| Build a real app/product                | Together.ai or Groq API         | "Reliable, scalable, fast"   |
| Maximum privacy & no recurring cost     |  Local with Ollama / LM Studio  | 100% offline after download  |
| Fine-tune your own version              | Local + LoRA on RunPod/Vast.ai  | Full control                 |
| Need 70B+ quality on low budget         | Together.ai Llama-3.1-70B       | Best price/performance       |
| Enterprise / compliance                 | Amazon Bedrock or Azure         | "Audits, SLAs, security"     |

---







---

**Phase 4 of "All You Need to Know About Prompt Engineering" — Portfolio Project by Mirza (BS AI Student, Karachi)**

**[← Back to Chapter 3](../Chapter-3-Memory-Management-Multi-turn-Chats/README.md)** | **[Next Section →](#)** | **[↑ Back to Top](#chapter-4-local-llms--running-models-on-your-machine-)** | **[Back to Phase 4 Main Page](../README.md)**

