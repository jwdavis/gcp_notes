# AI/ML

## Links

## Search tool (built-in)

1. When using this tool, Google will return Search suggestions, which you must display. The searchEntryPoint provides the URL to render graphic plugs for the suggestions
2. Grounding chunks is a list of sources used for grounding (domain, uri, title)
3. Grounding supports shows which segments of final answer are supported by which grounding chunks (with a confidence score for each supporting chunk)
4. 

## Running agents

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
   5. 