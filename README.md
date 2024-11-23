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

`npm i @langchain/community @langchain/core @langchain/langgraph @langchain/anthropic @langchain/openai`

```
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "",
  "dependencies": {
    "@langchain/anthropic": "^0.3.8",
    "@langchain/community": "^0.3.15",
    "@langchain/core": "^0.3.18",
    "@langchain/langgraph": "^0.2.22",
    "@langchain/openai": "^0.3.14"
  }
}

```

### Define the graph

```ts
// agent.ts
import { TavilySearchResults } from "@langchain/community/tools/tavily_search";
import { MessagesAnnotation, StateGraph } from "@langchain/langgraph";
import { ToolNode } from "@langchain/langgraph/prebuilt";
import { ChatOpenAI } from "@langchain/openai";

// Define the tools for the agent to use
const tools = [new TavilySearchResults({ maxResults: 3 })];
const toolNode = new ToolNode(tools);

// Create a model and give it access to the tools
const model = new ChatOpenAI({
  model: "gpt-4o-mini",
  temperature: 0,
}).bindTools(tools);

// Define the function that determines whether to continue or not
function shouldContinue({ messages }: typeof MessagesAnnotation.State) {
  const lastMessage = messages[messages.length - 1];

  // If the LLM makes a tool call, then we route to the "tools" node
  if (lastMessage.additional_kwargs.tool_calls) {
    return "tools";
  }
  // Otherwise, we stop (reply to the user) using the special "__end__" node
  return "__end__";
}

// Define the function that calls the model
async function callModel(state: typeof MessagesAnnotation.State) {
  const response = await model.invoke(state.messages);

  // We return a list, because this will get added to the existing list
  return { messages: [response] };
}

// Define a new graph
const workflow = new StateGraph(MessagesAnnotation)
  .addNode("agent", callModel)
  .addEdge("__start__", "agent") // __start__ is a special name for the entrypoint
  .addNode("tools", toolNode)
  .addEdge("tools", "agent")
  .addConditionalEdges("agent", shouldContinue);

// Finally, we compile it into a LangChain Runnable.
export const graph = workflow.compile();
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
OPENAI_API_KEY="sk-xxxx"
TAVILY_API_KEY=tvly-xxxx
LANGSMITH_API_KEY=lsv2_xxxx

LANGCHAIN_CALLBACKS_BACKGROUND="true"
LANGCHAIN_TRACING_V2="true"
LANGCHAIN_PROJECT="Quickstart: LangGraphJS"
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

## Running using LangGraph Studio

- Open LG Studio into the directory that contains the langgraph.json file

# Deploy to LangGraph Cloud

Initialize a git repo at the root of my-app and push up to GitHub
