# ResearchMind — Multi-Agent Research System

ResearchMind (Multi-Agent Research System) is a lightweight Python project that chains together specialized LLM agents to research a topic end-to-end: searching the web, scraping a chosen source, drafting a structured research report, and producing automated critique/feedback.

- Purpose: Quickly generate focused, referenced research reports from recent web content using a small multi-agent pipeline.
- Audience: ML researchers, engineers, or hobbyists who want a prototype multi-agent research pipeline with a Streamlit UI and a small CLI pipeline.

## Stack
- Language(s): Python 3.10+ (primary)
- Framework / runtime: Streamlit (UI) + LangChain agents
- Notable libraries: langchain, langchain-openai / ChatOpenAI, tavily-python (search), BeautifulSoup (scraping)

## Contents / Project layout
```
app.py           # Streamlit web UI: inputs, pipeline orchestration, results & download
agents.py        # Agent and chain definitions: search/reader agent builders, writer & critic chains
tools.py         # Tool implementations used by agents: web_search (Tavily) and scrape_url (requests + BS4)
pipeline.py      # CLI-style pipeline runner (same pipeline logic as UI) and example main()
requirements.txt # Python dependencies
__pycache__/     # Python cache (ignored normally)
.gitignore
```

How it fits together:
- tools.py provides two LangChain-style tools:
  - web_search(query) → uses TavilyClient to produce a short list of titles/URLs/snippets.
  - scrape_url(url) → requests + BeautifulSoup to return cleaned page text (truncated).
- agents.py composes agents and chains:
  - build_search_agent: agent with web_search tool (uses ChatOpenAI).
  - build_reader_agent: agent with scrape_url tool.
  - writer_chain: prompt → LLM to produce full research report (Introduction, Key Findings, Conclusion, Sources).
  - critic_chain: prompt → LLM to evaluate the generated report.
- app.py wires these components into a Streamlit UI with a 4-step pipeline and download button.
- pipeline.py exposes the same pipeline for CLI usage (run_research_pipeline).

## Quickstart — run locally

1) Clone and install
```bash
git clone https://github.com/msncoder/Multi-Agent-Research-System.git
cd Multi-agent-research-system
python -m venv .venv
. .venv/bin/activate   # or .venv\Scripts\activate on Windows
pip install -r requirements.txt
```

2) Create a .env file in the project root with at least:
```
OPENAI_API_KEY=sk-...           # required by langchain_openai / OpenAI
TAVILY_API_KEY=...              # required by tavily-python web search tool
```
(You can also export these env vars directly.)

3) Run the Streamlit UI (recommended for interactive use)
```bash
streamlit run app.py
```
Open the provided local URL in a browser, enter a topic and click "Run Research Pipeline".

4) Or run from the CLI:
```bash
python pipeline.py
# then enter a research topic when prompted
```

## Environment variables / configuration
- OPENAI_API_KEY — API key for OpenAI models (used by langchain_openai.ChatOpenAI)
- TAVILY_API_KEY — API key for Tavily search (used by tools.web_search)
- (Optional) Any other env vars supported by installed libs (e.g., proxies)

Notes:
- The default model configured in agents.py is "gpt-4o-mini". Change the model string in agents.py if you need a different model.
- Web scraping in tools.scrape_url is intentionally simple; it strips script/style/nav/footer and returns the first ~3000 characters.

## Example pipeline flow (high level)
1. Search Agent queries Tavily for the topic and returns top titles/URLs/snippets.
2. Reader Agent picks a URL from search results and scrapes its content.
3. Writer Chain composes a structured markdown research report using both the search output and scraped content.
4. Critic Chain scores and lists strengths/areas for improvement for the report.

## Troubleshooting & tips
- If the Streamlit page shows LLM errors, confirm OPENAI_API_KEY is set and quota/permissions are OK.
- If web_search returns empty results, verify TAVILY_API_KEY and network access.
- To debug scraping issues, add logging around requests.get in tools.py or increase the timeout.
- To reduce API calls while developing, add simple caching around tools.web_search and tools.scrape_url.

## Suggested improvements (ideas)
- Add caching layer (Redis or filesystem) for web_search & scrape_url results to avoid repeated API calls.
- Replace Tavily with an alternative search provider (Google/Bing) or add multiple provider fallbacks.
- Improve the reader agent to select multiple URLs and aggregate content.
- Add tests, CI, and a license file (currently none included).

## Security & cost considerations
- This project calls external APIs (OpenAI, Tavily) that incur costs. Keep API keys private and monitor usage.
- Be mindful when scraping websites — respect robots.txt and site terms of use.

## Contributing
- Open an issue or PR with a clear description of the change.
- Add tests for new functionality where possible.
- If you add a license, include it in the repo root (e.g., LICENSE).

## License
- No license file is present in the repository. Add a LICENSE (e.g., MIT) if you want to allow reuse.

## Try asking
- How can I swap the model used in agents.py from "gpt-4o-mini" to another OpenAI model while preserving prompt behavior?
- Can web_search be extended to return full result metadata (timestamp, domain, language) and have the reader agent choose multiple URLs? (see tools.py and agents.py)
- Where would you add caching or rate-limiting to avoid repeated scraping when running the same topic multiple times? (see tools.py / pipeline.py)