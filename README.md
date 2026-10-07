# Structured News Summarizer with Gemini 2.0

A Generative AI prototype that uses **Google Gemini 2.0 Flash** to generate structured, concise answers and automatically evaluate the quality of those responses using a rubric-based self-evaluation process.

This project was developed as a **Kaggle competition submission/prototype** exploring few-shot prompting, structured generation, and automated evaluation with Gemini.

---

## Project Overview

Large Language Models can produce useful summaries and answers, but their responses can vary in structure, clarity, relevance, and quality.

This project explores a simple Generative AI pipeline that encourages Gemini to produce answers in a consistent format and then uses Gemini again to evaluate the generated response.

The system uses:

1. **Few-shot prompting** to guide the structure and style of responses.
2. **Gemini 2.0 Flash** to generate answers.
3. **Rubric-based evaluation** to automatically rate generated responses.
4. **Multiple generation attempts** to select the highest-rated response.
5. **Interactive question answering** through a notebook interface.

The original project goal was to develop a **grounded news summarizer with evaluation capabilities**. This version represents a work-in-progress implementation, with grounding and citation functionality still requiring further development.

---

## Objectives

The project aims to demonstrate how Generative AI can be used to:

- Generate concise and structured responses.
- Apply few-shot prompting to control output format.
- Automatically evaluate generated responses.
- Select better responses from multiple generations.
- Build an interactive question-and-answer workflow.
- Explore the foundations of a grounded news summarization system.

---

## Key Features

### 1. Few-Shot Prompting

The model is provided with example question-and-answer pairs before receiving the user's question.

The examples demonstrate the desired response structure:

- **Summary**
- **Key Points**
- **Sources**

This helps guide Gemini toward producing consistent and readable responses.

### 2. Structured Response Generation

For each question, Gemini is instructed to generate an answer using a predefined structure.

Example format:

```text
Summary:
A concise explanation of the topic.

Key Points:
- Important point 1
- Important point 2
- Important point 3

Sources:
[1] Source name
[2] Source name
```

### 3. Automated Evaluation

The generated response is passed to Gemini a second time for evaluation.

The evaluator considers four criteria:

| Criterion             | Description                                                                  |
| --------------------- | ---------------------------------------------------------------------------- |
| Instruction Following | How well the response follows the requested structure and instructions       |
| Groundedness          | Whether the response is appropriately grounded and avoids unsupported claims |
| Conciseness           | Whether the response is clear without unnecessary information                |
| Fluency               | Whether the response is natural, readable, and well-written                  |

Each response receives a score from **1 to 5**:

| Score | Rating    |
| ----: | --------- |
|     5 | Very Good |
|     4 | Good      |
|     3 | Okay      |
|     2 | Poor      |
|     1 | Very Bad  |

### 4. Response Selection

The system can generate and evaluate a response multiple times.

If multiple attempts are made, the implementation keeps the response with the highest evaluation score.

Generation stops early if a response receives a score of **5/5**.

### 5. Interactive Q&A

The notebook provides an interactive interface where users can enter questions and receive a structured Gemini response.

The session continues until the user enters:

```text
exit
```

---

## System Workflow

The overall pipeline can be represented as:

```text
User Question
      │
      ▼
Few-Shot Prompt
      │
      ▼
Gemini 2.0 Flash
      │
      ▼
Structured Response
      │
      ▼
Gemini Evaluation
      │
      ▼
1–5 Quality Score
      │
      ├── Score = 5 → Return Response
      │
      └── Otherwise → Generate Another Response
                         │
                         ▼
                   Select Best Response
```

---

## Technologies Used

- **Python**
- **Google Gemini 2.0 Flash**
- **Google Generative AI API**
- **Kaggle Notebooks**
- **IPython Markdown Display**
- **Python Enum**
- **Kaggle Secrets**

### Main Libraries

```python
import enum
from IPython.display import Markdown, display
import google.generativeai as genai
from kaggle_secrets import UserSecretsClient
```

---

## Prompting Approach

The project uses **few-shot prompting** rather than relying entirely on zero-shot instructions.

The prompt provides examples of how Gemini should answer questions before presenting the actual user query.

For example:

```text
Example 1
Q: What is the greenhouse effect?
A:
Summary:
...

Key Points:
- ...
- ...

Sources:
[1] ...
[2] ...

Now answer this:
Q: {question}
A:
```

This approach provides the model with an explicit example of the desired response format.

---

## Evaluation Approach

The project uses a second Gemini generation step as an automated evaluator.

The evaluator receives:

- The original question
- The generated response
- A defined evaluation rubric

It is then instructed to assign a score from **1 to 5**.

Conceptually:

```text
Question
   +
Generated Answer
   +
Evaluation Rubric
   ↓
Gemini Evaluator
   ↓
Quality Score
```

This creates a simple **generate → evaluate → select** workflow.

---

## Response Generation Logic

The core generation function attempts to improve response quality by generating multiple candidates.

```python
def generate_structured_answer(question, retries=2):
    best_response = None
    best_score = 0

    for _ in range(retries):
        ...
```

For each attempt:

