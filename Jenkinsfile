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
                    def mergedBranch = getLastMergedBranch()
                    if (mergedBranch == null) {
                        echo "Skipping tagging and release due to inability to determine merged branch."
                        return
                    }
                    
                    def versionType = determineVersionType(mergedBranch)
                    if (versionType == null) {
                        echo "Skipping tagging and release due to unsupported branch name format."
                        return
                    }
                    
                    // Generate tag
                    def tagName = generateTagName(versionType)
                    sh "git tag -a $tagName -m 'Automated tag generated for $tagName'"
                    sh "git push origin $tagName"
                    
                    // Update version number
                    def newVersion = incrementVersion(versionType)
                    sh "echo $newVersion > version.txt"
                    sh "git add version.txt"
                    sh "git commit -m 'Update version number to $newVersion'"
                    sh "git push origin HEAD"
                    
                    // Generate structured release notes
                    def releaseNotes = generateReleaseNotes()
                    
                    // Generate release
                    def releaseName = generateReleaseName(newVersion)
                    sh "curl -X POST -H 'Authorization: token <your_github_token>' -d '{\"tag_name\": \"$tagName\", \"name\": \"$releaseName\", \"body\": \"$releaseNotes\"}' https://api.github.com/repos/<your_username>/<your_repo>/releases"
                }
            }
        }
    }
}

def determineVersionType(branchName) {
    if (branchName.startsWith("feat/")) {
        return "major"
    } else if (branchName.startsWith("patch/") || branchName.startsWith("fix/")) {
        return "minor"
    } else {
        echo "Warning: Unsupported branch name format: ${branchName}."
        return null
    }
}

def generateTagName(versionType) {
    def version = getVersionNumber()
    return "v${version}"
}

def generateReleaseName(version) {
    return "Release ${version}"
}

def incrementVersion(versionType) {
    def currentVersion = getVersionNumber()
    def parts = currentVersion.tokenize('.')
    if (versionType == "major") {
        parts[0] = (parts[0] as Integer) + 1
        parts[1] = 0
        parts[2] = 0
    } else if (versionType == "minor") {
        parts[1] = (parts[1] as Integer) + 1
        parts[2] = 0
    } else {
        error "Unsupported version type"
    }
    return parts.join('.')
}

def getVersionNumber() {
    def versionFile = readFile('version.txt').trim()
    return versionFile ?: '0.1.0'
}

def generateReleaseNotes() {
    def commitLogs = sh(script: "git log --pretty=format:'%h - %s (%an)' origin/main..HEAD", returnStdout: true).trim()
    def releaseNotes = "## Changes since last release:\n\n${commitLogs}"
    return releaseNotes
}

def getLastMergedBranch() {
    def mergedBranchMessage = sh(script: "git log --merges --pretty=format:'%s' -n 1", returnStdout: true).trim()
    if (mergedBranchMessage.startsWith("Merge pull request")) {
        def parts = mergedBranchMessage.split(" ")
        if (parts.size() >= 7) {
            return parts[6]
        }
    } else if (mergedBranchMessage.startsWith("Merge")) {
        def parts = mergedBranchMessage.split(" ")
        if (parts.size() >= 5) {
            return parts[4]
        }
    }
    echo "Warning: Unable to determine merged branch name from commit message."
    return null
}
