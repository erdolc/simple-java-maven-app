pipeline {
    agent {
        kubernetes {
            inheritFrom 'default'
            yaml '''
            spec:
                containers:
                  - name: maven
                    image: 'maven:3-alpine'
                    command:
                      - cat
                    tty: true
            '''
            defaultContainer 'maven'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') { 
            steps {
                sh 'mvn test' 
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
            }
        }
    }
}