1. A structured prompt is created.
2. Gemini generates a response.
3. The response is sent to the evaluator.
4. The evaluator assigns a score.
5. The response is retained if its score is higher than the current best score.
6. Generation stops if a score of 5 is achieved.

This provides a basic mechanism for **response quality selection**.

---

## API Key Configuration

The project uses Kaggle Secrets to securely access the Gemini API key.

The notebook expects a secret named:

```text
GOOGLE_API_KEY
```

The key is loaded using:

```python
GOOGLE_API_KEY = UserSecretsClient().get_secret("GOOGLE_API_KEY")
genai.configure(api_key=GOOGLE_API_KEY)
```

### Setting up the API Key on Kaggle

1. Open the notebook in Kaggle.
2. Add your Gemini/Google API key to Kaggle Secrets.
3. Name the secret:

```text
GOOGLE_API_KEY
```

4. Run the notebook.

The API key should **not** be hard-coded directly into the notebook.

---

## Example Interaction

A user can enter a question such as:

```text
What caused the 2008 financial crisis?
```

The system generates a structured response containing:

```text
Summary:
...

Key Points:
- ...
- ...
- ...

Sources:
...
```

The response is then evaluated using the project's four criteria and displayed with its resulting score.

---

## Project Status

**Status: Work in Progress / Prototype**

The current implementation successfully demonstrates:

- Gemini-powered response generation
- Few-shot prompting
- Structured outputs
- Automated rubric-based evaluation
- Multiple response attempts
- Basic response selection
- Interactive notebook-based Q&A

However, the original goal of building a fully grounded news summarization system has **not yet been completely implemented**.

In particular, the current version does not yet provide a complete retrieval and verification pipeline for the sources displayed in generated responses.

---

## Limitations

### 1. Grounding Is Not Fully Implemented

Although groundedness is included as an evaluation criterion, the current system does not yet retrieve and verify external sources before generating the answer.

The model may therefore produce information that requires independent verification.

### 2. Source Citations Are Not Verified

The prompt encourages the model to provide sources, but the implementation does not currently verify whether those sources were actually consulted or whether the cited information supports the generated answer.

### 3. LLM-Based Evaluation

The evaluator is also a language model.

This means the evaluation score should be treated as an automated quality signal rather than an objective ground-truth measurement.

### 4. Simple Score Extraction

The current implementation extracts the first standalone digit from the evaluator's response:

```python
score = next(
    (int(s) for s in rating_response.text.split() if s.isdigit()),
    0
)
```

A more robust structured output or JSON-based evaluation format could make the scoring process more reliable.

### 5. Limited Iteration

The default implementation uses two generation attempts:

```python
retries=2
```

Increasing the number of attempts could potentially improve response selection but would also increase API usage and latency.

---

## Future Improvements

Several improvements could move the project closer to the original goal of a grounded news summarization system.

### Retrieval-Augmented Generation (RAG)

Integrate a retrieval component that obtains relevant news articles before generating the summary.

```text
User Question
      ↓
News Search / Retrieval
      ↓
Relevant Articles
      ↓
Gemini
      ↓
Grounded Summary
```

### Source Verification

Add a mechanism to verify that:

- cited sources actually exist;
- sources were retrieved by the system;
- claims in the summary are supported by retrieved sources.

### Structured Evaluation Output

Instead of extracting a score from free-form text, require the evaluator to return structured JSON such as:

```json
{
  "instruction_following": 5,
  "groundedness": 4,
  "conciseness": 5,
  "fluency": 5,
  "overall_score": 5
}
```

This would make automated evaluation more reliable and easier to analyze.

### News Retrieval

Connect the system to a news API, search system, or other trusted information retrieval source.

### Claim-Level Evaluation

Instead of evaluating only the overall response, evaluate individual claims against retrieved sources.

### Evaluation Dataset

Create a fixed set of questions and reference answers/articles to measure the system consistently across different versions.

### Evaluation Dashboard

Track metrics such as:

- Average evaluation score
- Groundedness score
- Instruction-following score
- Response length
- Generation success rate
- API usage

---

## What This Project Demonstrates

This project demonstrates several practical Generative AI concepts:

- **Few-shot prompting**
- **Prompt engineering**
- **Structured generation**
- **LLM-as-a-judge evaluation**
- **Iterative response generation**
- **Quality-based response selection**
- **Gemini API integration**
- **Secure API-key management**
- **Interactive Generative AI applications**

The project also highlights an important distinction between **generating a response that looks grounded** and building a genuinely **grounded system backed by retrieved and verifiable evidence**.

---

## Conclusion

This project provides a foundation for a structured Generative AI news summarization system using Gemini 2.0 Flash.

The current prototype focuses on controlling response structure and experimenting with automated evaluation. The next major development step is to introduce **retrieval and source verification**, transforming the current prompting-based prototype into a more reliable grounded summarization pipeline.

---

## Author

**Ibrahim Yisau**

Generative AI / Machine Learning

GitHub: **Mrtobiss**

---

## Acknowledgement

This project was developed as part of a **Kaggle Generative AI project/competition submission** and explores practical applications of Google's Gemini models for structured generation and automated evaluation.
