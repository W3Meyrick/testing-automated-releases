pipeline {
    agent any
    
    stages {
        stage('Automated Release') {
            when {
                // Trigger this stage only when on the main branch
                branch 'main'
            }
            steps {
                script {
                    // Determine the last branch merged into main
                    def mergedBranchMessage = sh(script: "git log --merges --pretty=format:'%s' -n 1", returnStdout: true).trim()
                    def parts = mergedBranchMessage.split(" ")
                    def lastElement = parts[-1]
                    def mergedBranch
                    if (lastElement.contains("/")) {
                        mergedBranch = lastElement.split("/")[1]
                    } else {
                        echo "Warning: Unable to determine merged branch name from commit message."
                        return
                    }
                    
                    def versionType
                    if (mergedBranch.startsWith("feat/")) {
                        versionType = "major"
                    } else if (mergedBranch.startsWith("patch/") || mergedBranch.startsWith("fix/")) {
                        versionType = "minor"
                    } else {
                        echo "Warning: Unsupported branch name format: ${mergedBranch}."
                        return
                    }
                    
                    // Generate tag
                    def latestTag
                    try {
                        latestTag = sh(script: "git describe --abbrev=0", returnStdout: true).trim()
                    } catch (Exception e) {
                        echo "Warning: No tags found in the repository. Setting version to 0.0.0."
                        latestTag = null
                    }
                    def version = latestTag ?: '0.0.0'
                    if (version.startsWith("v")) {
                        version = version.substring(1)
                    }
                    def tagName = "v${version}"
                    sh "git tag -a $tagName -m 'Automated tag generated for $tagName'"
                    sh "git push origin $tagName"
                    
                    // Update version number
                    def newVersion
                    def partsVersion = version.tokenize('.')
                    if (versionType == "major") {
                        partsVersion[0] = (partsVersion[0] as Integer) + 1
                        partsVersion[1] = 0
                        partsVersion[2] = 0
                    } else if (versionType == "minor") {
                        partsVersion[1] = (partsVersion[1] as Integer) + 1
                        partsVersion[2] = 0
                    } else {
                        error "Unsupported version type"
                    }
                    newVersion = partsVersion.join('.')
                    
                    sh "echo $newVersion > version.txt"
                    sh "git add version.txt"
                    sh "git commit -m 'Update version number to $newVersion'"
                    sh "git push origin HEAD"
                    
                    // Generate structured release notes
                    def commitLogs = sh(script: "git log --pretty=format:'%h - %s (%an)' origin/main..HEAD", returnStdout: true).trim()
                    def releaseNotes = "## Changes since last release:\n\n${commitLogs}"
                    
                    // Save release notes to a file
                    writeFile file: 'release_notes.md', text: releaseNotes
                    
                    // Generate release
                    def releaseName = "Release ${newVersion}"
                    sh "curl -X POST -H 'Authorization: token <your_github_token>' -F 'tag_name=$tagName' -F 'name=$releaseName' -F 'body=@release_notes.md' https://api.github.com/repos/<your_username>/<your_repo>/releases"
                }
            }
        }
    }
}
