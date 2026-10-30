1. Gather user requirements:
	1. functionality about the system - features, tools etc
	2. Infra requirements - 
		1. peak load handling: RPS
		2. Seasonality
		3. latency: real-time, batching
	3. Database: Relational or Vector DB
2. Design schema - 
	1. the database
3. mock up a user journey - 
	1. how user would flow through frontend
	2. how the user would interact with chat application
	3. what the AI would respond with: clarifying questions
	4. the followup actions
4. design services:
	1. application layer
		1. queuing layer
	2. payment layer
	3. split bill layer
	4. notification layer
	5. internt identification agent