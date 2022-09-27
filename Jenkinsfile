pipeline {
    agent any
    tools {
        maven 'Maven 3.6.2'
    }
    environment {
        GITLAB = credentials('a31843c7-9aa6-4723-95ff-87a1feb934a1')
    }
    stages {
        stage('Parameters Set-up') {
            steps {
                script {
                    properties([
                        disableConcurrentBuilds(), 
                        gitLabConnection(gitLabConnection: 'GitLab API Connection', jobCredentialId: ''), 
                        [$class: 'GitlabLogoProperty', repositoryName: 'adam/suggest-lib'], 
                        parameters([
                            validatingString(
                                description: 'Put "release" and a 2-digit value meaning the branch you would like to build your version on, as in the following examples: "release/1.0", "release/1.1", "release/1.2", etc. For either master or feature branch to choose, type "feature" or "master".', 
                                failedValidationMessage: 'Parameter format is not valid. Try again with valid parameter format.', 
                                name: 'Version', 
                                regex: '^master$|^feature$|release\\/[0-9]{1,}\\.[0-9]{1,}$'
                            )
                        ]), 
                    ])
                }
            }
        }
        stage('Release branch versioning') {
            when { branch "release/*" }
            steps{
                script {
                    BRANCH = params.Version
                    sh"""
                        git checkout master
                        git remote set-url origin http://\"$GITLAB\"@ec2-3-125-51-254.eu-central-1.compute.amazonaws.com/adam/suggest-lib.git
                        git pull --rebase
                    """
                    BRANCH_EXISTING = sh(
                        script: "(git ls-remote -q | grep -w $BRANCH) || BRANCH_EXISTING=False",
                        returnStdout: true,
                    )
                    if (BRANCH_EXISTING) {
                        echo "The $BRANCH branch is already existing."
                        sh """
                        git checkout $BRANCH
                        git pull origin $BRANCH --rebase
                        git fetch --tags
                        """
                    } else {
                        echo "The $BRANCH branch is not exsiting yet and needs to be created."
                        sh"""
                        git branch $BRANCH
                        git checkout $BRANCH
                        git remote set-url origin http://\"$GITLAB\"@ec2-3-125-51-254.eu-central-1.compute.amazonaws.com/adam/suggest-lib.git
                        git fetch --tags
                        """
                    }
                    MINOR_VERSION = BRANCH.split("/")[1]
                    LATEST_TAG = sh(
                        script: "git describe --tags --abbrev=0 | grep -E '^$MINOR_VERSION' || true",
                        returnStdout: true,
                    )
                    echo "This is $LATEST_TAG"
                    if (LATEST_TAG) {
                        NEW_PATCH = (LATEST_TAG.tokenize(".")[2].toInteger() + 1).toString()
                    } else {
                        NEW_PATCH = "0"
                    }
                    NEW_TAG = MINOR_VERSION + "." + NEW_PATCH
                    configFileProvider([configFile(fileId: 'fc3e184d-fd76-4262-a1e7-9a5671ebd340', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn versions:set -DnewVersion=$NEW_TAG"
                    }
                }
            }
        }
        stage('Suggest-Lib Build') {
            when {
                anyOf {
                    branch "release/*"
                    branch "master"
                    branch "feature"
                }
            }
            steps {
                script {
                    sh "mvn clean verify"
                }
            }
            post {
                success {
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
        stage('Suggest-Lib Publish') {
            when {
                anyOf {
                    branch "master"
                    branch "release/*"
                }
            }
            steps {
                script {
                    configFileProvider([configFile(fileId: 'fc3e184d-fd76-4262-a1e7-9a5671ebd340', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn  -Dmaven.test.failure.ignore=true -DskipTests -s $MAVEN_SETTINGS_XML deploy"
                    }
                    echo "New artifact has been published in to artifactory."
                }
            }
        }
        stage('Tagging and Pushing to GitLab Repository') {
            when { branch "release/*" }
            steps {
                script {
                    sh"""
                    git config --global user.email "adam.stegienko1@gmail.com"
                    git config --global user.name "Adam Stegienko"
                    git clean -f -x
                    git tag -a $NEW_TAG -m \"New $NEW_TAG tag added to branch $BRANCH\"
                    git push origin $BRANCH --tag
                    """
                }
            }
        }
    }
}
