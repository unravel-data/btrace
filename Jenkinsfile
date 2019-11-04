#!/usr/bin/env groovy

node {
    def gitUrl = 'git@github.com:unraveldata-org/btrace.git'
    def projectName = 'Btrace'
    def mvnHome
    def release
    def isCandidate = false
    def isRelease = false
    def topTag = ''

    stage("Checkout") {
        def branch = env.BRANCH_NAME
        if (branch.startsWith("PR-")) {
            branch = env.CHANGE_BRANCH
        }
        git url: gitUrl, branch: branch
    }

    stage("Setup") {
        mvnHome = tool 'M3'
        properties(
                [
                        disableConcurrentBuilds(),
                        pipelineTriggers([pollSCM('H/5 * * * *')]),
                ]
        )

    }

    try {
            stage("Build") {
                        sh "./gradlew build ; ./gradlew buildDistributions ; ./gradlew publish"
                }
    } catch (any) {
        currentBuild.result = 'FAILURE'
        throw any // rethrow exception
    } finally {
        // do some cleanup
    }
}

def getChangelog(prevRelease) {
    def log = gitChangelog customIssues: [[issuePattern: '[A-Z][A-Z]*-[0-9][0-9]*', link: 'https://unraveldata.atlassian.net/browse/${PATTERN_GROUP}', name: 'JIRA', title: '']], from: [type: 'REF', value: "${prevRelease}"], returnType: 'STRING', template: '''
        <h2>Changelog for {{ownerName}} {{repoName}}.</h2>

        {{#tags}}
         <table width="80%">
           <caption>Version: {{name}}</name>
           <thead>
             <th align="left" width="40%">JIRA</th>
             <th align="left" width="20%">GitHub</th>
             <th align="left" width="40%">Details</th>
           </thead>
           <tbody>
             {{#issues}}
               {{#commits}}
               <tr>
                 <td align="left"><a href='{{link}}'>{{{messageTitle}}}</a></td>
                 <td align="left"><a href='https://github.com/{{ownerName}}/{{repoName}}/commit/{{hash}}'>{{hash}}</a></td>
                 <td align="left">
                    {{authorName}} <i>{{commitTime}}</i>
                    <ul>
                        {{#messageBodyItems}}
                            <li>{{.}}</li>
                        {{/messageBodyItems}}
                    </ul>
                 </td>
                </tr>
              {{/commits}}

             {{/issues}}
           </tbody>
         </table>
        {{/tags}}''', to: [type: 'REF', value: "HEAD"], ignoreCommitsWithoutIssue: true

    return log
}

def splitVersion(versionStr) {
    def m = versionStr =~ /([0-9]+)\.([0-9]+)\.([0-9]+)/
    if (m) {
        return [Integer.valueOf(m.group(1)), Integer.valueOf(m.group(2)), Integer.valueOf(m.group(3))] as int[]
    }
    return null
}
