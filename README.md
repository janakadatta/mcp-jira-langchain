# mcp-jira-langchain

POC agent that connects to Jira and Confluence through the Model Context
Protocol (MCP), reads a ticket's acceptance criteria and the team's backend
coding standards, and generates backend code that follows them.

## How it works

1. **MCP server**: this project runs [`mcp-atlassian`](https://github.com/sooperset/mcp-atlassian),
   an existing open-source MCP server, via `uvx`. It exposes Jira and
   Confluence as a set of callable tools (get issue, search issues, get
   page, search pages, etc.) - we don't implement the Jira/Confluence API
   integration ourselves, the same way the reference sqlite-langchain
   project reused a pre-built sqlite MCP server instead of writing one.
2. **Agent**: `jira_confluence_codegen.py` connects to that MCP server,
   loads its tools into a LangChain agent (Gemini as the model), and asks
   the agent to (a) fetch the acceptance criteria for a given Jira issue,
   (b) fetch the coding standards from a given Confluence page, and (c)
   generate backend code that satisfies both.
3. **Output**: generated code is saved under `generated_code/<ISSUE_KEY>_generated.py`.

## Prerequisites

- Python 3.11+ and [`uv`](https://docs.astral.sh/uv/) installed
- A Google Gemini API key
- Jira/Confluence Cloud API token (Settings -> Security -> API tokens at
  id.atlassian.com)
- A Confluence page containing your team's backend coding standards
  (see `coding_standards.md` in this repo for a starting draft)

## Setup

```bash
uv sync
cp .env .env.local   # optional: keep a working copy, .env is git-ignored anyway
```

Fill in `.env` with your real values:

```
GOOGLE_API_KEY=...
JIRA_URL=https://your-company.atlassian.net
JIRA_USERNAME=your.email@company.com
JIRA_API_TOKEN=...
CONFLUENCE_URL=https://your-company.atlassian.net/wiki
CONFLUENCE_USERNAME=your.email@company.com
CONFLUENCE_API_TOKEN=...
```

## Run

```bash
uv run jira_confluence_codegen.py
```

You'll be prompted for:
- A Jira issue key (e.g. `PROJ-123`)
- The Confluence page title holding your coding standards
- The Confluence space key that page lives in

The agent fetches both, generates code, and writes it to `generated_code/`.

## Notes

- `READ_ONLY_MODE=true` is set in the MCP server env by default so this POC
  cannot accidentally create/edit real Jira issues or Confluence pages
  while you're testing. Remove it deliberately once you want write access.
- This is a POC using personal Atlassian credentials on a personal machine,
  per the scope given for this task. Before this touches any real
  production Jira/Confluence project data, loop in whoever owns
  security/data-handling policy at your company.
