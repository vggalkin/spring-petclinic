pipeline {
    agent any
    
    tools {
        jdk 'JDK-17'
        maven 'Maven-3.9.8'
    }
    
    environment {
        // ✅ URL без лишних пробелов!
        GIT_REPO = 'https://github.com/vggalkin/spring-petclinic.git'
        GIT_BRANCH = 'main'
        RECIPIENT_EMAIL = 'i@galkin-v.ru'
    }
    
    stages {
        stage('📥 Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", 
                    url: "${GIT_REPO}",
                    changelog: true,
                    poll: false
            }
        }
        
        stage('🔨 Build & Test') {
            steps {
                sh 'mvn -B clean test'
            }
        }
    }
    
    post {
        always {
            junit testResults: 'target/surefire-reports/*.xml',
                  allowEmptyResults: true,
                  skipPublishingChecks: true  // Подавляем сообщение о Checks API
        }
        
        success {
            mail to: "${RECIPIENT_EMAIL}",
                 from: "i@galkin-v.ru",
                 subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """
                    Build succeeded!
                    Job: ${env.JOB_NAME} | Build: #${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}
                    Test Report: ${env.BUILD_URL}testReport/
                 """
        }
        
        failure {
            mail to: "${RECIPIENT_EMAIL}",
                 from: "i@galkin-v.ru",
                 subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """
                    Build failed!
                    Job: ${env.JOB_NAME} | Build: #${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}console
                 """
        }
    }
}
