# Askimo + Ubuntu WSL2 research pack

This pack is tailored for Askimo on Windows and the existing WSL2 distribution Ubuntu-22.04

## What is included

- askimo-neuroai-wsl-skills.zip: six Askimo Agent Skills, ready for Settings > Agents & Skills > Import from ZIP.
- ubuntu-wsl-mcp-bridge.zip: an optional stdio MCP server for Askimo Chat. It exposes read-only Ubuntu status, project listing/search, bounded text reads, explicit project-file writes, and public arXiv search.
- skills/: editable source files for all Skills.
- mcp-bridge/: editable MCP server source and dependency list.

The bridge only exposes files below /home/users. It does not provide arbitrary shell execution. It omits hidden folders, .venv, .git, datasets, results, credential-like files, and binary/database files. File writes do not replace an existing file unless overwrite=true is explicitly passed.

## Import the Skills

In Askimo, open Settings > Agents & Skills > Import from ZIP and select askimo-neuroai-wsl-skills.zip. The six Skills should appear in the library. Askimo Agents choose Skills based on the task description.

Skills are instructions, not security controls. Askimo's Agent view launches the selected local CLI runtime with file and terminal access, and current Askimo documentation says its approval prompts are bypassed. Keep the selected workspace trusted and review tool activity.

## Install the Ubuntu bridge for Askimo Chat

1. Extract this pack somewhere on Windows, or extract ubuntu-wsl-mcp-bridge.zip.
2. In PowerShell, create the Linux destination and copy the two bridge files into Ubuntu:

   wsl.exe -d Ubuntu-22.04 --exec mkdir -p /home/users/tools/askimo-wsl-bridge

   Copy-Item -Path '.\mcp-bridge\*' -Destination '\\wsl.localhost\Ubuntu-22.04\home\users\tools\askimo-wsl-bridge' -Recurse -Force

   If you extracted only ubuntu-wsl-mcp-bridge.zip, change the source path to the extracted mcp-bridge folder.

3. In an Ubuntu terminal, install the MCP SDK in an isolated environment:

   cd ~/research/tools/askimo-wsl-bridge
   python3 -m venv .venv
   . .venv/bin/activate
   python -m pip install --upgrade pip
   pip install -r requirements.txt

   If python3 -m venv reports that venv is missing, install Ubuntu's python3-venv package using apt, then repeat. This bridge needs network access once to download the Python dependency.

4. In Askimo, open Settings > MCP Servers > Add MCP Server and enter:
   - Name: Ubuntu Research Bridge
   - Transport: Stdio
   - Command: wsl.exe
   - Arguments: -d Ubuntu-22.04 --exec /home/astraia1031/research/tools/askimo-wsl-bridge/.venv/bin/python /home/users/tools/askimo-wsl-bridge/server.py
   - Working directory: leave blank

   Click Test Connection, then Save. In an Askimo Chat conversation, open the Tools picker and enable Ubuntu Research Bridge. Try the tools ubuntu_status or search_arxiv.

If Askimo does not accept the argument string as entered, put each argument in its own argument row if the dialog provides that interface.

## How to give Askimo Agents access to Ubuntu

Askimo's Agent view launches a separate CLI runtime (Codex CLI, Claude Code, or Antigravity CLI). Its Skills and terminal tools belong to that runtime. Do not assume an MCP server configured under Askimo Settings > MCP Servers is automatically added to the Agent CLI.

For an Agent started by Askimo on Windows, import the WSL Ubuntu Bridge Skill and tell it to use Ubuntu-22.04 through wsl.exe. Example goal:

   Inspect the project under /home/astraia1031/research/projects/my-project using Ubuntu-22.04. Report the current Git status and Python environment. Do not modify files.

For repeatable CLI tool integration, configure MCP servers in the selected Agent CLI's own settings, following that runtime's official documentation. Askimo's normal Chat MCP settings are a separate feature.

## Give Askimo internet access

There are three separate connections:

1. Model connection: cloud providers such as OpenAI or Anthropic need a valid provider/API connection. Local Ollama/LM Studio models can work offline.
2. Askimo normal Chat web search: open Settings > Web Search, enable it, select DuckDuckGo (no API key needed), and test. Then open the Chat Tools picker and enable Web Search for that conversation. Askimo offers searchWeb and readWebPage.
3. Askimo Agent CLI web tools: availability depends on the selected runtime and its own tool/MCP configuration. The Askimo Chat Web Search setting and the Ubuntu bridge do not automatically turn into Agent CLI browser tools. Ask the Agent to search a recent topic and provide source links; if it reports no search tool, configure a web-search MCP server in that CLI's own settings.

The optional bridge's search_arxiv tool makes live requests from Ubuntu to the public arXiv API. It provides paper metadata and abstracts, not general-purpose web search. A direct Ubuntu connectivity check can be run in Ubuntu:

   python3 -c 'import urllib.request; print(urllib.request.urlopen("https://export.arxiv.org/api/query?search_query=all:neuroAI&max_results=1", timeout=15).status)'

## Tools exposed by the Ubuntu bridge

- ubuntu_status: OS, Python, CPU, memory, disk-free, and research-directory status.
- list_research_projects: list files and folders under the projects directory.
- read_project_file: read a bounded text/code file in a project.
- search_project_files: search allowed text/code files, skipping data, results, dependencies, and hidden folders.
- write_project_file: create or explicitly replace an allowed text/code file in a project.
- search_arxiv: query arXiv over HTTPS and return titles, dates, authors, abstracts, and links.

The bridge intentionally does not expose arbitrary shell commands, apt, Docker internals, Windows user directories, Zotero data, Obsidian vaults, datasets, or results. Keep secrets out of project source.

## Official references

- Askimo Agents and Skill format: https://askimo.chat/docs/desktop/agents/
- Askimo Web Search setup: https://askimo.chat/zh/docs/desktop/web-search/
- Askimo MCP setup: https://askimo.chat/docs/desktop/mcp-integration/
- MCP Python SDK: https://github.com/modelcontextprotocol/python-sdk
