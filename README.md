# poc-agy-jenkins
PoC Antigravity CLI inside Jenkins Pipeline

## Pipeline
This is a sample jenkins pipeline wher integrate antigravity CLI inside a stage to review our code. We must configure OPENAI_API_BASE and OPENAI_API_KEY

```shell
pipeline {
    agent {
        docker { 
            image 'node:22-alpine' 
            // Instalamos git en el contenedor alpine para que Jenkins pueda hacer el clone
            args '-u root:root' 
        }
    }
    environment {
        // Enrutamiento a tu infraestructura local (LiteLLM/Ollama)
        OPENAI_API_BASE = 'http://tu-servidor-litellm:4000'
        OPENAI_API_KEY = credentials('litellm-local-key')
    }
    stages {
        stage('Preparar Entorno') {
            steps {
                // Alpine base no trae git, es necesario para el step 'git' de Jenkins
                sh 'apk add --no-cache git'
            }
        }
        stage('Clonar Repositorio Ficticio') {
            steps {
                // Clonación del código fuente. Si fuera privado, añadirías: credentialsId: 'tus-credenciales-git'
                git branch: 'main', url: 'https://github.com/veradoc-demo/core-backend-ficticio.git'
                
                // Verificación rápida en los logs para confirmar que el código bajó correctamente
                sh 'ls -la'
            }
        }
        stage('Instalar Antigravity CLI') {
            steps {
                sh 'npm install -g @antigravity/cli'
                sh 'agy --version'
            }
        }
        stage('Code Review con agy (SDD Gate)') {
            steps {
                // El workspace actual (./) ya contiene el código clonado en el stage anterior
                sh '''
                agy "Analiza todo el código fuente del directorio actual. Compara la implementación con los requisitos técnicos definidos en el archivo 'spec.md' o 'constitution.md' de la raíz. Si detectas funciones sin tests unitarios, uso de dependencias no autorizadas, o violaciones de la arquitectura, finaliza el proceso inmediatamente con exit code 1. Si el código cumple la especificación al 100%, finaliza con exit code 0."
                '''
            }
        }
        stage('Build & Deploy (Simulado)') {
            steps {
                echo 'El código ha pasado la validación de la IA. Procediendo con el empaquetado...'
            }
        }
    }
    post {
        failure {
            echo '❌ Pipeline abortado: La IA de Antigravity detectó violaciones en el código (Code Review fallido).'
        }
        success {
            echo '✅ Pipeline completado: El código fue auditado por la IA y superó los estándares.'
        }
        cleanWs() // Limpia el workspace al finalizar para no dejar restos del repo en el nodo
    }
}
```
