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

As a prompt engineer, inputs for LLMs like Grok include understanding **"prompt technique," "prompt structure,"** and **"prompt format"** — they are related but distinct layers in a well-designed prompt.

<div style="display: flex; align-items: flex-start; gap: 25px; flex-wrap: wrap;">
  <div style="flex: 1; min-width: 300px;">
    Think of them as layers:
  </div>
  <div style="flex: 1; min-width: 280px; text-align: right;">
    <img src="../assets/images/phase2-technique-structure-format.png" width="88%" alt="Technique vs Structure vs Format Layers" style="border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);">
  </div>
</div>

---

## 1. Prompt Technique

**Definition:** The high-level method or strategy you employ to elicit the desired response from an LLM.

**Key Focus:** Why you’re prompting in a certain way and what cognitive process you’re simulating.

**When to Use:** For solving specific problems like reasoning, creativity, or bias mitigation.

**Examples:**
- **Chain-of-Thought (CoT):** "Solve this math problem step by step: What is 15% of 200?"
- **Few-Shot Prompting:** "Translate these: 'Hello' → 'Bonjour'. 'Goodbye' → 'Au revoir'. Now: 'Thank you' →"
- **Role-Playing:** "You are a pirate captain. Describe finding treasure."

**Prompt Engineer Tip:** Techniques are iterative — test with A/B variations.

---

## 2. Prompt Structure

**Definition:** The overall organisation and logical flow of the prompt’s content.

**Key Focus:** Hierarchy and sequencing of elements.

**When to Use:** For multi-step tasks or long prompts.

**Examples:**
- Basic: `[Instructions] + [Context] + [Query]`
- Advanced: Sections like "Task", "Constraints", "Examples", "Input"

**Prompt Engineer Tip:** Always start with clear instructions and end with a direct call-to-action.

---

## 3. Prompt Format

**Definition:** The stylistic and syntactic presentation of the prompt.

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

---

## Setting Up a System Prompt (or Meta Prompt)

<img src="../assets/images/phase2-system-prompt-setup.png" width="42%" align="right" style="margin-left: 25px; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);" alt="System Prompt Setup">

**Why Use a System Prompt?**  
It boosts the model's performance by defining role, tone, output format, and safety rules.

**Example Structure:**
- System: You're an AI assistant that helps people find information and responds in rhyme...
- User: What can you tell about me, John Doe?
- Assistant: Dear John...

**Key Concepts:**
- Role and Scope
- Output Contract
- Audience and Tone
- Tools and Data (Optional)
- Safety Constraints

<br clear="right"/>

---

## Creating Maintainable System Prompts + Best Practices + Safety

**System Message Template**
- Act as a [role].
- Your job is to [task] about [topic].
- ...

**Common Pitfalls**
- Conflicting instructions
- Overly long messages
- Hidden requirements

**Best Practices**
- Use clear language
- Be concise
- Emphasize key words with **bold**
- Refer to the AI in second person ("You are...")

**What is a Safety System Message?**  
It sets clear boundaries and refusal rules.

**Techniques for Safety** and **Recommended System Messages** are shown in the tables below (use the PDF screenshots for visual reference).

---

## Prompting Techniques (1–7)

All 7 categories (Basic Reasoning, Advanced Reasoning, Example-Driven, Self-Correction, Role/Creative, Structured & Framework, Optimization & Automation) are covered with full comparison tables in your original document.

**Use the screenshots from PDF pages 12–19** for the visual tables.

---

## Prompt Structures + Prompting Format + How to Combine Layers

All remaining tables (Prompt Structures, Prompting Format, Combining Technique+Structure+Format, Top 15, Top 10, etc.) are converted to clean Markdown tables in your original content.

**Place the main “Combine Layers” screenshot** using the Denis2054 style if you have it:
```html
<img src="../assets/images/phase2-combining-technique-structure-format.png" width="42%" align="left" style="margin-right: 25px; border-radius: 12px; box-shadow: 0 6px 16px rgba(0,0,0,0.18);" alt="Combining Layers Sandwich">
