pipeline {
    agent {
        label 'Jenkins-Agent'
    }

    environment {
        APP_NAME = 'register-app-pipeline'
        RELEASE = '1.0.0'
        DOCKER_USER = 'sureshit'
        DOCKER_IMAGE = "${DOCKER_USER}/${APP_NAME}"
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/SureshIT28/CI_CD-piepline.git'
            }
        }

        stage('Java Version') {
            steps {
                sh 'java -version'
            }
        }

        stage('Maven Version') {
            steps {
                sh '/home/ec2-user/tools/hudson.tasks.Maven_MavenInstallation/Maven3/bin/mvn -version'
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                    export PATH=$JAVA_HOME/bin:$PATH
                    /home/ec2-user/tools/hudson.tasks.Maven_MavenInstallation/Maven3/bin/mvn clean compile
                '''
            }
        }

        stage('Test Application') {
            steps {
                sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                    export PATH=$JAVA_HOME/bin:$PATH
                    /home/ec2-user/tools/hudson.tasks.Maven_MavenInstallation/Maven3/bin/mvn test
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(
                    installationName: 'Sonarqube2-server',
                    credentialsId: 'jenkins-sonarqube-tokens'
                ) {
                    sh '''
                        export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                        export PATH=$JAVA_HOME/bin:$PATH

                        /home/ec2-user/tools/hudson.tasks.Maven_MavenInstallation/Maven3/bin/mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.3.0.6276:sonar
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate(
                        abortPipeline: false,
                        credentialsId: 'jenkins-sonarqube-tokens'
                    )
                }
            }
        }

        stage('Package Application') {
            steps {
                sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                    export PATH=$JAVA_HOME/bin:$PATH

                    /home/ec2-user/tools/hudson.tasks.Maven_MavenInstallation/Maven3/bin/mvn package -DskipTests
                '''
            }
        }

        // NEW: Docker Build
        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${RELEASE}-${BUILD_NUMBER} .
                '''
            }
        }

        // NEW: Docker Push
        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'sureshit!',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${RELEASE}-${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline execution failed!'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
