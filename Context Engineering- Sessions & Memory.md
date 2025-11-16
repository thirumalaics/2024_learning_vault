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
	- context engineering: address the entire payload, dynamically constructing a state-aware prompt based on the user, conversation history, and external data
		- is like giving the chef mise en place: all the ingredients prepped - high quality, all the tools, dietary needs of the customer
		- has more tailored information that would help in responding better to the current prompt
		- pulls relevant bits from long term memory, fetching docs using RAG and adding the outputs from tools or convo summary that were just used
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
	- ends when the user leaves the conversation
	- tracks the history of the conversation and the agent's working memory for that interaction
- Memory
	- long term memory
	- consolidated knowledge across sessions

f

To enable Large Language Models (LLMs) to remember, learn, and personalize interactions, developers must dynamically assemble and manage information within their context window—a process known as Context Engineering.

These core concepts are summarized in the whitepaper below: 
• Context Engineering: The process of dynamically assembling and managing information within an LLM's context window to enable stateful, intelligent agents. 
• Sessions: The container for an entire conversation with an agent, holding the chronological history of the dialogue and the agent's working memory
• Memory: The mechanism for long-term persistence, capturing and consolidating key information across multiple sessions to provide a continuous and personalized experience for LLM agents.

Outside of their training data, their reasoning and awareness are confined to the information provided within the "context window" of a single API call. This presents a fundamental problem, as AI agents must be equipped with operating instructions identifying what actions can be taken, the evidential and factual data to reason over, and the immediate conversational information that defines the current task. To build stateful, intelligent agents that can remember, learn, and personalize interactions, developers must construct this context for every turn of a conversation. This dynamic assembly and management of information for an LLM is known as Context Engineering.

Context Engineering represents an evolution from traditional Prompt Engineering. Prompt engineering focuses on crafting optimal, often static, system instructions. Conversely, Context Engineering addresses the entire payload, dynamically constructing a state-aware prompt based on the user, conversation history, and external data. It involves strategically selecting, summarizing, and injecting different types of information to maximize relevance while minimizing noise. External systems—such as RAG databases, session stores, and memory managers—manage much of this context. The agent framework must orchestrate these systems to retrieve and assemble context into the final prompt.

1205
