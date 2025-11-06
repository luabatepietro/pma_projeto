# JENKINS

A orquestração da esteira de **CI/CD** do domínio **store** é realizada pelo **Jenkins**, que executa dois tipos principais de _pipelines_:

---

## 1. Pipelines de Interfaces

_(Ex.: account, auth, product, order, …)_

- Responsáveis por empacotar **artefatos e contratos** utilizados por outros módulos.
- **Não** geram nem publicam imagens Docker.

---

## 2. Pipelines de Serviços

_(Ex.: account-service, auth-service, product-service, order-service, gateway-service, …)_

- Realizam o build da aplicação (Java ou Python).
- Executam o build e push de imagens Docker para o **Docker Hub**.
- Podem acionar pipelines de dependência, como a compilação prévia da interface correspondente.

---

## Status Atual do Pipeline

![Status do Pipeline](img/image.png)

---

## Estrutura do Projeto

```
📁 api/
├── 📁 jenkins/
│   ├── 📄 compose.yaml
│   └── 📁 config/
├── 📁 account/
│   ├── 📄 Jenkinsfile
│   ├── 📄 Dockerfile
│   └── 📁 src/
├── 📁 auth-service/
│   ├── 📄 Jenkinsfile
│   ├── 📄 Dockerfile
│   └── 📁 src/
├── 📁 gateway-service/
│   ├── 📄 Jenkinsfile
│   ├── 📄 Dockerfile
│   └── 📁 src/
├── 📁 product/
│   ├── 📄 Jenkinsfile
│   └── 📁 src/
└── 📁 order/
    ├── 📄 Jenkinsfile
    └── 📁 src/
```

---

## Jenkins Setup

```yaml
# docker compose up -d --build --force-recreate
name: ops

services:
  jenkins:
    container_name: jenkins
    build:
      dockerfile_inline: |
        FROM jenkins/jenkins:jdk21
        USER root

        # Install tools
        RUN apt-get update && apt-get install -y lsb-release iputils-ping maven

        # Install Docker
        RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc           https://download.docker.com/linux/debian/gpg
        RUN echo "deb [arch=$(dpkg --print-architecture)           signed-by=/usr/share/keyrings/docker-archive-keyring.asc]           https://download.docker.com/linux/debian           $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
        RUN apt-get update && apt-get install -y docker-ce

        # Install kubectl
        RUN apt-get install -y apt-transport-https ca-certificates curl
        RUN curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
        RUN chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
        RUN echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list
        RUN chmod 644 /etc/apt/sources.list.d/kubernetes.list
        RUN apt-get update && apt-get install -y kubectl

        RUN usermod -aG docker jenkins
    ports:
      - 9080:8080
    volumes:
      - ${CONFIG:-./config}/jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
```

---

## Exemplo de Pipeline de Serviço

**Jenkinsfile para `account-service`:**

```groovy
pipeline {
    agent any
    environment {
        SERVICE = 'account'
        NAME = "luabatepietro/${env.SERVICE}"
    }
    stages {
        stage('Dependecies') {
            steps {
                build job: 'account', wait: true
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Build & Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credential',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'TOKEN')])
                {
                    sh "docker login -u $USERNAME -p $TOKEN"
                    sh "docker buildx create --use --platform=linux/arm64,linux/amd64 --node multi-platform-builder-${env.SERVICE} --name multi-platform-builder-${env.SERVICE}"
                    sh "docker buildx build --platform=linux/arm64,linux/amd64 --push --tag ${env.NAME}:latest --tag ${env.NAME}:${env.BUILD_ID} -f Dockerfile ."
                    sh "docker buildx rm --force multi-platform-builder-${env.SERVICE}"
                }
            }
        }
    }
}
```

---

## Exemplo de Pipeline de Contrato

**Jenkinsfile para `contract`:**

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean install'
            }
        }
    }
}
```

---

## Conclusão

> Cada enxadada, uma minhoca. 🪱
