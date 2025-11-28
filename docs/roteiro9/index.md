# PaaS

Neste projeto implementamos soluções de **PaaS (Plataforma como Serviço)** para abstrair a complexidade da infraestrutura e focar na lógica de negócio e na orquestração dos microsserviços.

### 1. Tecnologia: Amazon EKS
A principal utilização do modelo PaaS no projeto se deu através do **Amazon Elastic Kubernetes Service (EKS)**.

Embora o Kubernetes, por si só, seja um orquestrador de containers, ao utilizá-lo na modalidade gerenciada pela AWS (EKS), ele se enquadra na definição de PaaS (ou mais especificamente CaaS - *Container as a Service*), pois transfere para o provedor de nuvem a responsabilidade pelo gerenciamento de toda a camada de controle (*Control Plane*).

### 2. Como Utilizamos (Implementação Prática)

Utilizamos o PaaS (Amazon EKS) nas seguintes frentes:

* **Abstração do Control Plane:** Ao contrário de uma abordagem *IaaS* (onde teríamos que provisionar instâncias EC2, instalar o Linux, configurar o `etcd` e os servidores mestres do Kubernetes manualmente), utilizamos o EKS para nos entregar um cluster pronto. A AWS ficou responsável pela disponibilidade, *patching* de segurança e escalabilidade dos nós mestres.
* **Orquestração de Microsserviços:** Utilizamos a plataforma para fazer o *deploy* das nossas APIs (Product, Order, Exchange). O PaaS foi responsável por receber nossos manifestos (arquivos YAML de `Deployment` e `Service`) e garantir que o número desejado de réplicas (Pods) estivesse rodando, reiniciando automaticamente containers em caso de falha (Self-healing).
* **Exposição de Serviços:** Utilizamos os recursos nativos da plataforma para expor nossos microsserviços para a rede, permitindo a comunicação entre eles e o acesso externo, sem a necessidade de configurar balanceadores de carga manuais no nível do sistema operacional.

### 3. Porque Escolhemos Este Modelo de PaaS

A escolha por este modelo de PaaS atende diretamente aos tópicos estudados no curso, especificamente:

* **DevOps e Cloud Computing:** A utilização do EKS permitiu que o grupo adotasse práticas de *NoOps* na camada de infraestrutura base, focando os esforços de desenvolvimento na construção dos pipelines de CI/CD (Jenkins) e na qualidade do código das APIs.
* **Gestão de Níveis de Serviço (SLA):** Ao confiar a gestão do *Control Plane* à Amazon, herdamos o SLA de disponibilidade da AWS para a API do Kubernetes, garantindo uma arquitetura mais robusta para as aplicações de alto desempenho propostas no projeto.

---

### Resumo Técnico

| Tecnologia | Classificação no Projeto | Justificativa |
| :--- | :--- | :--- |
| **Amazon EKS** | **PaaS / CaaS** | A AWS gerencia o hardware, SO e o plano de controle do Kubernetes. O grupo apenas consome a API para rodar os workloads. |
| **Docker** | Ferramenta / Runtime | Utilizado como motor para empacotar as aplicações. É a base que roda *sobre* o PaaS. |
| **Jenkins** | Ferramenta / IaaS | Hospedado pelo grupo para orquestrar os pipelines. |
| **Redis** | Software (Container) | Rodado como container dentro do cluster (neste cenário, atua como componente da aplicação, não como serviço gerenciado externo). |