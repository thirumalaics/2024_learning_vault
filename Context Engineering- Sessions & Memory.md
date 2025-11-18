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
	- multi-agent systems
		- multiple agents collaborate
			- for efficiency, they must share information
		- agent frameworks handle session history for multi-agent systems using one of two primary approaches: 
			- a shared, unified history where all agents contribute to a single log
			- individual histories where each agent maintains its own perspective
		- the choice between the history management depends on the nature of the task and the desired collaboration style between the agents
		
	- shared unified history model
		- will be perfect for tightly coupled sub-tasks where on agent's output will be used as the input for the next agent
			- multi-step problem solving process
		- even with shared history, a sub-agent might process the log before passing it to the LLM
	- separate, individual history model
		- all intermediate thoughts and tool calls, reasoning steps are kept within the sub-agent and are not shared outside
		- the sub-agents are act like a black box
	- what is the difference between context and history?
		- session history: permanent, unabridged transcript of the entire conversation
		- context: crafted information payload sent to the LLM for a single turn
			- agent constructs this context by selecting only a relevant excerpt from the history or by adding special formatting
	- session history needs to under the law of policies and various security laws
		- PIIs to be redacted before data is persisted
		- TTL: session time to live and the event order must be deterministic
			- as retrieving and processing session history is a intensive task
	- history compaction becomes necessary as the sessions get larger
		- one noob strat is to start with the most recent token as the right end of a sliding window and go back till a threshold of tokens, retaining just this portion of tokens
		- recursive summarization: use the llm. periodically pass an old chunk of the convo history and use Llm to summarize the chunk. The summary replaces the original set of messages
			- computational heavy
			- hence, asynchronous operation triggered after a period of inactivity or number of turns
- Memory
	- long term memory
	- consolidated knowledge across sessions to provide a continuous and personalized information
	- session and memory share a symbiotic relationship
		- sessions are the primary data source for generating memories
		- memories are a key strategy for managing the size of a session
	- memory is a snapshot of extracted, meaningful information from a conversation or data source
	- memory manager: decoupled specialized service which enables multi-agent interoperability
		- uses framework agnostic data structures to store information that can be used across agents built using different frameworks
	- storing and retrieving memories is crucial for building sophisticated agents
		- it is one of aspects separating agents from simple chatbots
	- memories provide following capabilities:
		- personalization
		- context window management
			- what I mentioned above, as conversations become longer, the full history can exceed an LLM's context window
			- memory systems can compact this history by creating summary or extracting facts, preserving context without sending thousands of tokens in every turn
		- data mining and insight
			- analyzing stored memories across many users in an aggregated, privacy preserving way
		- agent self-improvement and adaptation
			- the memory allows the agent to reflect on it's past performance
			- the methods, steps that it used to solve a problem and which were more effective and which were not
	- creating, storing and utilizing memory is a collaborative proves, involving:
		- user: provides the raw data for memories
		- the agent(developer logic): configures how to decide what and when to remember, orchestrating calls to the memory manager
			- in simpler architectures, developer can implement logic such as: always retrieve memory and always generate memory from every convo
			- in sophisticated architectures: the dev may implement memory-as-a-tool, where the agent(via the LLM) decides when memory should be retrieved or generated
		- the agent framework(ADK, langGraph): provides the structure and tools for memory interaction
			- the framework acts as the plumbing
			- defines how the dev's logic can access conversation history and interact with the memory manager
			- defines how to stuff retrieved context into the context window
		- the session storage(agent engine sessions, spanner, redis): stores the turn by turn conversation of the session. the raw context will be ingested into memory manager to generate memories
		- the memory manager: handles the storage, retrieval and compaction of memories
			- mech to store, retrieve mem depend on what provider is used
			- this is the specialized service or component that takes the potential memory identified by the agent and handles its entire lifecycle
	- lifecycle within memory manager
		- extraction: key information distilled from the source data
		- consolidation: curates memories to merge duplicative entities
		- storage: persists the memory to persistent databases
		- retrieval: fetches relevant memories to provide context for new interactions


To enable Large Language Models (LLMs) to remember, learn, and personalize interactions, developers must dynamically assemble and manage information within their context window—a process known as Context Engineering.

These core concepts are summarized in the whitepaper below: 
• Context Engineering: The process of dynamically assembling and managing information within an LLM's context window to enable stateful, intelligent agents. 
• Sessions: The container for an entire conversation with an agent, holding the chronological history of the dialogue and the agent's working memory
• Memory: The mechanism for long-term persistence, capturing and consolidating key information across multiple sessions to provide a continuous and personalized experience for LLM agents.

1732
1807
1636