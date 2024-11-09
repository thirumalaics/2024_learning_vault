- challenge that businesses face today is understanding and using their data in such a way that impacts their business and ultimately their bottom line
- data
	- raw, unprocessed facts or measurements or values collected from various sources
	- lacks inherent context or structure 
- data analysis is the process of identifying, cleaning, transforming and modeling data to discover meaningful and useful information
- data is then crafted into a story through reports for analysis
- data driven businesses make decisions based on the story that their data tells
- data analysis can help an organization by helping to determine:
	- customer sentiment
	- market and product research
	- identifying trends or other data insights
- systematic computational analysis of data
- there are different types of analytics
	- descriptive
		- helps answer questions about what has happened based on historical data
		- summarizes large amounts of data to describe outcomes to stakeholders
		- KPI are developed, to track the status of key objectives
			- ex: ROI
	- diagnostic
		- helps answer why events happened
		- supplement basic descriptive analytics
		- generally uses findings from descriptive analytics to discover cause of the events
		- KPIs are further investigated to discover why the events improved or worsened
		- this investigation happens in 3 steps
			- identify anomalies in the data. unexpected changes in metrics or a particular market
			- collect data that's related to these anomalies
			- use statistical techniques to discover relationships and trends that explain these anomalies
	- prescriptive analytics
		- help answer questions about which actions should be taken to achieve a goal or target
		- rely on ML as one of the strategies to find patterns in large semantic models
		- by analyzing past decisions and events, organizations can estimate the likelihood of different outcomes
	- predictive analytics
		- helps answer questions about what will happen in the future
		- use historical data to identify trends and determine if they are likely to recur
		- predictive analytical tools provide valuable insights into what might happen in the future
		- techniques include a variety of statistical and machine learning tech
	- cognitive analytics
		- attempt to draw inferences from existing data and patterns, derive conclusions based on existing knowledge bases, and then add these findings back into the knowledge base for future inferences, a self-learning feedback loop
		- helps us learn what might happen if circumstances change and determine how we might handle these situations
		- inferences are not structured queries based on a rules db, rather they are unstructured hypotheses that are gathered from several sources and expressed with varying degrees of confidence
		- depends on MLA

**Data** and **information** are related but distinct concepts, particularly in data engineering and general data processing.

1. **Data**:
   - Raw, unprocessed facts or values that are collected from various sources.
   - Can take many forms, including numbers, text, images, sounds, or sensor readings.
   - Generally lacks context or structure on its own, making it challenging to interpret meaningfully.
   - Examples: a list of temperatures recorded every hour, a log of website visits, or survey answers with no analysis applied.

2. **Information**:
   - Data that has been processed, organized, or structured in a way that gives it context and meaning.
   - Helps make sense of data by highlighting trends, patterns, or insights.
   - Is often the end product of data processing, providing value to users for decision-making.
   - Examples: a report summarizing temperature changes over a week, insights on website traffic patterns, or survey results with conclusions drawn.

### Key Differences
| Aspect       | Data                                | Information                                         |
| ------------ | ----------------------------------- | --------------------------------------------------- |
| **Nature**   | Raw, unprocessed, unstructured      | Processed, organized, structured                    |
| **Meaning**  | Lacks inherent meaning              | Carries meaning, context, and relevance             |
| **Use**      | Foundation for creating information | Enables understanding, insight, and decision-making |
| **Examples** | Numbers in a dataset, log files     | Trends, summaries, actionable insights              |

In a data pipeline, for instance, **data** would be the initial input, such as log files, that needs to be processed. **Information** would be the transformed, meaningful output, like a report on user behavior that helps guide business decisions.