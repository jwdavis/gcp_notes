# Agents

## Links

- [Best practices for MCP tools in production](https://google.github.io/adk-docs/tools/mcp-tools/#production-deployment-checklist)



## ADK - Built-in Search tool (built-in)
1. When using this tool, Google will return Search suggestions, which you must display. The searchEntryPoint provides the URL to render graphic plugs for the suggestions
2. Grounding chunks is a list of sources used for grounding (domain, uri, title)
3. Grounding supports shows which segments of final answer are supported by which grounding chunks (with a confidence score for each supporting chunk)
4. 

## ADK - Running agents
1. The ADK Runtime is the engine that orchestrates and executes your agent and surrounding services (referred to as a Runner)
2. Runtime interacts with an Event Loop
   1. Receives user input (new_message)
   2. Interacts with Session service (create or append to existing)
   3. Starts event generation process by calling agent execution method and awaits emitted events
   4. Runner receives events and takes actions (like updating state) and forwards event (maybe to user)
   5. Signals to agent that this event is handled, allowing to resume and generate next event
   6. Loop continues until no more events for current query
3. Agent/tool/callback interaction with an Event Loop
   1. Runs its logic based on the current InvocationContext, including the session state as it was when execution resumed
   2. When the logic needs to communicate (send a message, call a tool, report a state change), it constructs an Event containing the relevant content and actions, and then yields this event back to the Runner
   3. Agent logic pause on yield and waits for Runner to signal event is processed
   4. Upon resumption, the agent logic can now reliably access the session state (ctx.session.state) reflecting the changes that were committed by the Runner from the previously yielded event.

## ADK Tool types
1. Built-in tools
   1. Google Search
   2. Code Execution
   3. Vertex AI RAG Engine
   4. Verte AI Search
   5. BigQuery
   6. Spanner
2. Function tools
3. 3rd party tools
   1. LangChain wrapper
   2. CrewAI wrapper
4. Cloud tools
   1. Apigee wrapper (toolset)
   2. Application integration wrapper (toolset) (connections, workflows)
   3. Toolbox for databases
5. MCP Tools
6. OpenAPI tools

## ADK - LongRunningFunctionTool
1. Used for tasks intended to run outside of agent workflow
2. Launches long-running activity and returns initial result (like an op id)
3. Can continue or wait until function finishes
4. Agent can query the progress and get intermediate or final results

## ADK - Agent as Tool
1. Basically, can call another agent as a tool
2. Differs from sub-agent, which takes over user responses
3. `tools=[AgentTool(agent=agent_name)]`

## ADK - MCP Tools
1. ADK handles connections to MCP server
2. Lots of examples use stdio and local servers
   1. If invoking npx, probably downloading a javascript server on the fly
   2. Not sure how realistic this is for production; likely networked
3. Lots of examples use SSE, even though MCP has moved to streamableHTTP
   1. ADK supports streamableHTTP
4. 