# bootcamp-jenkins-demo
An independent repository to expose Jenkins demos

```groovy
pipeline {
    agent any
    /*diff*/
    parameters {
        booleanParam(name: 'RC', defaultValue: false, description: 'Is this a Release Candidate?')
    }
    /*diff*/
    environment {
        VERSION = ""
        VERSION_RC = "rc.2"
    }
    stages {
        stage('Audit tools') {
            steps {
                sh '''
                    git version
                    docker version
                    node --version
                    npm version
                '''
            }
        }
        stage('Build') {
            /*diff*/
            environment {
                VERSION_SUFFIX = "${sh(script:'if [ "${RC}" == "false" ] ; then echo -n "${VERSION_RC}+ci.${BUILD_NUMBER}"; else echo -n "${VERSION_RC}"; fi', returnStdout: true)}"
            }
            /*diff*/
            steps { 
                env.VERSION = sh([ script: 'npm run env | grep "npm_package_version"', returnStdout: true ]).trim()
                // echo "Building version ${VERSION} with suffix: ${VERSION_RC}"
                echo "Building version ${VERSION} with suffix: ${VERSION_SUFFIX}"
                sh '''
                    npm install
                    npm run build
                '''
            }
        }
        stage('Unit Test') {
            steps {
                dir('./01/src') {
                    sh 'npm test'
                }
            }
        }
        stage('Publish') {
            when {
                expression { return params.RC }
                steps {
                    archiveArtifacts('app/')
                }
            }
        }
    }
}
```

