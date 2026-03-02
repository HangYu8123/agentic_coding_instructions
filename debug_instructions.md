# debug instructions
inputs: 
input 1: target bug 
input 2: important scripts (optional)


When ask to debug, always, first read the folliwing files (**REFER AS KEY MD FILES**):
1. codebase_overview.md
2. scripts_overview.md
3. update_logs.md
4. known_issues.md
Understand them, and keep them inside the memory. 
Then, the main agent should read through all the files inside the repo and get a detailed understanding. 


Then, the main agent should:
1. the main agent creates a subgent (plan agent), pass the input information to the subagent. The subagent should also read through the key md files. Then based on the bug information and the repo structure based on the key md files, read all scripts that could associated with the bug. according to the input information and key md files, analysis the issue and start addressing the cause of the bugs. After understand the bug, the subagent should draft a plan that can fix the bug while maintaining the entire codebase functions the same and not introducing any new bugs, then feed the plan back to the main agent. 
2. the main agent creates a subagent (plan agent, senior staff engineering role), pass the plan from the other subagent and bug information to this subagent. The subagent should additionally read through the key md file and all scripts in this repo to get a comprehensive understand of the entire repo. If the plan involves any repo outside this repo, go to that repo, if there are codebase_overview.md and scripts_overview.md, read through them too. Then the subagent reviews the plan based on the information it has from a senior staff engineer perspective, assess the plan's correctness and feasibility, making sure that the plan can 100% fix the bug without breaking the current codebase. feed the review back to the main agent.
3. the main agent reviews the plan from step 1 and the review from step 2. If the plan or the review involves any other repos, go to thoese repos, read their codebase_overview.md and scripts_overview.md if exist, and keep those in the memory. Finally, combine all those information and draft a final plan that is feasible, stable, and 100% correct. 
4. the main agent creates a subagent (code agent), pass the plan and bug information to the subagent. The subagent should also read through the key md files. Then based on the bug information, the plan, and the repo structure based on the key md files, read all scripts that could associated with the bug and the plan. Then implement the plan and fix the bug accordingly. feed a implementation report (just what has been changed, **no explaination** why it would fix bug) to the main agent. 
5. the main agent creates a subagent (code agent, senior staff engineering role), pass the plan, bug information, and implementation report from the other subagent to this subagent. The subagent should additionally read through the key md file and all the code changes in the repo. Then the subagent reviews the code changes and the implementation from a senior staff engineer perspective, assess the code implementation correctness, making sure that the bug is 100% fixed without breaking the current codebase. feed the review back to the main agent.
6. the main agent should read through the code changes, the implentation report and the review, finalize all the changes accordingly, and make sure the bug is 100% addressed and fixed. 
7. the main agent should summerize the bug fix in the follow format:
{=============================BUG FIX===============================}
{BUG Name (very high level description of the bug) and Bug Id (assign a number in order, i.e., plus 1 to the last bug id)}
{Bug description (one or two sentences of description of what the bug is)}
{Repo involved (what local repos are involved)}
{Implementation ( what has been changed to fix the bug)}
{Fixed (whether the bug has been fixed)}
8. write the summary to the update_logs.md and known_issues.md. do not add additional contents, just the bug fix report from previous step.  