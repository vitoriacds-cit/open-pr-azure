# open-pr-azure
  > Flow plugin for Claude Code — opens Azure DevOps Pull Requests with formatted descriptions linked to Jira tickets.                                                          
  ## What it does                                                                                                                                                              
  Complete workflow triggered by `/open-pr-azure` in Claude Code:                                                                                                         
  1. Checks repository state (uncommitted files, unpushed commits)                                                                                                              
  2. Collects branch context (commits, diff, changed files)                                                                                                                     
  3. Extracts the Jira ticket from the branch name                                                                                                                              
  4. Detects and fills in the repository's PR template (if any)                                                                                                                 
  5. Derives the PR title following Conventional Commits + Jira pattern                                                                                                         
  6. Creates the PR via `az repos pr create` and returns the URL                                                                                                                                                                                                                                                             
  ## Installation                                                                                                                                                            
  ```bash                                                                                                                                                                       
  flow plugin marketplace add vitoriacds-cit/open-pr-azure                                                                                                                      
  flow plugin install open-pr-azure@vitoriacds-cit/open-pr-azure                                                                                                                
                      
  Requirements                                                                                                                                                             
  - Azure CLI — brew install azure-cli                                                                                                                                          
  - Azure DevOps extension — az extension add --name azure-devops                                                                                                               
  - Authenticated: az login + az devops configure --defaults organization=https://dev.azure.com/<org>                                                                           
  - Branch name must contain the Jira ticket (e.g. feature/master/BEESKMM-1234-description)

PR title format                                                                                                                                                          
  <type>: [<JIRA-ID>] <Short description in imperative>                                                                                                        
  Types: feat, fix, chore, refactor, test, docs, perf, ci                                                                                                                 
  Examples:                                                                                                                                                                     
  feat: [BEESKMM-1234] Implement MemberHub use case                                                                                                                             
  fix: [BEESKMM-999] Prevent null pointer on enrollment mapper                                                                                                                  
  chore: [BEESKMM-777] Upgrade Kotlin coroutines version                                                                                                                                                                                                                                                                     
  Usage                                                                                                                                                                 
  Just ask Claude Code to open a PR:                                                                                                                                  
  - /open-pr-azure                                                                                                                                      - "abrir PR no Azure"                                                                                                                                 - "abre PR no ADO"                                                                                                                                    - "open PR azure"                                                                                                                                                          
  Claude will collect context, show you the title and description for review, and create the PR after confirmation. 
