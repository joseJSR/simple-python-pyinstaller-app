pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.11.5-alpine3.18'
                    args '-v /var/run/docker.sock:/var/run/docker.sock' // Montar el socket de Docker si es necesario
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock' // Montar el socket de Docker si es necesario
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            agent any
            environment {
                VOLUME = "${pwd()}/sources:/src"
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} pyinstaller -F add2vals.py"
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"
                }
            }
        }
    }
}
