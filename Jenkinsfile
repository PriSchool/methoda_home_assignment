pipeline {
    agent any

    stages {
        stage('Check if branch that is merged into is main') {
            steps {
                script {
                    def branchName = scm.branches[0].name
                    if (branchName == 'main') {
                        currentBuild.result = 'SUCCESS'
                    } else {
                        currentBuild.result = 'ABORTED'
                        error("Not main branch")
                    }
                }
            }
        }

        stage('Get the name of branch that is merging into main') {
            steps {
                script {
                    def mergedBranch = sh (script: 'git log --merges --pretty=format:%s | head -n 1', returnStdout: true).trim()
                    
                    if (mergedBranch) {
                        echo "Merged branch: ${mergedBranch}"
                        env.MERGED_BRANCH_NAME = mergedBranch
                    } else {
                        error("No merged branch was found")
                    }
                }
            }
        }

        stage('Extract Key name') {
            steps {
                script {
                    def mergedBranchName = env.MERGED_BRANCH_NAME
                    def regex = /([A-Z]+-\d+)/
                    def matcher = (mergedBranchName =~ regex)

                    if (matcher) {
                        def issueKey = matcher[0][0]
                        echo "Found matching Jira Issue Key name: ${issueKey}"
                        env.ISSUE_KEY = issueKey
                    } else {
                        error("No Jira Issue Key name found in branch name")
                    }
                }
            }
        }
        
        stage('Check Jira Issue') {
            steps {
                script {
                    def jiraIssueKey = env.ISSUE_KEY
                    def jiraAuthToken = env.JIRA_AUTH_TOKEN

                    if (jiraIssueKey) {
                        def jiraApiUrl = "http://jira:8080/rest/api/2/issue/${jiraIssueKey}"

                        def curlCommand = "curl -s -o /dev/null -H 'Authorization: Basic ${jiraAuthToken}' -w \"%{http_code}\" ${jiraApiUrl}"
                        def responseCode = curlCommand.execute().text.toInteger()

                        if (responseCode == 200) {
                            println "Jira issue ${jiraIssueKey} exists."
                        } else {
                            error("Jira issue ${jiraIssueKey} does not exist")
                        }
                    }
                }
            }
        }

        stage('Mark Jira Key as Done') {
            steps {
                script {
                    def jiraIssueKey = env.ISSUE_KEY
                    def jiraTransitionId = 31

                    def jiraApiUrl = "http://jira:8080/rest/api/2/issue/${jiraIssueKey}/transitions"
                    def jiraAuthToken = env.JIRA_AUTH_TOKEN

                    def response = sh(
                        script: """
                            curl -X POST -D- -H "Authorization: Basic ${jiraAuthToken}" -H 'Content-Type: application/json' --data '{
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
