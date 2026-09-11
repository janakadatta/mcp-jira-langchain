
# mcp-jira-langchain

This is a POC I put together to see if we can automate part of the coding
workflow. The idea: pull a Jira ticket's acceptance criteria, pull our
backend coding standards from Confluence, and have an AI agent generate a
first draft of the backend code that follows both.

## How it's put together

There's an existing open-source tool called `mcp-atlassian` that already
knows how to talk to Jira and Confluence's APIs - fetch a ticket, fetch a
page, search, etc. Instead of writing that integration myself, I'm just
running that tool and pointing my own script at it. Same approach I used in
the sqlite project - reuse the connector, write the logic on top.

My script (`jira_confluence_codegen.py`) does three things:
1. Starts that connector and asks it "what can you do" (get issue, get page,
   etc.)
2. Hands those abilities to a LangChain agent running on Gemini, along with
   an instruction: go get the ticket's acceptance criteria, go get the
   coding standards page, then write code that satisfies both.
3. Saves whatever code comes back into `generated_code/<ticket>_generated.py`

## What you need before running it

- Python 3.11+ and `uv` (https://docs.astral.sh/uv/)
- A Gemini API key
- A Jira/Confluence Cloud API token (get one at
  id.atlassian.com -> Settings -> Security -> API tokens)
- A Confluence page with our backend coding standards written on it -
  `coding_standards.md` in this repo is a starting draft I put together

## Setting it up

```bash
uv sync
```

Then open `.env` and fill in your real values:

```
GOOGLE_API_KEY=...
JIRA_URL=https://your-site.atlassian.net
JIRA_USERNAME=your.email@example.com
JIRA_API_TOKEN=...
CONFLUENCE_URL=https://your-site.atlassian.net/wiki
CONFLUENCE_USERNAME=your.email@example.com
CONFLUENCE_API_TOKEN=...
```

## Running it

```bash
uv run jira_confluence_codegen.py
```

It'll ask for three things:
- The Jira issue key, like `KAN-1`
- The Confluence page title with the coding standards
- The space key that page lives in

Then it fetches both, generates the code, and drops it in `generated_code/`.

## A few things worth knowing

- I set `READ_ONLY_MODE=true` on purpose so this can't accidentally change
  anything in real Jira/Confluence while I'm still testing it. Take that out
  once write access is actually wanted.
- This whole thing is running on my personal Atlassian account for now,
  since it's just a POC. Before it touches real project data, this needs a
  proper look from whoever handles security/data policy on our end.
- Free-tier Gemini API keys have a pretty low daily request limit - ran
  into that a few times while testing. Worth moving to a paid tier if this
  goes past the POC stage.
