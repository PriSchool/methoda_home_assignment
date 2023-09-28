pipeline {
    agent any

    stages {
        stage('Check if branch is main') {
            steps {
                script {
                    def branchName = build.changeSets[0].branch
                    if (branchName == 'origin/main') {
                        build.result = 'SUCCESS'
                    } else {
                        build.result = 'ABORTED'
                        error("Not main branch")
                    }
                }
            }
        }

        stage('Extract Key name') {
            steps {
                script {
                    def branchName = build.changeSets[0].branch
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
                            build.result = 'SUCCESS'
                        } else {
                            build.result = 'FAILURE'
                            error("Jira issue ${jiraIssueKey} does not exist.")
                        }
                    } else {
                        build.result = 'FAILURE'
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
