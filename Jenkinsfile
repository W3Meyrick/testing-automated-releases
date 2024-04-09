pipeline {
    agent any
    
    stages {
        stage('Automated Release') {
            when {
                // Execute this stage only when triggered by a pull request merge event
                expression {
                    return env.CHANGE_TARGET == 'master' && env.CHANGE_ID != null
                }
            }
            steps {
                script {
                    def versionType = determineVersionType(env.CHANGE_BRANCH)
                    
                    // Generate tag
                    def tagName = generateTagName(versionType)
                    sh "git tag -a $tagName -m 'Automated tag generated for $tagName'"
                    sh "git push origin $tagName"
                    
                    // Update version number
                    def newVersion = incrementVersion(versionType)
                    sh "echo $newVersion > version.txt"
                    sh "git add version.txt"
                    sh "git commit -m 'Update version number to $newVersion'"
                    sh "git push origin master"
                    
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
        error "Unsupported branch name format"
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
    def commitLogs = sh(script: "git log --pretty=format:'%h - %s (%an)' origin/master..HEAD", returnStdout: true).trim()
    def releaseNotes = "## Changes since last release:\n\n${commitLogs}"
    return releaseNotes
}
