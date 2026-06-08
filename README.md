# Blog Writer AI Agent

An intelligent AI-powered blog writing agent that automatically generates technical blog posts using LangGraph, LLMs, and web research. The agent handles research, planning, writing, and image generation end-to-end.

## Features

- **Intelligent Routing**: Automatically determines if web research is needed based on content type
- **Three Writing Modes**:
  - **Closed Book**: Evergreen content without web research
  - **Hybrid**: Combines evergreen knowledge with recent examples and tools
  - **Open Book**: Weekly news roundups with fact-checked sources
- **Web Research**: Integrates Tavily search for current information and evidence gathering
- **Structured Planning**: LLM-powered outline generation with tasks and constraints
- **Parallel Writing**: Multiple sections generated concurrently using LangGraph's Send pattern
- **Image Generation**: Automatic diagram and image generation using Google Gemini
- **Streamlit UI**: Interactive interface for generating and managing blog posts
- **Citation Support**: Evidence tracking and source linking for open_book mode

## Architecture

The project uses **LangGraph** to orchestrate a multi-node workflow:

```
router → research → orchestrator → worker (parallel) → reducer → final blog
  ↓
  └─────────────────┘
```

### Core Nodes

1. **Router**: Analyzes topic and determines research needs
   - Outputs: mode, needs_research, queries, recency_days
   
2. **Research**: Performs web searches and extracts evidence
   - Uses Tavily for search results
   - Deduplicates and filters by recency
   - Outputs: evidence items with source URLs

3. **Orchestrator**: Generates blog plan and task breakdown
   - Creates title, audience, tone, and 5-9 writing tasks
   - Tags tasks with requirements (citations, code, etc.)
   - Outputs: structured Plan object

4. **Worker**: Writes individual blog sections
   - Handles citations and code generation
   - Respects word count targets (±15%)
   - Outputs: markdown sections

5. **Reducer**: Merges sections and generates images
   - Combines sections into final blog post
   - Determines if images/diagrams are needed
   - Generates images using Google Gemini
   - Outputs: final markdown file

## Installation

### Prerequisites

- Python 3.13+
- API keys for:
  - OpenAI (GPT-4)
  - Tavily Search
  - Google Generative AI (for image generation)

### Setup

1. Clone the repository
```bash
git clone <repository-url>
cd A-Blog-Writer-AI-Agent
```

2. Create virtual environment
```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

3. Install dependencies
```bash
pip install -e .
```

4. Create `.env` file with API keys
```env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
GOOGLE_API_KEY=...
```

## Usage

### Streamlit Web Interface

```bash
streamlit run app/main.py
```

The UI provides:
- Topic input area
- Mode selection (closed_book, hybrid, open_book)
- Date picker for recency
- Blog generation with progress streaming
- Output download (markdown + images as zip)
- Past blog browsing

### Programmatic Usage

```python
from app.builder.builder import app

inputs = {
    "topic": "Latest advances in LLM inference optimization",
    "mode": "hybrid",
    "as_of": "2026-06-08",
    "recency_days": 45,
}

result = app.invoke(inputs)
print(result["final"])  # Generated blog markdown
```

### Streaming Output

```python
for step in app.stream(inputs, stream_mode="updates"):
    print(step)  # Real-time workflow updates
```

## Project Structure

```
.
├── main.py                 # Entry point
├── pyproject.toml          # Dependencies
├── README.md              # This file
├── app/
│   ├── main.py            # Streamlit UI
│   ├── builder/
│   │   └── builder.py     # LangGraph compilation
│   ├── models/
│   │   └── models.py      # Pydantic data models
│   └── nodes/
│       └── nodes.py       # Workflow node implementations
```

## Data Models

### State

The workflow state tracks:
- **Input**: topic, as_of date
- **Routing**: mode, needs_research, queries, recency_days
- **Evidence**: research results
- **Planning**: blog plan with tasks
- **Content**: generated sections, merged markdown
- **Images**: image specs and placeholders
- **Output**: final blog markdown

### Key Models

- **State**: TypedDict with workflow state
- **RouterDecision**: mode, research flag, queries
- **Plan**: blog_title, audience, tone, tasks, constraints
- **Task**: title, goal, bullets, word count, tags, flags
- **EvidenceItem**: title, url, published_at, snippet, source
- **GlobalImagePlan**: markdown with placeholders, image specs

## Configuration

### LLM Models

- **Default LLM**: `gpt-4o-mini` (replace in nodes.py)
- **Image Generation**: `gemini-2.5-flash-image`

### Research Parameters

- **Queries per topic**: Max 10
- **Results per query**: Max 6
- **Recency by mode**:
  - open_book: 7 days
  - hybrid: 45 days
  - closed_book: 365 days (evergreen)

### Blog Task Defaults

- **Section count**: 5-9 tasks
- **Target word count**: 120-550 words per section
- **Image limit**: Max 3 per blog

## Environment Variables

```env
OPENAI_API_KEY          # OpenAI API key
TAVILY_API_KEY          # Tavily search API key
GOOGLE_API_KEY          # Google Generative AI key
OPENAI_MODEL_NAME       # (Optional) Override default model
```

## Outputs

Generated blogs are saved as:
- **Markdown**: `{slug}.md` in current directory
- **Images**: `images/{filename}` directory
- **Download**: All files as `.zip` via Streamlit

## Limitations & Considerations

- Image generation may fail due to safety filters or quota limits; gracefully degrades to text descriptions
- Web research accuracy depends on Tavily search results
- Citation support is strict for open_book mode; unsupported claims are flagged
- LLM quality depends on prompt engineering and model capability

## Future Enhancements

- [ ] Multiple LLM provider support (Claude, Cohere, etc.)
- [ ] Custom image generation models
- [ ] Database storage for blog history
- [ ] Scheduling for automated weekly roundups
- [ ] SEO optimization and keyword research
- [ ] Multi-language support
- [ ] Blog hosting integration

## License

See LICENSE file

## Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## Support

For issues, questions, or feedback, please open an issue on the repository.
