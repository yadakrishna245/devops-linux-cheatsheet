# Day 13 — Jenkins CI/CD 🔄

## LinkedIn Post:

```
🔄 [Day 13/15] DevOps Cheat Sheet: Jenkins CI/CD

"Can you set up the pipeline?"

Here's a production-ready Jenkinsfile you can
copy-paste and modify. I've used this pattern
across dozens of projects:

━━━━━━━━━━━━━━━━━

pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        APP_NAME = 'myapp'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/org/repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm ${APP_NAME}:${BUILD_NUMBER} npm test'
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login ${DOCKER_REGISTRY} -u $USER --password-stdin'
                    sh 'docker push ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}'
            }
        }
    }

    post {
        failure {
            slackSend channel: '#alerts',
                message: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        success {
            slackSend channel: '#deploys',
                message: "✅ Deployed: ${APP_NAME}:${BUILD_NUMBER}"
        }
    }
}

━━━━━━━━━━━━━━━━━

💡 Pro tip: NEVER put credentials in your Jenkinsfile.

Use `withCredentials` + Jenkins Credential Store.
Your secrets stay encrypted, rotatable, and auditable.

Also: Tag images with BUILD_NUMBER, not "latest".
You need to know EXACTLY what's running in production.

♻️ Repost for your CI/CD community
#Jenkins #CICD #DevOps #Pipeline #Automation #SRE
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Complete CI/CD to K8s pattern — star for reference!

Tomorrow: Security Hardening — lock down your servers 🔐
```
