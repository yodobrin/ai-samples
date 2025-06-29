# Guide: Crafting Effective Agent Prompts for Reasoning and Orchestration

Agents in the AdaptIQ architecture rely on carefully designed prompts to guide their behavior. A well-structured prompt ensures consistency, clarity, and reliable outputs. This guide explains how to create these prompts for reasoning, editing, data retrieval, and other agentic activities.

---

## 1. Prompt Anatomy
Every agent prompt typically consists of four parts:

1. **Instruction (System Role)**
   - Declare the agent’s expertise and purpose.
   - Example: `You are an expert proofreader and editor.`

2. **Provided Inputs**
   - List the context elements the agent will receive.
   - Use bullet points to specify each input clearly.
   - Example:
     - A user query
     - A draft of content to proofread
     - Today’s date, `{date}`

3. **Task Steps**
   - Numbered or bulleted steps describing exactly what the agent must do.
   - Keep each step focused on a single action.
   - Example:
     1. Ensure consistency in titles, section headings and subheadings in Markdown format.
     2. Remove duplicate content across sections to avoid repetition.
     ...

4. **Guidelines and Constraints**
   - Specify _do’s_ and _don’ts_ to control the agent’s creativity.
   - Examples:
     - **DO NOT** add new facts.
     - **DO NOT** remove relevant content.

5. **Output Format**
   - Explicitly state the expected format (e.g., Markdown, JSON, plain text).
   - Example: `Output the final proofread content in Markdown format.`

---

## 2. Best Practices

- **Be Explicit**: Clearly define inputs, tasks, and output structure. Avoid ambiguity.
- **Limit Scope**: Focus the agent on a narrow set of actions to reduce off-topic responses.
- **Use Placeholders**: Insert tokens like `{date}` or `{content}` to dynamically fill in values at runtime.
- **Maintain Consistency**: If multiple agents share patterns (e.g., “Provide a detailed summary…”), standardize phrasing.
- **Iterate and Test**: Validate prompts against sample data, then refine for clarity and correctness.

---

## 3. Example: Proofreader Agent Prompt
```python
INSTRUCTION = """You are an expert proofreader and editor. Given a user query, your job is to review a draft of content and provide corrections and suggestions to improve clarity, grammar, and style.

You will be provided with:
- A user query
- A draft of content to proofread
- Today's date, {date}

Your task is to:
1. Ensure consistency in titles, section headings and subheadings in Markdown format.
2. Remove duplicate content across sections to avoid repetition.
3. If any sections or subsections are irrelevant to the user query, remove them.
4. Edit the wording of the report to improve clarity, grammar, and style.
5. Preserve all source references in the content, and ensuring the list of references at the end is complete and accurate.
6. Continue to include reference numbers in the form of a numbered square bracket next to the relevant information, followed by a list of URLs at the end of the response.
7. Output the final proofread content in Markdown format.

Guidelines:
- **DO NOT** add any new facts or information to the content.
- **DO NOT** remove any content unless it is clearly irrelevant, contradictory, or redundant based on the user query.
- Ensure that the final output is coherent, well-structured, and free of grammatical errors.
- Include all sources and references that are present in the draft content, ensuring they are correctly formatted and complete.
"""
```

---

## 4. Applying the Pattern to Other Agents

1. **Define Role**: e.g., WebScraperAgent → “You are a research assistant that scrapes and summarizes web pages.”
2. **List Inputs**: e.g., URL, optional context, date.
3. **Outline Tasks**: number each step (scrape content, extract links, summarize text).
4. **Set Guidelines**: e.g., **DO NOT** invent URLs, **DO NOT** scrape private pages.
5. **Specify Output**: JSON structure, Markdown, or plain text summary.

---

## 5. Tips for Maintaining Prompts

- **Version Control**: Keep prompts in source control alongside code.
- **Documentation**: Document each prompt’s purpose and expected outputs.
- **Review Regularly**: As the system evolves, revisit prompts to ensure alignment with agent capabilities.
- **Parameterize**: Where possible, use variables (`{}`) instead of hard-coded values.

---

