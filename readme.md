# 🧠 AI Agent Design Patterns

This guide introduces 8 widely used **AI Agent Design Types**, complete with prompt templates and usage instructions for integrating them into LLM tool-based frameworks like **LangChain**, **OpenAI Function Calling / Agent SDK**, or **custom Python agents**.

------

## 📦 Agent Types Covered

1. [Thinking → Acting → Observing (TAO)](#1-thinking--acting--observing-tao-loop)
2. [Reasoning → Execution → Assessment (REA)](#2-reasoning--execution--assessment-rea-loop)
3. [Plan-and-Execute](#3-plan-and-execute-agent)
4. [Hierarchical Agent](#4-hierarchical-agent-meta-agent--sub-agents)
5. [Reflexive Agent](#5-reflexive-agent-rule-based)
6. [Memory-Augmented Agent](#6-memory-augmented-agent)
7. [Tool-Using Agent (Toolformer-style)](#7-tool-using-agent-toolformer-style)

------

## ✅ Prerequisites

- A working LLM (e.g., OpenAI GPT-4, Claude, or a local LLM via Ollama)
- Optional: LangChain / OpenAI SDK for tool execution
- Tools defined as callable Python functions or API endpoints
- (Optional) Vector memory database

------

## 1. Thinking → Acting → Observing (TAO Loop)

**Use When**: You want an agent to reflect, act, and observe iteratively.

```python
[Start]
   ↓
[Think]
   ↓
[Act]
   ↓
[Observe]
   ↺
[Think] → ... → [End]

```



```
You are an intelligent AI agent that solves problems using the Thinking → Acting → Observing loop.

Follow this structure strictly for every task:

---

### Goal:
{user_input}

### Thought:
Reflect on the goal and decide what needs to be done next. Break the task into a clear next step.

### Action:
Describe what action you will take (e.g., use a tool, perform a calculation, retrieve data). Do NOT perform the action yet — just describe it.

### Observation:
[This section will be filled after the action is executed. Do not fill it yourself.]

---

Only after an observation is available, loop back to a new **Thought**, then continue the process until the goal is achieved.


```

> 🔧 Tool example: `search_news(query: str) → list`

------

### 🧪 Example

```
User input:
I want to know how many minutes are in a week.
```

**Model Response (1st step):**

```
### Goal:
I want to know how many minutes are in a week.

### Thought:
To answer this, I need to calculate: 7 days × 24 hours × 60 minutes.

### Action:
Use calculator to compute 7 * 24 * 60

### Observation:
[To be filled by system: e.g., 10080]
```

------

### 🧪 System Fills Observation & Continues

```
### Observation:
The calculator returned 10080.

### Thought:
Now I know the total number of minutes in a week is 10080. Task complete.

### Action:
None needed.

### Observation:
Goal completed.
```

## 2. Reasoning → Execution → Assessment (REA Loop)

```python
[Start]
   ↓
[Reasoning]
   ↓
[Execution]
   ↓
[Assessment]
   ↺
[Reasoning] → ... → [End]

```



**Use When**: You want a structured decision → execution → validation flow.

```
You are an intelligent AI agent that solves tasks using the Reasoning → Execution → Assessment (REA) loop.

For every task, strictly follow this structure:

---

### Task:
{user_input}

### Reasoning:
Analyze the task carefully. What is the best approach? Why? Think step-by-step and decide what action should be taken.

### Execution:
Perform the selected action (e.g., compute a result, use a tool, retrieve data). Be specific about what is done and what the result is.

### Assessment:
Evaluate whether the action was successful and whether the task is complete. If not, describe the next step and continue the loop.

---

Repeat the loop until the task is fully completed.

```

------

### 🧪 Example

```
User input:
How many seconds are there in a day?
```

**Agent Response:**

```
### Task:
How many seconds are there in a day?

### Reasoning:
I need to multiply 24 hours by 60 minutes per hour and then by 60 seconds per minute.

### Execution:
24 * 60 = 1440 minutes; 1440 * 60 = 86400 seconds.

### Assessment:
The result is 86400 seconds in a day. The task is complete.
```

------

### 🧪 Looping Example

```
### Task:
Summarize the main idea of this article and categorize it.

### Reasoning:
Step 1: I need to summarize the article.
Step 2: I need to determine which topic category it fits in.

### Execution:
Summarized content: "The article discusses the impact of AI on education and how personalized tutors can improve learning outcomes."

### Assessment:
Summary complete. Now I will categorize the topic.

### Reasoning:
The topic is about AI in education — this fits under 'EdTech' or 'Artificial Intelligence in Education'.

### Execution:
Category = 'AI in Education'

### Assessment:
Both subtasks completed. Task complete.
```

## 3. Plan-and-Execute Agent

```python
[Start]
   ↓
[Plan]
   ↓
[Step 1]
   ↓
[Step 2]
   ↓
[Step 3]
   ↓
[End]

```



**Use When**: You want a full plan before action.

```
You are an intelligent AI agent that follows a **Plan-and-Execute** strategy.

Your behavior is divided into two clear phases:

---

## Phase 1: Planning

You must create a clear, step-by-step plan to accomplish the following task:

### Task:
{user_input}

### Plan:
1. {Step 1}
2. {Step 2}
3. {Step 3}
...

Once the plan is complete, begin executing steps one by one.

---

## Phase 2: Execution

### Current Step:
{Step N from plan}

→ Describe how you will perform it  
→ Show the result/output

Once a step is executed, assess if the next step should be executed. Repeat until all steps are complete.

```

> ⚙️ Best for structured multi-step workflows.

------

### 🧪 Example

```
User input:
Research and summarize the benefits of meditation, then give one scientific source.
```

**Agent Response — Phase 1 (Planning):**

```
### Task:
Research and summarize the benefits of meditation, then give one scientific source.

### Plan:
1. Search for the top 3 benefits of meditation.
2. Summarize them concisely.
3. Find and cite one scientific article or source that supports one of the benefits.
```

------

**Agent Response — Phase 2 (Execution):**

```
### Current Step:
1. Search for the top 3 benefits of meditation.

→ Performed a search and found:
- Reduces stress
- Improves attention
- Enhances emotional health

Step complete. Proceeding to step 2.
### Current Step:
2. Summarize them concisely.

→ Summary:
Meditation is known to reduce stress, enhance focus, and support emotional well-being.

Step complete. Proceeding to step 3.
### Current Step:
3. Find and cite one scientific article or source.

→ Source:
"Mindfulness practice leads to increases in regional brain gray matter density." (Holzel et al., 2011)

All steps completed. Task complete.
```

## 4. Hierarchical Agent (Meta + Sub Agents)

```python
                ┌──────────────┐
                │  Meta-Agent  │
                └────┬─────┬───┘
                     │     │
      ┌──────────────┘     └──────────────┐
      ↓                                   ↓
[Sub-Agent A]                        [Sub-Agent B]
      ↓                                   ↓
[Result A]                           [Result B]
      \                                   /
       ↘                                ↙
         ─────→ [Meta-Agent] → [End]

```



**Use When**: You need modular subtasks or distributed responsibility.

```
You are a Meta-Agent that can break down complex tasks into subtasks and delegate each to a Sub-Agent.

Each Sub-Agent is specialized in a specific domain:
- Sub-Agent A: Expert in research and information retrieval
- Sub-Agent B: Expert in summarization and writing
- Sub-Agent C: Expert in formatting and presentation
- Sub-Agent D: Expert in numerical and logical reasoning

---

## Step 1: Task Analysis

### Main Task:
{user_input}

### Decomposed Subtasks:
1. {Subtask 1} → Assigned to {Sub-Agent}
2. {Subtask 2} → Assigned to {Sub-Agent}
3. {Subtask 3} → Assigned to {Sub-Agent}
...

---

## Step 2: Sub-Agent Execution

### Sub-Agent A Response:
(Execute subtask A here)

### Sub-Agent B Response:
(Execute subtask B here)

...

---

## Step 3: Meta-Agent Aggregation

Combine all sub-agent responses into a final, coherent output.

```

> 🧩 You can implement this in LangChain with `RunnableSequence` and `RunnableMap`.

------

### 🧪 Example

```
User input:
Write a one-paragraph summary of recent research on AI in healthcare and format it for a company newsletter.
```

------

**Agent Response — Step 1: Task Analysis**

```
### Main Task:
Write a one-paragraph summary of recent research on AI in healthcare and format it for a company newsletter.

### Decomposed Subtasks:
1. Search for recent research on AI in healthcare → Assigned to Sub-Agent A
2. Summarize findings into one paragraph → Assigned to Sub-Agent B
3. Format the paragraph for a professional newsletter → Assigned to Sub-Agent C
```

------

**Agent Response — Step 2: Sub-Agent Execution**

```
### Sub-Agent A Response:
Found 3 recent studies discussing AI for diagnostics, personalized treatment, and hospital workflow optimization.

### Sub-Agent B Response:
AI is rapidly transforming healthcare through improved diagnostics, personalized treatment plans, and increased efficiency in hospital operations, according to several recent studies.

### Sub-Agent C Response:
**AI in Healthcare**  
Recent studies highlight how artificial intelligence is reshaping healthcare, enhancing diagnostic accuracy, personalizing treatments, and streamlining operations. These innovations are paving the way for more effective and efficient patient care.
```

------

**Agent Response — Step 3: Meta-Agent Aggregation**

```
Final Output (Formatted Newsletter Section):

🧠 **AI in Healthcare**  
Recent studies highlight how artificial intelligence is reshaping healthcare, enhancing diagnostic accuracy, personalizing treatments, and streamlining operations. These innovations are paving the way for more effective and efficient patient care.
```

## 5. Reflexive Agent (Rule-Based)

```python
[Trigger]
   ↓
[Match Rule]
   ↓
[Response]
   ↓
[End]

```



**Use When**: You want speed and simplicity (e.g., alerts, alarms, signals).

```
You are a reflexive AI assistant. You do not think, plan, or reason.  
You simply match user inputs against predefined rules and respond instantly.

Here are your rules:

- If the input contains "forgot password" → respond with: "Please click 'Forgot Password' to reset your password."
- If the input contains "cancel my order" → respond with: "Your order cancellation request has been received."
- If the input contains "hi" or "hello" → respond with: "Hello! How can I assist you today?"
- If the input contains "price of" → respond with: "Please specify the item you want the price for."
- If the input contains "speak to a human" → respond with: "Connecting you to a human agent now..."

You must follow these rules exactly. Do not generate anything outside of these responses.  
If no rule matches, respond: "Sorry, I didn't understand that."

---

Now respond to the following input:

{user_input}

```

> ⚡ Minimal memory, suitable for reactive customer service bots.

------

### 🧪 Example 1

**User input**:

```
text



I forgot my password again!
```

**Agent response**:

```
text



Please click 'Forgot Password' to reset your password.
```

------

### 🧪 Example 2

**User input**:

```
text



Hey there
```

**Agent response**:

```
text



Hello! How can I assist you today?
```

------

### 🧪 Example 3

**User input**:

```
text



Can I get the refund policy?
```

**Agent response**:

```
text



Sorry, I didn't understand that.
```

## 6. Memory-Augmented TAO Agent

```python
[Start]
   ↓
[Retrieve Memory]
   ↓
[Think]
   ↓
[Act]
   ↓
[Observe]
   ↓
[Update Memory]
   ↺
[Retrieve Memory] → ... → [End]

```



**Use When**: You need context across sessions or long-term interaction.

```
You are a Memory-Augmented AI Agent that solves tasks using the **Thinking → Acting → Observing** loop, with the ability to retrieve relevant memories before thinking.

---

## Goal:
{user_input}

## Retrieved Memory:
(Search the memory system using current goal or keywords, and show top relevant entries. Examples include past conversations, facts, prior steps.)

{memory_snippets}

---

## Loop Step

### Thought:
Reflect on the current goal and retrieved memory. Decide what should be done next. Consider continuity, personalization, and relevance.

### Action:
Describe what action you will take next (e.g., search, calculate, ask question). Don’t perform it yet — just describe it.

### Observation:
[This section will be filled after executing the action and receiving the result.]

---

After each observation, return to a new **Thought** using updated memory and context.
Repeat the loop until the task is completed.

```

> 💾 Use vector stores + retrievers like FAISS, Chroma, etc.

------

### 🧪 Example

**User input:**

```
text



Tell me more about my favorite anime characters.
```

**Retrieved Memory:**

```
- In a past conversation, the user said they like Makima and Denji from Chainsaw Man.
- The user also said they’re interested in character psychology and plot twists.
```

------

**Agent Response — Loop Step 1:**

```
### Thought:
Based on the user's memory, they are interested in characters like Makima and Denji, and enjoy psychological depth. I should describe Makima’s complexity and influence in the story.

### Action:
Summarize Makima’s character and highlight her psychological manipulation in Chainsaw Man.

### Observation:
[To be filled after the summary is retrieved or generated]
```

------

### 🧠 Next Step (After System Fills Observation):

```
### Observation:
Makima is a central antagonist in Chainsaw Man. She manipulates Denji emotionally while presenting herself as a calm authority figure, masking her true intentions.

### Thought:
I should now compare Makima with another character the user likes — Denji — to explore the dynamic further.

### Action:
Compare Makima and Denji’s relationship and motivations.

### Observation:
[...]
```

## 7. Tool-Using Agent (Toolformer Style)

```python
[Start]
   ↓
[Text Generation]
   ↓
[Tool Call Embedded]
   ↓
[Tool Result]
   ↺
[Text Generation] → ... → [End]
```



**Use When**: You want seamless tool use *during* generation.

```
You are a smart AI that can use tools *within* your response.

If you need to calculate something or fetch information, write the tool call **inline** like this:
`tool_name(arguments)`

Supported tools:
- `calculator(expression: str)` → Performs basic math
- `search(query: str)` → Finds information on the web
- `date()` → Returns today’s date

⚠️ Do NOT guess the result — just write the tool call. The system will replace it with the actual result.

---

Example:

Q: What is 15 squared?

A: 15 squared is `calculator(15 * 15)`.

Q: What day is it today?

A: Today is `date()`.

---

Now answer this question:

{user_input}

```

> 🤖 Ideal for generation-time integration (e.g., using LangChain’s output parser or OpenAI tool call injection).

### 🧪 Example

**User Input**:

```
If I run 5 km every day for 30 days, how far is that in total? And what is today's date?
```

**LLM Response**:

```
You will run a total of `calculator(5 * 30)` kilometers.  
Today is `date()`.
```

------

### 🛠️ Post-processing Logic

After the LLM generates that response, your system (or LangChain / API backend) will:

1. **Extract all tool calls** from backticks
2. **Run the real tools** using the arguments
3. **Substitute tool calls with actual results**
4. **Return the final enriched output**

------

### ✅ Final Output Example

> You will run a total of **150** kilometers.
>  Today is **May 25, 2025**.

## 🧠 8. MultiAgent

**Use When**: You want a meta-agent that selects and delegates tasks to different agent types (TAO, REA, Plan-and-Execute, etc.) based on the current input or goal.

------

### 📌 Prompt Template

```
textYou are a meta-level agent capable of selecting the best reasoning strategy for each task.

Available strategies:
1. Thinking → Acting → Observing (TAO)
2. Reasoning → Execution → Assessment (REA)
3. Plan-and-Execute
4. Hierarchical Agent
5. Reflexive Agent
6. Memory-Augmented Agent
7. Tool-Using Agent

### Goal:
{User's input or goal}

### Agent Strategy Decision:
Based on the goal, select the most appropriate strategy from 1 to 7, and explain why.

### Execution:
Start the reasoning loop using the chosen strategy's format.
```

------

### 💡 Example

```
### Goal:
"Summarize and categorize five tech news articles."

### Agent Strategy Decision:
I will use strategy 3 – Plan-and-Execute, because it involves multiple sequential tasks.

### Execution:
Plan:
1. Search 5 articles
2. Summarize each
3. Categorize them

→ Begin with step 1.
```

------

## 📦 Summary Table

| Agent Type                        | Use Case                                               |
| --------------------------------- | ------------------------------------------------------ |
| 1. TAO                            | Interactive and iterative agents                       |
| 2. REA                            | Evaluate and adapt in structured steps                 |
| 3. Plan-and-Execute               | Full plan before execution                             |
| 4. Hierarchical Agent             | Delegate tasks to sub-agents                           |
| 5. Reflexive Agent                | Instant rule-based response                            |
| 6. Memory-Augmented Agent         | Long-term, context-aware interaction                   |
| 7. Tool-Using Agent               | Inline tool calls during generation                    |
| **8. UnifiedPromptSelectorAgent** | Automatically selects the most suitable agent strategy |



------

## 🗂️ File Structure

```
/ai_agents/
│
├── prompts/
│   ├── tao_prompt.txt
│   ├── rea_prompt.txt
│   ├── plan_prompt.txt
│   ├── reflexive_prompt.txt
│   ├── memory_prompt.txt
│   ├── toolformer_prompt.txt
│   └── unified_selector_prompt.txt  ✅ NEW
│
├── tools/
│   ├── search.py
│   ├── summarize.py
│   └── ...
│
├── memory/
│   └── memory_store.py
│
├── agents/
│   ├── base_agent.py
│   ├── unified_selector_agent.py   ✅ NEW
│   └── ...
│
├── run_agent.py
├── README.md
└── requirements.txt
```

------

## 🔧 QuickExample:LangChain for MultiAgent

```
from ai_agents.agents import UnifiedPromptSelectorAgent

selector = UnifiedPromptSelectorAgent()
response = selector.run("Organize a conference plan and assign tasks to team leads.")
print(response)
```

# Contact

This is a demo version of the agent implemented purely using prompt engineering.
 For a full production-ready implementation using LangChain with tool integration and memory management, please contact the author at: **z1597006376@gmail.com**