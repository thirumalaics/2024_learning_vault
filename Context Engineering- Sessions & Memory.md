- LLM's biggest weakness is being stateless
	- every api call is a standalone with no data carried
	- model has 0 memory of the last turn
	- to have statefulness we have to build the complex info package every single time: context engineering 
	- prompt engineering: static system message of sorts
		- like giving a chef a recipe
- Context Engineering
	- dynamically grabbing and managing all the relevant info within the LLM's context window every time we talk to it
	- context rot: as the conversation continues, noise might increase
		- model's ability to concentrate and reason sharply degrades
		- context engineering fights this using dynamic history mutation
			- summarization, selective pruning or other compaction strategy
			- to preserve vital information while managing the overall token count
	- context engineering: address the entire payload, dynamically constructing a state-aware prompt based on the ***evidential or factual information***, ***context to guide reasoning*** and ***immediate conversational information***
		- is like giving the chef mise en place: all the ingredients prepped - high quality, all the tools, dietary needs of the customer
		- has more tailored information that would help in responding better to the current prompt
		- pulls relevant bits from long term memory, fetching docs using RAG and adding the outputs from tools or convo summary that were just used
	- context to guide reasoning:
		- system instructions that sets the personal and constraints
		- tool schema information that provides the information on what tools are available
		- few shot examples, curated examples that guide the model's reasoning process
	- evidential and factual data
		- long term memory for the user
		- factual data retrieved using RAGs
		- Tool outputs
		- sub-agent outputs: the output from an agent to which a task was delegated
		- artifacts: non textual data associated with user or session
	- immediate conversational information:
		- conversation history
		- state/scratchpad: temporary, in-progress information or calculations the agent uses for its immediate reasoning process
		- user's prompt that need to be addressed
	- the data that is retrieved from long term memory or using RAGs are results of search for relevant information relating to the query
	- fight against context rot has to be continuous: context management
	- steps in context management: 
		1. fetch context: context is retrieved
			- involves user memories, rag documents and recent conversation events
			- looks like to determine what to retrieve, the user's query and other metadata are used
		2. Prepare context: the agent framework dynamically constructs the full prompt for the LLM call
			- this is a blocking process, agent cannot proceed until the context is ready, even if API calls are asynchronous
		3. Invoke the LLM and Tools: the agent iteratively calls the LLM and any necessary tool until a final response for the user is generated
			- tool and model output is appended to the context
		4. Upload Context: new information gathered during the turn is uploaded to persistent storage
			- bg process, allowing the agent to complete execution while memory consolidation or other post processing occurs asynch
- Session 
	- encapsulates the immediate dialogue history and working memory for single, continuous conversation
	- a user can have multiple sessions but each session functions as a distinct, disconnected log of a specific interaction
	- every session contains two key components: events(chronological history) and state(agent's working memory)
	- common types of events include:
		- user input
		- agent response
		- tool call
		- tool output
	- state - structured working memory or scratchpad
		- holds temporary, structured data relevant to the current conversation
		- as the conversation progresses, it may mutate the state based on logic in the agent
	- tracks the history of the conversation and the agent's working memory for that interaction
	- different frameworks handles sessions differently
		- ADK uses an explicit Session object that contains a list of Event objects and a separate state object
			- the session is like a filing cabinet, with one folder for the conversation history(events) and another for working memory(state)
		- LangGraph does not have a formal session object
			- state is the session
			- state object holds the conversation history and all other working data
			- unlike the append only log of a traditional session, langgraph state is mutable
			- history compaction can alter the record inplace
			- useful for managing long convos and token limits
	- session for multi-agent systems
		- multiple agents collaborate
			- for efficiency, they must share information
		- history can be shared or separate
		- shared: all agents read an write to a central log
			- clutter easily
		- separate: black boxes that communication using tool like calls
			- loose rich shared context
- Memory
	- long term memory
	- consolidated knowledge across sessions



To enable Large Language Models (LLMs) to remember, learn, and personalize interactions, developers must dynamically assemble and manage information within their context window—a process known as Context Engineering.

These core concepts are summarized in the whitepaper below: 
• Context Engineering: The process of dynamically assembling and managing information within an LLM's context window to enable stateful, intelligent agents. 
• Sessions: The container for an entire conversation with an agent, holding the chronological history of the dialogue and the agent's working memory
• Memory: The mechanism for long-term persistence, capturing and consolidating key information across multiple sessions to provide a continuous and personalized experience for LLM agents.

1530
