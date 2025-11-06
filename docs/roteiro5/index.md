# KUBERNETES

## Repositórios Utilizados

| Repositório                                                            | Descrição                            |
| ---------------------------------------------------------------------- | ------------------------------------ |
| [Account-Service](https://github.com/pma2025/pma252.account-service)   | Serviço de contas de usuário         |
| [Auth-Service](https://github.com/pma2025/pma252.auth-service)         | Serviço de autenticação              |
| [Gateway-Service](https://github.com/pma2025/pma252.gateway-service)   | API Gateway                          |
| [Product-Service](https://github.com/pma2025/pma252.product-service)   | Serviço de produtos                  |
| [Order-Service](https://github.com/pma2025/pma252.order-service)       | Serviço de pedidos                   |
| [Exchange-Service](https://github.com/pma2025/pma252.exchange-service) | Serviço de câmbio                    |
| —                                                                      | Banco de dados (Postgres, gitignore) |

---

## Estrutura do Projeto

```
📁 api/
├── 📁 account-service/
│   ├── 📄 Jenkinsfile
│   ├── 📄 Dockerfile
│   ├── 📁 k8s/
│   │   └── 📄 k8s.yaml
│   └── 📁 src/
├── 📁 auth-service/
│   ├── 📄 Jenkinsfile
│   ├── 📄 Dockerfile
│   ├── 📁 k8s/
│   │   └── 📄 k8s.yaml
│   └── 📁 src/
├── 📁 gateway-service/
│   ├── 📄 Jenkinsfile
│   ├── 📄 Dockerfile
│   ├── 📁 k8s/
│   │   └── 📄 k8s.yaml
│   └── 📁 src/
├── 📁 product-service/
│   ├── 📄 Jenkinsfile
│   ├── 📄 Dockerfile
│   ├── 📁 k8s/
│   │   └── 📄 k8s.yaml
│   └── 📁 src/
└── 📁 order-service/
    ├── 📄 Jenkinsfile
    ├── 📄 Dockerfile
    ├── 📁 k8s/
    │   └── 📄 k8s.yaml
    └── 📁 src/
```

---

## Configurações do Kubernetes

Os serviços foram configurados com Deployments e Services separados, utilizando boas práticas de isolamento e gerenciamento de variáveis de ambiente por meio de **ConfigMaps** e **Secrets**.

=== "Account Service"

```yaml
# k8s/k8s.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: account
spec:
  replicas: 1
  selector:
    matchLabels:
      app: account
  template:
    metadata:
      labels:
        app: account
    spec:
      containers:
        - name: account
          image: luabatepietro/account:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
            - name: POSTGRES_DB
              valueFrom:
                configMapKeyRef:
                  name: postgres-configmap
                  key: POSTGRES_DB
            - name: DATABASE_USERNAME
              valueFrom:
                secretKeyRef:
                  name: postgres-secrets
                  key: POSTGRES_USER
            - name: DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secrets
                  key: POSTGRES_PASSWORD
            - name: DATABASE_URL
              value: "jdbc:postgresql://postgres:5432/$(POSTGRES_DB)"
          resources:
            requests:
              memory: "200Mi"
              cpu: "50m"
            limits:
              memory: "300Mi"
              cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: account
  labels:
    app: account
spec:
  type: ClusterIP
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: account
```

=== "Auth Service"

```yaml
# k8s/k8s.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
        - name: auth
          image: luabatepietro/auth:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
            - name: POSTGRES_DB
              valueFrom:
                configMapKeyRef:
                  name: postgres-configmap
                  key: POSTGRES_DB
            - name: DATABASE_USERNAME
              valueFrom:
                secretKeyRef:
                  name: postgres-secrets
                  key: POSTGRES_USER
            - name: DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secrets
                  key: POSTGRES_PASSWORD
            - name: DATABASE_URL
              value: "jdbc:postgresql://postgres:5432/$(POSTGRES_DB)"
          resources:
            requests:
              memory: "200Mi"
              cpu: "50m"
            limits:
              memory: "300Mi"
              cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: auth
  labels:
    app: auth
spec:
  type: ClusterIP
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: auth
```

=== "Gateway Service"

```yaml
# k8s/k8s.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gateway
  template:
    metadata:
      labels:
        app: gateway
    spec:
      containers:
        - name: gateway
          image: luabatepietro/gateway:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "200Mi"
              cpu: "50m"
            limits:
              memory: "300Mi"
              cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: gateway
  labels:
    app: gateway
spec:
  type: LoadBalancer
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: gateway
```

=== "Product Service"

```yaml
# k8s/k8s.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product
  template:
    metadata:
      labels:
        app: product
    spec:
      containers:
        - name: product
          image: luabatepietro/product:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
            - name: POSTGRES_DB
              valueFrom:
                configMapKeyRef:
                  name: postgres-configmap
                  key: POSTGRES_DB
            - name: DATABASE_USERNAME
              valueFrom:
                secretKeyRef:
                  name: postgres-secrets
                  key: POSTGRES_USER
            - name: DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secrets
                  key: POSTGRES_PASSWORD
            - name: DATABASE_URL
              value: "jdbc:postgresql://postgres:5432/$(POSTGRES_DB)"
            - name: SPRING_CACHE_TYPE
              value: redis
            - name: SPRING_DATA_REDIS_HOST
              value: redis
            - name: SPRING_DATA_REDIS_PORT
              value: "6379"
          resources:
            requests:
              memory: "200Mi"
              cpu: "50m"
            limits:
              memory: "300Mi"
              cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: product
  labels:
    app: product
spec:
  type: ClusterIP
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: product
```

=== "Order Service"

```yaml
# k8s/k8s.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order
  template:
    metadata:
      labels:
        app: order
    spec:
      containers:
        - name: order
          image: luabatepietro/order:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
            - name: POSTGRES_DB
              valueFrom:
                configMapKeyRef:
                  name: postgres-configmap
                  key: POSTGRES_DB
            - name: DATABASE_USERNAME
              valueFrom:
                secretKeyRef:
                  name: postgres-secrets
                  key: POSTGRES_USER
            - name: DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secrets
                  key: POSTGRES_PASSWORD
            - name: DATABASE_URL
              value: "jdbc:postgresql://postgres:5432/$(POSTGRES_DB)"
          resources:
            requests:
              memory: "200Mi"
              cpu: "50m"
            limits:
              memory: "300Mi"
              cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: order
  labels:
    app: order
spec:
  type: ClusterIP
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: order
```

---

## Foto

![K8S](./imgs/k8s.png)
