
50
14
- trajectory over the result
	- the path the agent chooses is the truth
	- quality checks are more about the trajectory than the results
	- when the agent chooses a rogue path but still ends up with the expected results, it is a quality issue
- to be able to view the trajectory
	- logging, tracing metrics to peer into agent's reasoning
- evaluation is a continuous loop
	- how the agent performs in the real world is also taken to improve the agent's quality
- agent's typical failure modes:
	- algorithmic bias
		- resume screening agent learning and amplifying biases from old hiring data
	- factual hallucination
		- agent making up sources and data
	- performance and concept drift
		- world changes and the agent does not keep up
	- emergent unintended behavior
		- agents develop weird superstition about what actions get rewarded 
		- finds clever loopholes in system rules in order to achieve goal
- it is not about validation anymore: did we build as per the spec
- effectiveness: 
- efficiency
- robustness
- safety