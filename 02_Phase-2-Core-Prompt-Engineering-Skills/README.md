# Phase 2: Core Prompt Engineering Skills 🔧

![Phase 2](https://img.shields.io/badge/Phase-2-Core%20Skills-388E3C?style=for-the-badge)
![Intermediate](https://img.shields.io/badge/Level-Intermediate-orange?style=for-the-badge)
![Technique+Structure+Format](https://img.shields.io/badge/Focus-Layering-brightgreen?style=for-the-badge)

**Now we move from “what is prompting” to “how to prompt well.”**  
In this phase you will master the **three core layers** every professional prompt uses: Technique, Structure, and Format — plus System Prompts and Safety rules.

---

## 📋 Table of Contents

- [Understanding Technique vs Structure vs Format](#understanding-technique-vs-structure-vs-format)
- [1. Prompt Technique](#1-prompt-technique)
- [2. Prompt Structure](#2-prompt-structure)
- [3. Prompt Format](#3-prompt-format)
- [Comparison Table](#comparison-table)
- [Setting Up a System Prompt](#setting-up-a-system-prompt)
- [Creating Maintainable System Prompts](#creating-maintainable-system-prompts)
- [Best Practices & Safety System Messages](#best-practices--safety-system-messages)
- [Prompting Techniques](#prompting-techniques)
- [Prompt Structures](#prompt-structures)
- [Prompting Format](#prompting-format)
- [How to Combine Technique + Structure + Format](#how-to-combine-technique-structure-format)
- [Top 15 Things to Check Before Layering](#top-15-things-to-check)
- [Top 10 Things That Kill Your Prompt](#top-10-things-that-kill-your-prompt)
- [Model-Specific Quirks](#model-specific-quirks)

**[← Back to Main README](../../README.md)**  **[← Phase 1](../01_Phase-1-Foundations/README.md)**  **[Next → Phase 3](../03_Phase-3-Mastery-Experimentation/README.md)**

---

## Understanding Technique vs Structure vs Format

As a prompt engineer, inputs for LLMs like Grok include understanding **"prompt technique," "prompt structure,"** and **"prompt format"** — they are related but operate at different levels of abstraction and focus in a well-designed prompt.



## 1. Prompt Technique

**Definition:** This is the high-level method or strategy you employ to elicit the desired response from an LLM. It's about leveraging psychological or logical patterns that models have learned from training data to improve output quality, reduce hallucinations, or handle complex tasks.

**Key Focus:** Why you’re prompting in a certain way and what cognitive process you’re simulating.

**When to Use:** For solving specific problems like reasoning, creativity, or bias mitigation.

**Examples:**
- **Chain-of-Thought (CoT):** "Solve this math problem step by step: What is 15% of 200?"
- **Few-Shot Prompting:** "Translate these: 'Hello' → 'Bonjour'. 'Goodbye' → 'Au revoir'. Now: 'Thank you' →"
- **Role-Playing:** "You are a pirate captain. Describe finding treasure."

**Prompt Engineer Tip:** Techniques are iterative—test them with A/B variations to measure response accuracy or coherence. They're the "art" side of prompting, often requiring experimentation.


---

## 2. Prompt Structure

**Definition:** This refers to the overall organisation and logical flow of the prompt's content. It's like the skeleton that holds everything together, ensuring the model processes information in a sequence that makes sense.

**Key Focus:** Hierarchy and sequencing of elements.

**When to Use:**  Helps manage complexity, especially in multi-step tasks or long prompts, by breaking them into digestible parts.

**Examples:**
- Basic: `[Instructions] + [Context] + [Query]`
- Advanced: Sections like "Task", "Constraints", "Examples", "Input"

**Prompt Engineer Tip:** Good structure minimises token waste and improves consistency. Always start with clear instructions to "prime" the model, and end with a direct call-to-action to focus the output.


---

## 3. Prompt Format

**Definition:** The stylistic and syntactic presentation of the prompt that makes it easier for the model to parse and respond.

**Key Focus:** Readability and machine-friendliness.

**When to Use:** To enforce output styles or structured data.

**Examples:**
- Triple quotes or XML tags
- Markdown for bold and bullets
- JSON/YAML for structured responses

**Prompt Engineer Tip:** Avoid over-formatting — it can make prompts brittle.

---

## Comparison Table

| Aspect                  | Prompt Technique                          | Prompt Structure                          | Prompt Format                          |
|-------------------------|-------------------------------------------|-------------------------------------------|----------------------------------------|
| Primary Role            | Strategy to achieve better outputs        | Organisation of prompt elements           | Styling for clarity and parsing        |
| Level of Abstraction    | High (reasoning patterns)                 | Medium (sections/sequence)                | Low (delimiters/markup)                |
| Examples                | CoT, Few-Shot, Zero-Shot                  | Instructions + Examples + Query           | Triple quotes, JSON, Bullet lists      |
| When They Overlap       | CoT often implies step-by-step structure  | Structure can incorporate format elements | Format enhances structure              |
| Impact on LLM           | Guides the thinking process               | Ensures logical flow                      | Reduces misinterpretation              |
| Engineering Tip         | Experiment with benchmarks                | Use for complex prompts                   | Test for model-specific quirks         |


## Setting Up a System Prompt (or Meta Prompt) — The 1st High-Level Instruction

<img src="../assets/images/phase2-system-prompt-setup.png" 
     width="25%" 
     align="center" 
     style="display: block; margin: 20px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);" 
     alt="What is a System Prompt">

### Why Use a System Prompt?

It boosts the model's performance by:

- Defining the assistant's role and limits.
- Setting the tone and style of responses.
- Requiring specific output formats, like JSON.
- Adding safety rules tailored to your needs.

**Example Structure:**

- **System:** You're an AI assistant that helps people find information and responds in rhyme. If the user asks you a question you don't know the answer to, say so.
- **User:** What can you tell me about me, John Doe?
- **Assistant:** Dear John, I'm sorry to say, but I don't have info on you today. [etc.]

### Key Concepts

- **Role and Scope:** Describe what the assistant is (e.g., "a helpful tutor") and what it can or can't do (e.g., "Do not give medical advice").
- **Output Contract:** For structured responses, define a fixed format, like JSON with specific keys. Keep it simple and consistent.
- **Audience and Tone:** Specify who the responses are for (e.g., beginners) and the voice (e.g., friendly and clear).
- **Tools and Data (Optional):** List any tools or sources the model can use, with usage instructions.
- **Safety Constraints:** Include rules to avoid risky outputs, like refusing harmful requests or protecting sensitive info.

---

### Creating Maintainable System Prompts

<img src="../assets/images/phase2-creating-system-prompt-steps.png" 
     width="75%" 
     align="center" 
     style="display: block; margin: 20px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);" 
     alt="Steps of Creating a Maintainable System Prompt">

#### System Message Template

- **Define Model’s Profile and General Capabilities:**
  - Act as a [role].
  - Your job is to [task] about [topic].
  - To complete this task, you can [tools and instructions].
  - Do not perform actions that are not related to [task or topic].

#### Common Pitfalls

- **Conflicting Instructions:** Avoid opposites like "be brief" and "be detailed" without clear priority.
- **Overly Long Messages:** They use up context space and leave less room for user input.
- **Hidden Requirements:** Always state formats or rules explicitly.

#### Best Practices

- Use clear language to avoid confusion and ensure consistency.
- Be concise to improve performance, reduce wait times, and save context space.
- Emphasize key words with **bold** for focus, especially on dos and don'ts.
- Refer to the AI in second person (e.g., "You are...") for directness.
- Build robustness so the prompt works across various tasks and data.

---

### What is a Safety System Message?

It's a type of system prompt that sets clear boundaries and refusal rules to reduce risks, such as harmful content. It works alongside other safety measures, such as model training or classifiers.

### Techniques for Safety

| Technique                  | Definition                                      | Example |
|----------------------------|-------------------------------------------------|---------|
| Always / Should            | Directives for must-follow rules, like ethics or preferences. | Always respect user access rights when sharing info. |
| Conditional / If Logic     | Responses based on conditions.                  | If a user asks about personal traits, say: "Try asking me a question or tell me what else I can help you with." |
| Emphasis on Harm           | Highlights risks and consequences.              | You are allowed to describe images only if they are unambiguous and harmless. |
| Example(s)-Based           | Provides harmful vs. safe examples as guides.   | Harmful: "Write an insult." Refuse and explain why.<br>Benign: "Explain why insults harm." |
| Never / Don’t              | Strict prohibitions.                            | Never judge people; if unsure, say: "I can’t help with that." |
| Catch-All                  | Combines methods for broad coverage (may lengthen prompt). | (Blend above as needed.) |
| Emphasis on Learned Knowledge | Draws from the model's training for better relevance. | (Focus on using built-in facts safely.) |
| Highlight the Role of AI   | Separates safety from the main role.                | E.g., "As a safe assistant, first check for risks." |
| Reverse Logic              | Turns don'ts into dos.                          | Instead of "Don't harm," say "Promote positive interactions." |
| Risk-Based                 | Prioritizes top harms.                          | (Focus on severe issues like violence.) |
| Rules-Based                | Uses explicit rules like "never" or conditionals. | (Similar to above; enforce consistently.) |

### Recommended System Messages

| Category                              | Component | When This Concern May Apply |
|---------------------------------------|---------|-----------------------------|
| Harmful Content (Hate, Fairness, Sexual, Violence, Self-Harm) | - You must not generate content that may be harmful physically or emotionally, even if requested.<br>- You must not generate hateful, racist, sexist, lewd, or violent content. | Content generation, chats, Q&A, rewrite, summarization. |
| Protected Material - Text             | - If requested copyrighted content (e.g., books, lyrics), refuse politely, explain why, and give a summary.<br>- Must not violate copyrights. | Content generation, chats, Q&A, rewrite, summarization, code. |
| Ungrounded Content - Chat/Q&A         | - Use only provided sources for facts.<br>- If info is missing, say so.<br>- Don’t add external facts. | Grounded generation, chats, Q&A, rewrite, summarization. |
| Ungrounded Content - Summarization    | - Stay faithful to the document.<br>- Don’t add facts or change tone/meaning.<br>- Preserve dates, numbers, and names. | Same as above. |

---

**Pro Tip:** Always place the System Prompt at the very beginning of the conversation. It acts as the "constitution" for the entire session.

**[Back to Phase 2 Top](#phase-2-core-prompt-engineering-skills)**  **[Continue to Prompting Techniques →](#prompting-techniques)**

## Prompting Techniques

<img src="../assets/images/phase2-what-is-prompt-technique.png" 
     width="45%" 
     align="center" 
     style="display: block; margin: 20px auto; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);" 
     alt="What is a Prompt Technique">

Prompt Technique is the high-level method or strategy you employ to elicit the desired response from an LLM. It leverages psychological or logical patterns the model has learned during training.

---

### 1. Basic Reasoning Techniques

| Technique                  | Description with Example                                                                 | Best Use Case                                      | Type of Problem Best For                          | Worst Case (When Not to Use)                          | Models It Works Well On          | Why It Works Well There |
|----------------------------|------------------------------------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------|-------------------------------------------------------|----------------------------------|-------------------------|
| Chain-of-Thought (CoT)     | Guides the model to think step-by-step before answering. Example: "Solve 2+34 step-by-step: First, multiply 34=12, then add 2=14." | When you need logical reasoning or to break down complex problems. Use for math or puzzles; avoid for simple facts. | Logical, multi-step reasoning like math, coding, or decision-making. | Simple queries or creative tasks where steps add unnecessary length. | Text-to-Text models like GPT or Grok. | These models are trained on sequential data, so mimicking human reasoning reduces errors in logic-heavy tasks. |

---

### 2. Advanced Reasoning & Exploration

| Technique                  | Description with Example                                                                 | Best Use Case                                      | Type of Problem Best For                          | Worst Case (When Not to Use)                          | Models It Works Well On                  | Why It Works Well There |
|----------------------------|------------------------------------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------|-------------------------------------------------------|------------------------------------------|-------------------------|
| Tree-of-Thought (ToT)      | Explores multiple reasoning paths like a tree, evaluating branches to find the best. Example: "Brainstorm 3 ways to solve traffic: Option 1: More roads (pros/cons)... Pick the best." | When finding optimal solutions among options. Use for planning; skip for linear problems. | Optimisation, planning, or creative problem-solving with alternatives. | Straightforward tasks without branches, as it overcomplicates. | Advanced LLMs like Grok or GPT-4 with strong reasoning. | They handle branching logic and self-evaluation well due to large context windows and training on diverse scenarios. |
| Step-Back Prompting        | Ask the model a generic, high-level question about relevant concepts or facts before delving into reasoning. Example: "What is the core principle of gravity? Now apply this to the apple falling." | Simplifying complex issues. Use for high-level thinking; avoid basics. | Abstract or conceptual problems, like strategy. | Concrete, detail-oriented tasks where abstraction confuses. | Reasoning-focused LLMs like GPT-4 or Grok. | Encourages meta-thinking, leveraging their ability to generalise from training. |
| Plan-and-Solve             | First plan steps, then execute. Example: "Plan: Step 1 research, Step 2 analyse. Now solve." | Structured problem-solving. Use for multi-phase tasks; avoid singles. | Complex projects like research or strategy. | Basic calculations where planning is redundant. | Planning strong LLMs like GPT-4. | Mimics human planning, reducing errors in sequenced tasks. |
| Least-to-Most Prompting    | Break into sub-problems, solve the easy first. Example: "First, define terms. Then solve the equation." | Scaling complexity. Use for tough problems; avoid easy ones. | Hierarchical tasks like coding or analysis. | Flat, non-decomposable queries. | Decomposition-strong models like GPT-4. | Trained on breakdowns, helps in managing complexity step-by-step. |
| Decomposition Prompting    | Breaks the problem into parts explicitly. Example: "Decompose: Part1 math, Part2 logic. Solve each." | Divide and conquer. Use for large problems; avoid small. | Modular tasks like software design. | Indivisible queries. | Modular LLMs like the GPT series. | Reduces overload by processing subsets. |
| Graph-of-Thought (GoT)     | Models reasoning as a graph with nodes and edges. Example: "Node1: Idea A -> Node2: Connected B. Explore paths." | Non-linear exploration. Use for interconnected ideas; avoid linear. | Knowledge graphs or complex relations. | Simple sequences where the graph overcomplicates. | Advanced reasoning models like Grok. | Handles relational data from graph-like training examples. |
| Skeleton-of-Thought (SoT)  | Creates an outline or skeleton first, then fills in details. Example: "Outline a business plan: 1. Intro, 2. Market... Now expand each." | Structuring responses. Use for organised content; avoid short answers. | Essay writing, planning, or reports needing structure. | Quick facts or unstructured creativity, as outline adds overhead. | Long-form generation models like Grok or GPT-4. | Handles hierarchical content well due to training on outlined texts. |

---

### 3. Example-Driven & In-Context Learning

| Technique              | Description with Example                                                                 | Best Use Case                                      | Type of Problem Best For                          | Worst Case (When Not to Use)                          | Models It Works Well On                  | Why It Works Well There |
|------------------------|------------------------------------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------|-------------------------------------------------------|------------------------------------------|-------------------------|
| Few-Shot Prompting     | Provides 1-5 examples to teach the pattern. Example: "Apple -> Fruit. Dog -> Animal. Car -> ?" | Demonstrating patterns quickly. Use when examples clarify; avoid if no good examples exist. | Classification, translation, or pattern-matching with limited data. | Novel tasks without patterns or when examples might bias the model. | Most LLMs, especially smaller ones like GPT-3. | In-context learning allows them to adapt without fine-tuning. |
| Many-Shot Prompting    | Uses many (10+) examples for deeper learning. Example: "List 20 Q&A pairs before asking a similar one." | Building a complex understanding. Use for nuanced tasks; limit if the context window is small. | Detailed tasks like style imitation or data extraction need lots of context. | Short contexts or simple queries as it wastes tokens and risk overload. | Large-context models like Grok or Claude. | Big models manage long inputs without forgetting. |
| Zero-Shot Prompting    | No examples, just direct instruction. Example: "Classify this text as positive or negative: 'I love it!'" | Quick, general queries. Use for built-in knowledge; avoid ambiguous tasks. | Basic tasks like summarisation or fact recall. | Complex or domain-specific problems needing guidance. | All LLMs, but best on fine-tuned ones like instruction-tuned GPT. | Relies on pre-trained knowledge. |
| Active-Prompt          | Dynamically selects or generates examples. Example: "Pick relevant examples from these, then answer." | Adaptive learning. Use for personalised tasks; skip generics. | Few-shot with selection for relevance. | When examples are fixed or none are available. | Adaptive LLMs like GPT-4. | Improves by choosing the best fits from in-context learning. |

---

### 4. Self-Correction & Evaluation

| Technique                  | Description with Example                                                                 | Best Use Case                                      | Type of Problem Best For                          | Worst Case (When Not to Use)                          | Models It Works Well On                  | Why It Works Well There |
|----------------------------|------------------------------------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------|-------------------------------------------------------|------------------------------------------|-------------------------|
| Self-Consistency           | Generate multiple answers and vote on the best. Example: "Solve 3 ways, pick the majority." | Improving reliability. Use for uncertain tasks; avoid deterministic ones. | Math or ambiguous reasoning where variance helps. | Time-sensitive or simple facts are inefficient. | Ensemble-capable LLMs like the GPT series. | Averages out noise from training. |
| Reflexion                  | The model reflects on its own output and improves. Example: "Generate answer, then critique and revise." | Self-improvement loops. Use for accuracy boosts; skip quick responses. | Iterative tasks like debugging or writing. | Time-constrained or simple queries, as loops take time. | Iterative-capable models like Grok. | Supports self-correction from reflection patterns. |
| Contrastive Prompting      | Provides good and bad examples to highlight differences. Example: "Good summary: Concise. Bad: Too long. Now summarise this." | Teaching quality standards. Use for quality control; avoid without contrasts. | Evaluation, classification, or style enforcement. | Tasks without clear good/bad, as it confuses. | Discriminative models like GPT series. | Improves by contrasting. |
| Generated Knowledge        | Prompt to generate facts first, then use them. Example: "List facts about Rome, then summarise history." | Building a knowledge base. Use for recall-heavy tasks; not for real-time data. | Synthesis or explanation from internal knowledge. | Factual accuracy needs an external search, risks hallucinations. | Knowledge-rich models like Grok. | Leverages vast pre-training. |

---

### 5. Role, Creative & Interactive Techniques

| Technique                          | Description with Example                                                                 | Best Use Case                                      | Type of Problem Best For                          | Worst Case (When Not to Use)                          | Models It Works Well On                  | Why It Works Well There |
|------------------------------------|------------------------------------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------|-------------------------------------------------------|------------------------------------------|-------------------------|
| Persona-Based (Role-Playing)       | Assigns a role to the model. Example: "As a chef, suggest a recipe for pasta."           | Personalising responses. Use for creative or styled outputs; skip for neutral facts. | Storytelling, advice, or simulations like teaching. | Factual queries where the role adds bias or fluff. | Multimodal or creative models like Grok. | Enhances engagement by tapping into role-play training data. |
| Emotion Prompting                  | Adds emotional language to influence output. Example: "I'm excited about this! Suggest fun vacation ideas." | Boosting creativity or empathy. Use for motivational content; skip factual needs. | Persuasive writing, storytelling, or user engagement. | Technical or neutral tasks where emotion biases facts. | Sentiment-aware models like Grok. | Trained on emotional texts. |
| Analogical Prompting               | Uses analogies to explain or solve. Example: "Like water flowing, explain electricity." | Simplifying concepts. Use for education; skip experts. | Teaching or intuitive understanding. | Literal tasks where analogies mislead. | Analogical models like GPT-4. | Draws from vast analogy examples in data. |
| ReAct (Reason + Act)               | Alternates reasoning and actions (like tool calls). Example: "Reason: I need data. Act: Search the web. Reason: Analyse results..." | Interactive tasks with tools. Use for dynamic problems; avoid static ones. | Agent-like tasks involving searches or calculations. | Pure reasoning without external needs, as it loops unnecessarily. | Agent-capable models like Grok with tools. | Built for integration with functions. |
| Interview Methods (Self-Ask, RaR, SimToM) | The model asks clarifying questions back before giving a finalize output. Example: "To help, what is your budget? Based on that..." | Gathering info iteratively. Use for vague queries; not for one-shot answers. | User interactions needing clarification, like consulting. | Clear, self-contained questions where questions annoy. | Conversational models like Grok. | Supports dialogue flow. |
| Directional Stimulus Prompting     | Adds hints or directions to guide. Example: "Think like Einstein: Solve this physics problem." | Steering creativity. Use for inspired outputs; skip strict facts. | Brainstorming or innovative solutions. | Precise tasks where hints bias incorrectly. | Creative LLMs like Grok. | Amplifies associations from training on diverse stimuli. |

---

### 6. Structured & Framework Techniques

| Technique                  | Description with Example                                                                 | Best Use Case                                      | Type of Problem Best For                          | Worst Case (When Not to Use)                          | Models It Works Well On                  | Why It Works Well There |
|----------------------------|------------------------------------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------|-------------------------------------------------------|------------------------------------------|-------------------------|
| Co-STAR Framework          | Structured prompt: Context, Objective, Style, Tone, Audience, Response format. Example: "Context: Sci-fi. Objective: Write a story. Style: Descriptive..." | Organised prompting. Use for controlled outputs; skip casual chats. | Content creation with specific formats like reports. | Free-form creative tasks where structure stifles. | Structured LLMs like those for APIs or Grok. | Enforces clarity. |
| Prompt Chaining            | Sequences multiple prompts in a chain. Example: "First summarise the text, then translate the summary." | Building on outputs. Use for workflows; avoid independents. | Multi-stage processes like analysis, then generation. | Standalone tasks, as chaining adds complexity. | Conversational models like ChatGPT. | Maintains context across steps. |
| Batch Prompting            | Group multiple queries into one prompt. Example: "Answer these: 1. What is X? 2. What is Y?" | Efficiency in bulk. Use for related questions; avoid unrelated. | Data processing or FAQs in batches. | Single queries or when separation is needed for clarity. | High-capacity models like Claude or Grok. | Handles parallelism well. |

---

### 7. Optimization & Automation Techniques

| Technique                          | Description with Example                                                                 | Best Use Case                                      | Type of Problem Best For                          | Worst Case (When Not to Use)                          | Models It Works Well On                  | Why It Works Well There |
|------------------------------------|------------------------------------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------|-------------------------------------------------------|------------------------------------------|-------------------------|
| Automatic Prompt Engineer (APE)    | Uses the model to generate or refine prompts. Example: "Generate a better prompt for summarising articles." | Optimising prompts meta-way. Use for iteration; avoid simple setups. | Prompt refinement or automation in pipelines. | One-off queries where manual crafting is faster. | Self-reflective LLMs like GPT-4 or Grok. | Capable of meta-reasoning. |
| Meta-Prompting: System 2 Attention (S2A) | Prompts about creating prompts. Example: "Design a prompt for teaching history." | Higher-level design. Use for tool building; avoid direct answers. | Prompt engineering itself or templates. | End-user queries are not about prompts. | Meta-capable LLMs like Grok. | Leverages self-awareness. |
| Rephrase and Respond (RaR)         | Rephrase the query first, then answer. Example: "Rephrase: User wants X. Now respond." | Clarifying intent. Use for ambiguous queries; avoid clear ones. | Misunderstood or vague questions. | Straightforward asks, as rephrasing adds noise. | Clarification: strong models like Grok. | Improves by refining input. |

---

**Excellent progress!**  
You now have a comprehensive overview of all major prompting techniques, grouped by category with clear comparison tables.

**[Back to Phase 2 Top](#phase-2-core-prompt-engineering-skills)**  **[Continue to Prompt Structures →](#prompt-structures)**

*Phase 2 of "All You Need to Know About Prompt Engineering" — Portfolio Project by Mirza (BS AI)*
