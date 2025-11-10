# Bottlenecks – Escalabilidade e Desempenho

Este documento consolida as estratégias de **balanceamento de carga** e **cache distribuído** utilizadas no ecossistema **store-api**, responsáveis por mitigar gargalos de desempenho e garantir **resiliência, disponibilidade e escalabilidade** no ambiente **Kubernetes (AWS EKS)**.

---

## 1. Load Balancer (AWS EKS)

### Visão Geral
O **Gateway Service** atua como ponto de entrada único para todas as requisições externas ao cluster.  
Ele é configurado com um **Service Kubernetes** do tipo `LoadBalancer`, instruindo o **EKS (Elastic Kubernetes Service)** a provisionar automaticamente um **Elastic Load Balancer (ELB)** na AWS.

Essa camada é essencial para distribuir o tráfego de forma equilibrada entre as instâncias dos microserviços, garantindo alta disponibilidade e evitando sobrecarga em um único nó.

---

### Fluxo de Tráfego

```text
Usuário
   ↓
Load Balancer
   ↓
Gateway Service
   ↓
 ├── Account
 ├── Auth
 ├── Order
 ├── Product
 └── Exchange
```

### Configuração do Gateway

O Gateway Service é implementado com recursos Kubernetes do tipo Deployment e Service, responsáveis por expor o ponto de entrada público e rotear as requisições internas.

Para detalhes completos sobre o deployment e o service, consulte a documentação da Gateway API.

2. Cache Distribuído (Redis)
- Visão Geral
    - O Product Service utiliza o Redis como camada de cache distribuído para reduzir a latência e o número de leituras diretas no banco de dados PostgreSQL.

Essa abordagem permite que respostas a consultas recorrentes sejam entregues rapidamente, diminuindo o uso de recursos e o tempo de resposta percebido pelo usuário final.

O cache é implementado por meio da abstração nativa do Spring Boot Cache, com gerenciamento centralizado pelo RedisCacheManager.

### Arquitetura do Cache
```text
Copiar código
          ┌────────────────────┐
          │ Client / Gateway   │
          └─────────┬──────────┘
                    │
                    ▼
          ┌────────────────────┐
          │ Product Service    │
          ├────────────────────┤
          │ Cache Hit  → Redis │
          │ Cache Miss → PostgreSQL │
          └────────────────────┘
```

### Estratégia de Cache
Tipo de Operação	Estratégia de Cache	TTL
findAll()	Cache da lista completa (products-list)	2 minutos
findById(id)	Cache individual (product-by-id)	10 minutos
create() / delete()	Evict automático das chaves afetadas

A implementação do cache é feita com as anotações @Cacheable e @CacheEvict, definindo o comportamento de leitura e invalidação dos dados.
Para mais detalhes, consulte a documentação da Product API.

## Redis no Kubernetes
#### Implantação
O Redis é executado em um Deployment dedicado dentro do cluster, com exposição interna por meio de um Service do tipo ClusterIP, acessível apenas por outros microserviços.

```yaml
Copiar código
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:latest
          ports:
            - containerPort: 6379
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "100m"
---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  type: ClusterIP
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis
O serviço Redis é resolvido via DNS interno (redis.store.svc.cluster.local) e acessado pelos microserviços utilizando:
```

### A arquitetura de Bottlenecks integra três pilares essenciais:

- **Elastic Load Balancer (ELB)**
    - distribuição inteligente de tráfego no nível de entrada, com resiliência automática no EKS.

- **Redis Cache**
    - camada de cache eficiente que reduz a latência e o consumo de banco de dados.
    - ClusterIP Interno — comunicação segura entre microserviços, isolada do tráfego externo.

**Essas estratégias combinadas resultam em:**
1. Redução significativa da latência média das APIs.

2. Maior resiliência em cenários de pico de carga.

3. Escalabilidade horizontal automatizada.
