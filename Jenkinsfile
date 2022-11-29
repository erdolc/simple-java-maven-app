pipeline {
    agent {
        kubernetes {
            inheritFrom 'default'
            yaml '''
            spec:
                containers:
                  - name: maven
                    image: 'maven:3.8-jdk-8-slim'
                    command:
                    - cat
                    tty: true
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:debug
                    command:
                    - /busybox/cat
                    tty: true
                    volumeMounts:
                    - name: kaniko-secret
                      mountPath: /kaniko/.docker
                    volumes:
                    - name: kaniko-secret
                      secret:
                        secretName: registry-credentials
                        items:
                        - key: .dockerconfigjson
                          path: config.json
            '''
            defaultContainer 'maven'
        }
    }
    stages {
        stage('Maven Test') { 
            steps {
                sh 'mvn test' 
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Docker Build') {
            agent { 
                kubernetes {
                    defaultContainer 'kaniko'
                }
            }
            steps {
                script {
                    sh "/kaniko/executor --dockerfile `pwd`/Dockerfile --context `pwd` --destination=docker.olcay.net/app:${env.BUILD_ID}"
                }
            }
        }
    }
}
