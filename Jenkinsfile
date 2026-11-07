pipeline {
    agent {
        label 'Jenkins-Agent'
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
                sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                    export PATH=$JAVA_HOME/bin:$PATH
                    java -version
                '''
            }
        }

        stage('Maven Version') {
            steps {
                sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                    export PATH=$JAVA_HOME/bin:$PATH
                    /home/ec2-user/tools/hudson.tasks.Maven_MavenInstallation/Maven3/bin/mvn -version
                '''
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
                script {
                    withSonarQubeEnv(
                        installationName: 'Sonarqube2-server',
                        credentialsId: 'jenkins-sonarqube-tokens'
                    ) {
                        sh '''
                            export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                            export PATH=$JAVA_HOME/bin:$PATH
                            /home/ec2-user/tools/hudson.tasks.Maven_MavenInstallation/Maven3/bin/mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar
                        '''
                    }
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
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline execution failed!'
        }

        always {
            echo 'Pipeline execution completed!'
        }
    }
}
