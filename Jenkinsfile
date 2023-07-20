pipeline {
    agent none 
    stages {
        stage('Build') { 
            agent {
                docker {
                    image 'python:3.12.0b4-slim-bullseye' 
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py' 
                stash(name: 'compiled-results', includes: 'sources/*.py*') 
                archiveArtifacts artifacts: 'compiled-results', fingerprint: true
            }
        }
        stage('Test') { 
                agent {
                    docker {
                        image 'qnib/pytest' 
                    }
                }
                steps {
                    sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py' 
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
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:latest'
            }
            steps {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}

