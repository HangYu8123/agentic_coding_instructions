This is a repo that collects MD files for enhancing coding agents and building workflow for them.


To get the best experience, enable auto approve (** KEEP ANY CMDS THAT CAN DELETE YOUR FILES DISABLED ** )

#How to use

## Create a .github folder under your repo if you dont hve one

## Drag the instructions folder into the .github folder

## Add this to your VS Code settings.json to ensure the folders are scanned:

{
  "chat.instructionsFilesLocations": {
    ".github/instructions": true,
    ".claude/rules": true
  }
}

## initialize the repo using initialize.md with prompt:
following the instructions in @.github/instructions/workflow/initialize.instructions.md to initialize the repo [repo name]

## then it will automatically generates files that keeps logs of system overview, functionality updates, bug logs, known issues, and past Q&A. 

## then, when you have new requests, the copilot agent should follow the corresponding instructions **If you want to be 100% sure that copilots follows the instructions**: 
1. drag the instruction.md to the chat window
2. drag the repo folder to the chat window if you have multiple repos in your workspace
3. drag the files you want to modify to the chat window
4. use the request templates in the general folder

# Ongoing:

create prompts for generating a .claude.md that logs all mistatkes that claude has made to prevent future errors in general. 