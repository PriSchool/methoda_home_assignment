# Mark Jira Issues as "Done" - Pipeline

## Overview

This repository is for simulating a feature/issue branch that is being merged into the main branch. All three services (Confluence, Jenkins, Jira) are accessible using ngrok and are defined in a separate repository [here](https://github.com/PriSchool/methoda_home_assignment_done_issue).

## Usage

To trigger the pipeline and mark a Jira issue as "Done," follow these steps:

1. Create a branch following the naming convention: `Capital Letters -> Hyphen -> Integer`, for example, "HOM-33."

2. Merge the branch into the main branch using the following command:
```
git merge --no-ff <BRANCH_ISSUE_NAME>'
```

3. Push the changes to GitHub.

4. The webhook connected to Jenkins will automatically detect the push and start the pipeline.

## Repository Structure

This repository contains the necessary configuration and scripts to facilitate the automation of marking Jira issues as "Done" when a branch is merged into the main branch.
