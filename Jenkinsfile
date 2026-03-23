pipeline {
    agent any

    // 7. Добавление параметров (Parameters)
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Целевое окружение для деплоя')
        string(name: 'APP_VERSION', defaultValue: '1.0.0-SNAPSHOT', description: 'Версия приложения')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Пропустить этап тестирования')
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Ветка репозитория для сборки')
    }

    tools {
        jdk 'JDK-17'
        maven 'Maven-3.9.8'
    }

    environment {
        GIT_REPO = 'https://github.com/vggalkin/spring-petclinic.git'
        RECIPIENT_EMAIL = 'i@galkin-v.ru'
        // Используем параметр ветки в переменную окружения для удобства
        ACTIVE_BRANCH = "${params.GIT_BRANCH}"
    }

    stages {
        // 4 & 5. Этап Checkout
        stage('📥 Checkout') {
            steps {
                git branch: "${ACTIVE_BRANCH}",
                    url: "${GIT_REPO}",
                    changelog: true,
                    poll: false
            }
        }

        // 4 & 5. Этап Build
        stage('🔨 Build') {
            steps {
                script {
                    // 8. Обработка ошибок (Error Handling) - явная проверка статуса
                    echo "Starting build for version ${params.APP_VERSION}..."
                    def buildStatus = sh(script: 'mvn -B clean compile', returnStatus: true)
                    
                    if (buildStatus != 0) {
                        echo "⚠️ Ошибка компиляции! Код возврата: ${buildStatus}"
                        // Дополнительные действия перед падением (логирование, очистка)
                        sh 'echo "Cleaning up failed build artifacts..."'
                        error "Build step failed with exit code ${buildStatus}"
                    } else {
                        echo "✅ Build completed successfully."
                    }
                }
            }
        }

        // 4 & 5. Этап Test
        stage('🧪 Test') {
            // 6. Условия выполнения (When) - только если не пропущены тесты
            when {
                expression { return !params.SKIP_TESTS }
            }
            steps {
                sh 'mvn -B test'
            }
            post {
                always {
                    junit testResults: 'target/surefire-reports/*.xml',
                        allowEmptyResults: true,
                        skipPublishingChecks: true
                }
            }
        }

        // 4 & 5. Этап Deploy
        stage('🚀 Deploy') {
            when {
                expression { return params.GIT_BRANCH == 'main' }
            }
            steps {
                script {
                    echo "🚀 Deploying version ${params.APP_VERSION} to ${params.DEPLOY_ENV}..."
                    sh '''
                    echo "Connecting to server..."
                    echo "Uploading artifact..."
                    echo "Deployment finished."
                    '''
           }
        }
    }
    post {
        always {
            // Очистка рабочего пространства (опционально)
            cleanWs()
        }
        success {
            mail to: "${RECIPIENT_EMAIL}",
                from: "i@galkin-v.ru",
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build succeeded!
                Job: ${env.JOB_NAME} | Build: #${env.BUILD_NUMBER}
                Branch: ${ACTIVE_BRANCH}
                Version: ${params.APP_VERSION}
                Environment: ${params.DEPLOY_ENV}
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
                Branch: ${ACTIVE_BRANCH}
                Version: ${params.APP_VERSION}
                URL: ${env.BUILD_URL}console
                """
        }
    }
}

