---
name: 'Code Implementation'
description: 'Instructions for implementing new functionalities'
applyTo: '**'
---
# add new functions to an existing repo
**DO NOT TRY TO COMMIT CHANGES TO GITHUB**
**DO NOT WRITE SPAM FILES INTO THE REPO**
**DO NOT USE SUDO**
inputs:
input 1: target functionalities
input 2: important files (optional)
input 3: target repo (optional)

**read through this entire file and follow the instructions carefully**. 

When ask to implement new functionalities, always, first read the folliwing files @.github/instructions/workflow (REFER AS KEY MD FILES):
1. codebase_overview.md
2. scripts_overview.md
3. update_logs.md
4. known_issues.md
Understand them, and keep them inside the memory. 
The main agent should through all files and scripts inside the repo and get a detailed understanding. 


#CREATE ONE TODO FOR EACH OF THE FOLLOWING STEPS
then, for implementing new functionalities to an existing codebase, **CREATE ONE TODO FOR EACH STEP**:
1. if preferred files are specified, the main agent should read through the preferred files, and understand them. then combine the understanded knowledge with the key md files. 
2. the main agent creates a subgent (plan agent), pass the input information to the subagent. The subagent should also read through the key md files. Then based on the input information and the repo structure based on the key md files, read all scripts that could associated with the new functionalities. according to the input information and key md files, analysis the new functionalities and how to intergrate the new functionalites to the existing codebase. Then, the subagent should draft a plan that intergrates the new functionalities into the existing codebase and draft a diagram that intergrates the new functionalities to the codebase diagram, while maintaining the entire codebase performs stable, not introducing any new bugs, and would not repeate any known issues/bugs in known_issues.md. then the subagent feeds the plan and the implementation diagram back to the main agent.
3. the main agent creates a subagent (plan agent, senior staff engineering role), pass the plan and the implementation diagram from the other subagent and input information to this subagent. The subagent should additionally read through the key md file and all scripts in this repo to get a comprehensive understand of the entire repo. If the plan involves any repo outside this repo, go to that repo, if there are codebase_overview.md and scripts_overview.md, read through them too. Then the subagent reviews the plan and the diagram based on the information it has from a senior staff engineer perspective, assess the plan's and the diagram's correctness and feasibility, making sure that the plan and the diagram can 100% achieves the new functionalities without breaking the current codebase. feed the review back to the main agent. 
4. the main agent reviews the plan and the implementation diagram from step 2 and the review from step 3. If the plan or the review involves any other repos, go to thoese repos, read their codebase_overview.md and scripts_overview.md if exist, and keep those in the memory. Finally, combine all those information and draft a final plan that is feasible, stable, and 100% correct. 
5. the main agent creates a subagent (code agent), pass the plan and input information to the subagent. The subagent should also read through the key md files. Then based on the input information, the plan, and the repo structure based on the key md files, read all scripts that could associated with the input, the new functionalities and the plan. Then implement the plan and acheive the new functionalities accordingly. feed a implementation report (just what has been changed, **no explaination** why it would acheives the new functionalities) to the main agent. 
6. the main agent creates a subagent (code agent, senior staff engineering role), pass the plan, input information, new functionalites, and implementation report from the other subagent to this subagent. The subagent should additionally read through the key md file and all the code changes in the repo. Then the subagent reviews the code changes and the implementations from a senior staff engineer perspective, assess the code implementation correctness, making sure that the new functionalities are 100% achieved without breaking the current codebase. feed the review back to the main agent.
7.  the main agent creates a subagent (code agent), pass the plan, input information, new functionalites, and implementation report from the other subagent to this subagent. The subagent should additionally read through the key md file and all the code changes in the repo. Then the subagent run through the entire repo pipeline, test: a. if the entire repo still performs correctly; b. if the new implemented functionalities performs as expected with 0 error. Then report the running results to the main agent. 
8. the main agent should read through the code changes, the implentation report, the review, and the running results, then follow the codebase pipeline specified by the diagram and check the correctness of the implementation for the entire codebase. Then, finalize all the changes accordingly, and make sure the new functionalities are 100% achieved without any bugs introduced.  
9. the main agent should summerize the implementation in the follow format, for each new functionality:
{=============================Function Update===============================}
{functionality Name (very high level description of the functionality) and functionality Id (assign a number in order, i.e., plus 1 to the last functionality id)}
{functionality description (one or two sentences of description of what the functionality is)}
{Repo involved (what local repos are involved)}
{Implementation ( what has been implemented to achieve the functionality)}
{Achived (whether the functionality has been achieved)}

