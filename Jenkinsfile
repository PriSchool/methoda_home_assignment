pipeline {
    agent any

    stages {
        stage('Check if branch that is merged into is main') {
            steps {
                script {
                    def branchName = scm.branches[0].name
                    println branchName
                    if (branchName == '*/main') {
                        currentBuild.result = 'SUCCESS'
                    } else {
                        currentBuild.result = 'ABORTED'
                        error("Not main branch")
                    }
                }
            }
        }

        stage('Get name of branch that is merging into main') {
            steps {
                script {
                    def currentBranch = scm.branches[0].name
                    def mergedBranch = sh(script: "git log --merges --pretty=format:'%h %s' --abbrev-commit", returnStdout: true).trim()
                    println mergedBranch
                    def regex = /Merge branch '(.+)' into '${currentBranch}'/

                    def matcher = (mergedBranch =~ regex)
                    if (matcher) {
                        def mergedBranchName = matcher[0][1]
                        echo "Merged branch: ${mergedBranchName}"
                        env.MERGED_BRANCH_NAME = mergedBranchName
                    } else {
                        error("No merged branch found.")
                    }
                }
            }
        }

        stage('Extract Key name') {
            steps {
                script {
                    def branchName = scm.branches[0].name
                    def regex = /([A-Z]+-\d+)/
                    def matcher = (branchName =~ regex)

                    if (matcher) {
                        def issueKey = matcher[0][0]
                        echo "Found Jira Issue Key: ${issueKey}"
                        env.ISSUE_KEY = issueKey
                    } else {
                        error("No Jira Issue Key found in branch name")
                    }
                }
            }
        }
        
        stage('Check Jira Issue') {
            steps {
                script {
                    def jiraIssueKey = env.ISSUE_KEY
                    if (jiraIssueKey) {
                        // Use the jiraIssueKey to check if the issue exists
                        def issueExists = checkIfIssueExists(jiraIssueKey) // Implement this function
                        if (issueExists) {
                            currentBuild.result = 'SUCCESS'
                        } else {
                            currentBuild.result = 'FAILURE'
                            error("Jira issue ${jiraIssueKey} does not exist.")
                        }
                    } else {
                        currentBuild.result = 'FAILURE'
                        error("Issue key not found.")
                    }
                }
            }
        }

        stage('Mark Jira Key as Done') {
            steps {
                script {
                    def jiraIssueKey = env.ISSUE_KEY // Replace with the extracted issue key
                    def jiraTransitionId = 21 // Replace with the transition ID for "Done" status

                    def jiraApiUrl = "https://jira:8080/rest/api/2/issue/${jiraIssueKey}/transitions"
                    def jiraCredentialsId = 'tomerprielg'

                    def response = sh(
                        script: """
                            curl -X POST -D- -u ${jiraCredentialsId} -H 'Content-Type: application/json' --data '{
                                "transition": {
                                    "id": ${jiraTransitionId}
                                }
                            }' ${jiraApiUrl}
                        """,
                        returnStatus: true
                    )

                    if (response != 0) {
                        error("Failed to transition Jira issue to 'Done'")
                    }
                }
            }
        }
    }
}