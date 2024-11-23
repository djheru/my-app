# LangGraph

## My First LG

`mkdir my-app`

```
my-app/
|-- agent.ts            # code for your LangGraph agent
|-- package.json        # Javascript packages required for your graph
|-- langgraph.json      # configuration file for LangGraph
|-- .env                # environment files with API keys
```

### Specify the dependencies

`npm i @langchain/community @langchain/core @langchain/langgraph @langchain/anthropic`

```
{
  "name": "my-app",
  "packageManager": "yarn@1.22.22",
  "dependencies": {
    "@langchain/community": "^0.3.11",
    "@langchain/core": "^0.3.16",
    "@langchain/langgraph": "0.2.18",
    "@langchain/anthropic": "^0.3.7"
  }
}
```

### Define the graph

```ts
// agent.ts
import { ChatAnthropic } from "@langchain/anthropic";
import { TavilySearchResults } from "@langchain/community/tools/tavily_search";
import { createReactAgent } from "@langchain/langgraph/prebuilt";

const model = new ChatAnthropic({
  model: "claude-3-5-sonnet-20240620",
});

const tools = [new TavilySearchResults({ maxResults: 3 })];

// compiled graph
export const graph = createReactAgent({ llm: model, tools });
```

### LangGraph configuration file

```
{
  "node_version": "20",
  "dockerfile_lines": [],
  "dependencies": ["."],
  "graphs": {
    "agent": "./agent.ts:graph"
  },
  "env": ".env"
}
```

### Dotenv file

```
ANTHROPIC_API_KEY=sk-ant-xxxx
TAVILY_API_KEY=tvly-xxxx
LANGSMITH_API_KEY=lsv2_xxxx
```

## Running it using local server

### Install LangGraph CLI

Create virtual env

`cd .. && python3 -m venv .venv && source .venv/bin/activate && cd my-app`

Install LangGraph CLI

`pip install langgraph-cli`

### Start Server

`langgraph up`

Output:

```
(.venv) (base)  ✘  AWS: dev  ~/Workspace/_sandbox/langchain/my-app   main ±  langgraph up                                                                                                                                                                                               <aws:dev> <region:us-east-1>
Starting LangGraph API server...
For local dev, requires env var LANGSMITH_API_KEY with access to LangGraph Cloud closed beta.
For production use, requires a license key in env var LANGGRAPH_CLOUD_LICENSE_KEY.
Ready!
- API: http://localhost:8123
- Docs: http://localhost:8123/docs
- LangGraph Studio: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:8123
```

### CURL check

`curl --request GET --url http://localhost:8123/ok`

Output:

`{ "ok": true }
