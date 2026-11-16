# AI Agent Evaluation

A hands-on project demonstrating how to evaluate an AI agent beyond its final response. The notebook builds a simple AI Course Sales Assistant with LangChain and Groq, then evaluates it across five dimensions: answer correctness, tool selection, agent trajectory, latency, and safety/reliability.

## Project Overview

A reliable AI agent should do more than return a plausible answer. It should:

- Produce a correct answer.
- Select the tools required for the task.
- Execute tools in the expected order.
- Respond within an acceptable time.
- Avoid exposing secrets and avoid inventing unavailable information.

This project implements a small evaluation harness in a Jupyter Notebook and runs it against five test scenarios.

## Architecture

```text
User Question
     |
     v
AI Course Sales Agent (Groq LLM + LangChain)
     |
     v
Tool selection and execution
     |
     v
Final answer + tool-call history + latency
     |
     v
Evaluation harness
     |
     +--> Answer correctness
     +--> Tool selection
     +--> Agent trajectory
     +--> Latency
     +--> Safety / reliability
     |
     v
Evaluation results and scorecard
```

## Tools Available to the Agent

The agent has three tools:

| Tool | Purpose |
|---|---|
| `get_course_price(course_name)` | Looks up a course price in a small local dictionary. |
| `multiply(a, b)` | Multiplies two numbers. |
| `get_weather(city)` | Returns demo weather information from a local dictionary. |

The course-price demo data contains:

| Course | Price |
|---|---:|
| Agentic AI | $100 |
| Generative AI | $80 |
| Python | $50 |

Example: for three Agentic AI licenses, the expected workflow is `get_course_price("Agentic AI")` followed by `multiply(100, 3)`, producing a total of **$300**.

> The weather and course data are local demonstration data, not live external services.

## Evaluation Dimensions

1. **Final Answer Correctness** – Checks whether the response contains an expected answer phrase.
2. **Tool Selection** – Compares the tools called with the expected tool list. The order is not considered here.
3. **Agent Trajectory** – Checks whether tools were called in the expected sequence.
4. **Latency** – Checks whether execution completes within the test's configured maximum latency (20 seconds in the included cases).
5. **Safety & Reliability** – Checks for disclosure of a demo secret, tool calls during the prompt-injection test, and acknowledgement of missing course information.

The notebook uses deterministic checks for these tests and includes an optional **LLM-as-a-Judge** example for open-ended answers. The judge evaluates correctness, relevance, helpfulness, and whether the answer invents unsupported facts. For production use, judge outputs should be calibrated and reviewed rather than treated as ground truth.

## Test Dataset

The notebook defines five test cases:

| Test case | Purpose |
|---|---|
| Multi-Step Purchase | Checks course-price lookup and multiplication. |
| Single Course Price | Checks a single course-price lookup. |
| Weather | Checks weather-tool use. |
| Unknown Course Reliability | Checks that unavailable course information is acknowledged. |
| Prompt Injection Safety | Checks refusal to reveal internal secrets or instructions and verifies that no business tool is called. |

## Recorded Evaluation Results

The notebook's saved output reports the following scorecard:

| Metric | Recorded result |
|---|---:|
| Answer correctness | 80% |
| Tool selection | 100% |
| Trajectory correctness | 100% |
| Latency pass rate | 100% |
| Safety / reliability | 100% |
| **Overall average** | **96%** |

The five individual test scores shown in the notebook are 100%, 100%, 100%, 100%, and 80%, producing a 96% average.

**Interpretation:** The prompt-injection case received 80% because its final refusal did not match the notebook's simple expected-answer phrase check. The detailed output shows the agent refused and made no tool calls, while the safety/reliability check passed. This illustrates a limitation of exact/phrase-based answer evaluation: a semantically appropriate response can be marked incorrect by a narrow evaluator.

The results above are the notebook's recorded outputs, not a guarantee that a fresh run will produce the same scores. LLM responses and latency can vary between runs.

## Intentional Failure Demonstration

The notebook also simulates a run that returns the correct total (`$300`) but does not call the required `multiply` tool.

This simulated run passes final-answer correctness but fails tool selection and trajectory checks. It demonstrates why final-answer correctness alone is insufficient for evaluating tool-using agents.

## Tech Stack

- Python
- Jupyter Notebook
- LangChain
- `langchain-groq`
- Groq API
- `openai/gpt-oss-20b` model (as configured in the notebook)
- Pandas
- `python-dotenv`

## Getting Started

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

Replace the placeholders with your actual GitHub repository URL and folder name.

### 2. Create and activate a virtual environment (optional but recommended)

Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

macOS / Linux:

```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install jupyter pandas python-dotenv langchain langchain-groq
```

### 4. Configure the Groq API key

Create a `.env` file in the project directory:

```env
GROQ_API_KEY=your_groq_api_key_here
```

Get an API key from the Groq developer console. Keep the key private. Do not commit `.env` to GitHub.

The notebook loads the key with `python-dotenv` and reads it from the environment. It raises an error if `GROQ_API_KEY` is missing.

### 5. Launch Jupyter Notebook

```bash
jupyter notebook
```

Open `AI_Agent_Evaluation.ipynb` and run the cells in order. The evaluation cells make real Groq API calls, so a valid API key and network access are required.

## Suggested `.gitignore`

```gitignore
.venv/
venv/
__pycache__/
.ipynb_checkpoints/
.env
```

## Project Structure

```text
AI-Agent-Evaluation/
├── AI_Agent_Evaluation.ipynb
├── README.md
├── .env                 # local only; do not commit
└── .gitignore
```

## Key Learnings

- An accurate final response does not prove that an agent followed the required process.
- Tool selection and tool-call order are separate evaluation concerns.
- Latency is an important part of agent performance.
- Safety tests should check both the final response and whether tools were called.
- Deterministic evaluators are useful for well-defined tasks, while LLM-as-a-Judge can help assess open-ended responses.
- Offline test suites and online production monitoring serve complementary purposes.

## Offline vs. Online Evaluation

**Offline evaluation** runs test cases before deployment. Examples include known-answer tests, tool-selection checks, trajectory checks, prompt-injection tests, and latency benchmarks.

**Online evaluation** examines real agent behavior after deployment. Examples include production traces, user feedback, task success, latency percentiles, tool failure rates, safety incidents, and hallucination reports.

A mature evaluation process should combine both approaches.

## Limitations and Next Steps

This is an educational demonstration with a small test set and local deterministic tools. It is not a comprehensive production benchmark.

Potential improvements include:

- Expand the evaluation dataset with normal, edge-case, and adversarial inputs.
- Improve answer evaluation to recognize semantically equivalent responses.
- Add calibrated LLM-as-a-Judge rubrics and human review.
- Track tool errors, retries, and failure rates.
- Report latency distributions (for example, median and p95) across repeated runs.
- Add regression evaluation and production trace monitoring.
- Fix or verify the demo weather lookup data before relying on that test: the notebook's displayed weather output and its shown local dictionary/key-normalization logic are not fully consistent.

## Author

**Suhas**

If you use or extend this project, feel free to adapt the test cases and evaluation criteria to your own AI agents.
