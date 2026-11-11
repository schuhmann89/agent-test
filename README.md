# Forschungsassistent

Ein [Managed Agent](https://docs.mistral.ai/studio/agents/overview) auf Basis des Mistral Agents
SDK (`mistralai-agents`). Er recherchiert ein Thema \u00fcber angebundene Recherche-Connector,
verifiziert zentrale Aussagen an mehreren unabh\u00e4ngigen Quellen und antwortet mit Titel und URL
jeder Quelle.

- Modell: `mistral-large-4`, `reasoning_effort="high"`, `max_iterations=25`
- Connector: `deepwiki_5c66` (DeepWiki-Recherche \u00fcber GitHub-Repos: `ask_wiki_question`,
  `read_wiki_contents`, `read_wiki_structure`) sowie `web_search` (Websuche; muss im Workspace
  bzw. der Organisation als Connector verf\u00fcgbar sein, sonst meldet die Session eine Notice und
  f\u00e4hrt ohne ihn fort)
- Agent-Name: `forschungsassistent` (eindeutig pro Workspace)

## Voraussetzungen

- [uv](https://docs.astral.sh/uv/getting-started/installation/) und Python 3.12\u20133.14
- Einen Mistral API Key: `cp .env.example .env` und `MISTRAL_API_KEY` eintragen
  ([API Keys in Mistral Studio](https://console.mistral.ai/api-keys))

## Lokal ausf\u00fchren

```bash
uv sync
uv run --env-file .env python main.py
```

Im Terminal chatten (jede Tool-Call wird angezeigt):

```bash
uv run --env-file .env agents chat
```

Konfiguration pr\u00fcfen, die die CLI l\u00e4dt:

```bash
uv run agents info
```

## In Mistral Studio deployen (Mistral-managed)

```bash
git add --all && git commit -m "Add forschungsassistent" && git push
uv run --env-file .env agents deploy
```

`agents deploy` baut den gepushten Commit mit dem `Dockerfile` in diesem Repo und registriert den
Agenten unter dem Namen `forschungsassistent`. Danach:

```bash
uv run --env-file .env agents chat forschungsassistent   # mit dem deployen Agent chatten
uv run --env-file .env agents logs forschungsassistent --follow
uv run --env-file .env agents list
```

Voraussetzungen f\u00fcr das Deploy: GitHub-Connector in Mistral Studio verbunden, und ein API Key
dessen User eine Workspace-Rolle mit Workflows-Zugriff hat (z. B. Developer).

F\u00fcr einen vollst\u00e4ndig gepinnten Build: `uv lock` lokal ausf\u00fchren, die `uv.lock` committen und im
`Dockerfile` `uv sync --frozen --no-dev` verwenden (so macht es das von `agents deploy`
generierte Dockerfile).

## Aus der eigenen App aufrufen

```python
import asyncio

from mistralai import agents


async def main() -> None:
    agent = agents.Agent.load_from(agent_name="forschungsassistent")
    result = await agent.run("Recherchiere: ...")
    if isinstance(result, agents.Completed):
        print(result.text)


asyncio.run(main())
```

## Weitere Quellen anbinden

In `main.py` weitere Connector erg\u00e4nzen, z. B. allgemeine Websuche \u00fcber einen eigenen MCP
(z. B. Tavily, in Mistral Studio unter Connectors als Custom MCP hinzugef\u00fcgt) oder
wissenschaftliche Literatur:

```python
connectors=[
    agents.connector("web_search"),
    agents.connector("deepwiki_5c66"),
    agents.connector("pubmed"),
]
```

Ein Connector, den der jeweilige Session-User nicht verbunden hat, l\u00f6st einen Sign-in aus; nicht
verf\u00fcgbare Connector meldet der Agent als Notice, ohne den Turn abzubrechen.

## Doku

- [Managed Agents \u00dcbersicht](https://docs.mistral.ai/studio/agents/overview)
- [Quickstart](https://docs.mistral.ai/studio/agents/getting-started/quickstart)
- [Connectors & MCP](https://docs.mistral.ai/studio/agents/learn/agent/connectors)
- [Deploy your agent](https://docs.mistral.ai/studio/agents/getting-started/deploy-it)
