Agent anatomy is broken down to three core parts:
1. Model: takes care of reasoning, key function as part of an agent is to manage the context window
	- constantly deciding what is important right now as the information keeps coming either from the user or the tool use
	- model decides which tool to use
2. orchestration layer: like a conductor, handles memory and state. calls tools
	- the result of the tool execution gets fed back to the model
	- governs the entire process of think, act and observe
	- Manages the operational loop: planning, keeping track of the memory or state, executing the reasoning strat
3. tools: interface with the real world, means for action
 
- think act observe
For a simple task, the following are the typical stages:
1. Get mission
2. scan the scene: for tools, relevant info
3. think it through
4. take action based on the plan formulated as part of step 3
	1. use orchestration layer to call the tool
5. observe and iterate
	1. add the context acquired and go back to step 3

there are different levels of agent capabilities
level 0: just the LLM on it's own without any tool
level 1: llm connected with tools
level 2:strategic problem solver which can curate the context that is given as input to the next stage(context engineering)
	- moves on from single simple tasks to complex tasks which will be broken down to stages
	- takes the output of a tool and uses it to craft a very focused query for another tool
level 3:collaborative multi agent system
	- each specializing in a domain. the agent to which a task has been provided, can break down the task and can call on other specialized agents for help to complete the task as a whole
	- is not same as calling an api, as here the autonomy of the agent is called to solve a sub task
	- the solution is not static, there is reasoning involved
level 4: self evolving, self aware system 
	- identifies and knows it's gaps and remediates them. 
	- when a task is given and for part of the task if there are no sub-agents that it can call on, it can use tools to create one
	- adapting and expanding it's own toolkit

HOW TO MAKE AGENTS WORK RELIABLY IN PROD?
- starts with model selection
	- bigger the model does not necessarily translate to better
	- we require models that can reason well and can perform tool select appropriate for the domain it will be deployed in
	- since the model selection is a crucial step, and in order to have options, we have model routing
		- sending different tasks to different model
		- routing can be based on the type of task, something complex can be sent to flagship model, while menial tasks such as summarizing text can be sent to cheap models
- tools
	- retrieval : we use RAG with vector databases for searching unstructured documents and other tools to query databases for structured info
	- action is done through api wrapped as tools
	- for the model to decide which tool to call, the functions should have proper documentation that will describe the function in depth and also about what it returns  
	- open api spec
- orch
	- responsible for defining agent's persona and it's operating rules
		- done via a system prompt or constitution
		- ex: agent built for a financial agency might be advised not give any financial advice
	- manage memory: short term, long term
	- short term: agent's scratch pad for the task at hand
		- running history of action observation pairs within the current session
	- long term: users preferences, what it learnt from previous tasks across sessions
		- architecturally implemented as another tool usually a rag system talking to vector database where memories are stored ang retrieved