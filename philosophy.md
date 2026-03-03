Purpose of Subagent Creation: keep the information in the main agent clean and the 128k context window of the main agent sufficient for finishing the task. 
Purpose of the Main Agent: The main agent should have high-level information about the task, and a clear overviwe about the entire workflow. Thus, the main agent should:
1. have sufficient  context window for knowing the overall workflow and the big picture of the task
2. have sufficient information for making decisions once the subagents report back
3. have sufficient space in 128k context window to last for the entire task