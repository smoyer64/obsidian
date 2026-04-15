
* Formics (Memory)
	* Used for RAG
	* Memory for LLMs
	* Consolidated information from projects
	* Obsidian is the human front-end for Formics
	* Requires bi-directional synchronization
* Smobot
	* Automation
	* Agents
		* Orchestrator
		* Architect/designer
		* Project manager (task breakdown)
		* Go coder
		* Go tester
		* Maintainer (remediates tech debt)
	* Agent framework (Anthropic headers front-matter plus custom Selesy attributes).
		* SOUL.md for "main" system prompt, personality, etc.
		* Add follow-up prompts (rule-of-five)
		* Allow prompt repetition
- Kokomo
	- Generate conventional and idiomatic Go code
	- Agent should prefer generators over LLM output
	- Provide skills to allow coding agents to use generation
	- Many common Go patterns
	- Inspired by Lombok