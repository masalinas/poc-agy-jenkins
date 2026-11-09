# poc-agy-jenkins
PoC Antigravity CLI inside Jenkins Pipeline

## Pipeline
This is a sample jenkins pipeline wher integrate antigravity CLI inside a stage to review our code. We must configure GEMINI_API_KEY from out Google AI Studio:

```shell
pipeline {
    agent {
        docker { 
            image 'node:22-alpine' 
            args '-u root:root' 
        }
    }
    environment {
        // Inject the official Gemini key from Jenkins credentials manager
        GEMINI_API_KEY = credentials('gemini-api-key')
    }
    stages {
        stage('Prepare Environment') {
            steps {
                sh 'apk add --no-cache git'
            }
        }
        stage('Clone Dummy Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/veradoc-demo/core-backend-ficticio.git'
            }
        }
        stage('Install Antigravity CLI') {
            steps {
                sh 'npm install -g @antigravity/cli'
            }
        }
        stage('Code Review with agy (Gemini API)') {
            steps {
                // Execute agy. By detecting the GEMINI_API_KEY variable, 
                // it will use the Google engine. You can force the exact model if the CLI requires it.
                sh '''
                agy --model gemini-1.5-pro "Analyze all the source code in the current directory. Compare the implementation with the technical requirements defined in the 'spec.md' or 'constitution.md' file in the root. If you detect functions without unit tests, use of unauthorized dependencies, or architecture violations, exit the process immediately with exit code 1. If the code meets the specification 100%, exit with exit code 0."
                '''
            }
        }
        stage('Build & Deploy (Simulated)') {
            steps {
                echo 'Code has passed Gemini validation. Proceeding with packaging...'
            }
        }
    }
    post {
        failure {
            echo '❌ Pipeline aborted: Gemini detected code violations (Code Review failed).'
        }
        success {
            echo '✅ Pipeline completed: The code was audited by Gemini and passed all standards.'
        }
        cleanWs()
    }
}
```
