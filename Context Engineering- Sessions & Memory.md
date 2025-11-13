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
	- started by a new prompt
	- ends when the user leaves the conversation
	- tracks the history of the conversation and the agent's working memory for that interaction
- Memory
	- long term memory
	- consolidated knowledge across sessions