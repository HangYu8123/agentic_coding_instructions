# create necessary files for guiding the entire agentic coding workflow

#Procedure 1
First go through the entire repo, keep what files exist in this repo in the memeory. 

#Procedure 2
Then, check the existence of the folliwing files:
1. codebase_overview.md
2. scripts_overview.md
3. update_logs_auto_generated.md
4. known_issues_auto_generated.md
5. update_logs.md
6. known_issues.md

#Procedure 3
Create a subagent, read through all the scripts in the code repo, understand them, and create a pipeline for the entire repo. The subagent should feed the pipeline of the code back to the main agent. 


#Procedure 4
create/update the files in step 2 with the **follwoing specifications in the following order**. 


##1 codebase_overview.md:
If the file does not exist, create an empty file, and set the pipeline to be the pipeline generate by Procedure 3.
If the file exists, the main agent should read through the file and keep it inside the memeory, and set the pipeline to be the diagram pipeline in the file.
Then:
1. create a subagent, follow the pipeline of the repo, go through all the scripts in the order that specifies the pipeline and check the correctness of the pipeline. Then return the results to the main agent. 
2. based on the existing pipeline in the memory, and the review from the subagent, the main agent specifically checks the inconsistency, and make the final decision. 
3. the main agent should update the pipeline based on the final decision
4. the main agent should convert the pipeline into a code diagram. In each block in the diagram, the associate scripts should also be mentioned. 
5. Then, create a subagent, pass the pipeline diagram to the subagent, the subagent go through the generated diagram and associate scripts step by step and make sure the correctness and consistency. Then feed the review back to the main agent. 
6. Then based on the diagram and the review, the main agent check the correctness by itself and update the diagram accordingly.
7. Finally, write the diagram into the codebase_overview.md, along with a description of the repo. 


##2 scripts_overview.md
If the file does not exist, create an empty file.
Then:
1. create a subagent, the subagent should go through all the files in the repo, then:
    a. read through files one by one. 
    b. for reach file, if it is a code script, summerize each module (function, method, class, code blocks) with two sentences: one sentence of function name, paramters, and outputs, one short sentece describes the functionality.
    c. orgnize the summerization by files: give each file a high level summerization, and give out a list of denpendencies of that file. 
    d. write everything to scripts_overview.md.
2. create a subagent (review agent), the subagent first reads the scripts_overview.md, follows the scripts_overview.md go through all the scripts and files one by one, first read the orignal code/text, then validate the summerization of the scripts_overview.md. Report inconsistence back the main agent.
3. the main agent reads the report from  step 2 and scripts_overview.md file, validate:
    a. whether all scripts and files are in the scripts_overview.md, if not, create a subagent to repeat the previous step and cover the missing files.
    b. the main agent, based on scripts_overview.md and the report from step 2, check the inconsistency and finalize the scripts_overview.md
4. write the scripts_overview.md
  

##3 known_issues_auto_generated.md:
if the file exists, do nothing. 
if the file does not exist, create an empty file.
then:
1. create a subagent (plan agent), go through the codebase_overview.md and scripts_overview.md, point out the weakness of the code architecture and all possible issues. report back to the main agent.
2. create a subagent (code agent), go through the codebase_overview.md and scripts_overview.md, and then go through all the scripts one by one, find any pontial issues, code could lead to problems, errors, and bugs. find anything that could affect the code be 100% correct. find anything that prevent code being running 100% correct or function as expected. report back to main agent.
3. the main agent summerize the reviews from step 1 and step 2, based on the information it has, use its best ablity to combine two reviews with only correct and fair part.
4. write the contents to known_issues.md in the format of: 
{Problem Title (very high level summerization)}
{Problem description ( a short description of the problem)}
{Root causes (for example, what code/function is cause the problem )}
{Consequences (what issues can this problem lead to)}. 

##4 update_logs_auto_generated.md
get the git commit history, and create a file with the git commit history ( do not add any interpurtations, be faithful to the original contents) logs. 


##5 known_issues.md:
if the file exists, do nothing. 
if the file does not exist, create an empty file.

##6 known_issues.md:
if the file exists, do nothing. 
if the file does not exist, create an empty file. 

**Finally. add  code_instructions.md, debug_instructions.md, and initialize.md files to .gitignore**