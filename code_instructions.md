General Coding Guidance:
when ask for adding new functionalities to an existing codebase:
    1. always, first read through the target scripts, keep the scripts in the memory. 
    2. then the main agent creates a subagent (code agent) to read through the target scripts' dependicies, and the scripts that depen on the target scripts. feed the list of the scripts and their necessary information to the main agent. 
    3. then the main agent creates a subagent (plan agent) to plan the implementation of the new functionalities, and feed the plan to the main agent.
    4. then the main agent creates a subagent (plan agent) to review and validate the implementation of the new functionalities, and feed the review result to the main agent.
    5. in the main agent, based on the information from the code agent, the plan from the plan agent, and the review result from the other plan agent, from a senior engineer's perspective, combines all the information, and draft the final plan for the implementation of the new functionalities. keep the final plan in the memory.
    6. then, the main agent creates a subagent (coding agent), feed the final plan to the coding agent, and ask the coding agent to implement the new functionalities based on the final plan. After the coding agent finishes the implementation, this coding agent should give a summary of the implementation back to the main agent.
    7. then, the main agent creates a subagent (coding agent), feed the summary of the implementation to the coding agent, the final implementaion plan, and the necessary information about the target scripts and its dependencies, and ask the coding agent to review the implementation of the new functionalities from a senior engineer's perspective, majorly focus on  thge correctness of the code and the robustness of the code and whether it will break the existing codebase. Then return the review back to the main agent.frozenset
    8. then, based on the review results, the main agent processes the reviews, decide what are necessary to follow or fix, then creates a subagent (coding agent) to fix the necessary problems.
    9. after all the above steps are done, go through the main agent should create a subagent (coding agent) to go through the entire workflow again, and make sure the new functionalities are implemented 100% correctly, and the entire codebase is 100% functional.
    10. then the main agent should provide a summary of the final implementation result.



Purpose of Subagent Creation: keep the information in the main agent clean. 
Purpose of the Main Agent:The main agent should have detailed information about the target scripts, high-level information about the associated dependencies about the target scripts and the entire codebase, and the necessary and sufficent information about the implementation and the entire workflow. Thus, the main agent can:
1. have sufficient information and context window for knowing the overall workflow and make the right decision on each step. 