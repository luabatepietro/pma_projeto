### Order API

- feito por Lucas Abatepietro

## Estrutura da requisição

```mermaid
flowchart LR
    subgraph api [Trusted Layer]
        direction TB
        gateway --> account
        gateway --> auth
        account --> db@{ shape: cyl, label: "Database" }
        auth --> account
        gateway --> product
        gateway e6@==> order:::red
        product --> db
        order e3@==> db
        order e4@==> product
    end
    internet e1@==>|request| gateway
    e1@{ animate: true }
    e3@{ animate: true }
    e4@{ animate: true }
    e6@{ animate: true }
    classDef red fill:#fcc
    click order "#order-api" "Order API"
```

## Tarefas

Implementar um microserviço **ORDER** que tenha os seguintes requisitos:

- `POST/order`: cria um pedido
- `GET/order`: listar todos os pedidos
- `GET/order/{id}` busca pedido por ID

Ela segue o padrão adotado no projeto: interface (order) e service (order-service) atrás do gateway e protegidos por JWT.

## Estrutura do projeto

### Order

```
📁 api/
└── 📁 order/
    ├── 📁 src/
    │   └── 📁 main/
    │       └── 📁 java/
    │           └── 📁 store/
    │               └── 📁 order/
    │                   ├── 📄 OrderController.java
    │                   ├── 📄 OrderIn.java
    │                   ├── 📄 OrderOut.java
    │                   ├── 📄 OrderItemIn.java
    │                   └── 📄 OrderItemOut.java
    └── 📄 pom.xml

```

### Order-service

```
📁 api/
└── 📁 order-service/
    ├── 📁 src/
    │   └── 📁 main/
    │       ├── 📁 java/
    │       │   └── 📁 store/
    │       │       └── 📁 order/
    │       │           ├── 📄 Order.java
    │       │           ├── 📄 OrderItem.java
    │       │           ├── 📄 OrderApp.java
    │       │           ├── 📄 OrderModel.java
    │       │           ├── 📄 OrderParser.java
    │       │           ├── 📄 OrderRepo.java
    │       │           ├── 📄 OrderResource.java
    │       │           ├── 📄 OrderService.java
    │       │           └── 📄 FeignAuthInter.java
    │       └── 📁 resources/
    │           ├── 📄 application.yaml
    │           └── 📁 db/
    │               └── 📁 migration/
    │                   ├── 📄 schema.sql
    │                   ├── 📄 create_table.sql
    │                   └── 📄 orderitem.sql
    ├── 📄 pom.xml
    └── 📄 Dockerfile
```

## Order API

Foram implementados os seguintes endpoints com request body e response da seguinte forma:

!!! info "POST /order"

    Cria um novo order 

    === "Request"

        ``` { .json .copy .select linenums='1' }
        {
            "items": [
                {
                    "idProduct": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8",
                    "quantity": 2
                },
                {
                    "idProduct": "0195abfe-e416-7052-be3b-27cdaf12a984",
                    "quantity": 1
                }
            ]
        }
        ```

    === "Response"

        ``` { .json .copy .select linenums='1' }
        {
            "id": "0195ac33-73e5-7cb3-90ca-7b5e7e549569",
            "date": "2025-09-01T12:30:00",
            "items": [
                {
                    "id": "01961b9a-bca2-78c4-9be1-7092b261f217",
                    "product": {
                        "id": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8"
                    },
                    "quantity": 2,
                    "total": 20.24
                },
                {
                    "id": "01961b9b-08fd-76a5-8508-cdb6cd5c27ab",
                    "product": {
                        "id": "0195abfe-e416-7052-be3b-27cdaf12a984"
                    },
                    "quantity": 10,
                    "total": 6.2
                }
            ],
            "total": 26.44
        }
        ```
        ```bash
        Response code: 201 (created)
        Response code: 400 (bad request), if the product does not exist.
        ```

!!! info "GET /order"

    Pega todos os orders

    === "Response"

        ``` { .json .copy .select linenums='1' }
        [
            {
                "id": "0195ac33-73e5-7cb3-90ca-7b5e7e549569",
                "date": "2025-09-01T12:30:00",
                "total": 26.44
            },
            {
                "id": "0195ac33-cbbd-7a6e-a15b-b85402cf143f",
                "date": "2025-10-09T03:21:57",
                "total": 18.6
            }

        ]
        ```
        ```bash
        Response code: 200 (ok)
        ```

!!! info "GET /order/{id}"

    pega order por {id}
    
    === "Response"

        ``` { .json .copy .select linenums='1' }
        {
            "id": "0195ac33-73e5-7cb3-90ca-7b5e7e549569",
            "date": "2025-09-01T12:30:00",
            "items": [
                {
                    "id": "01961b9a-bca2-78c4-9be1-7092b261f217",
                    "product": {
                        "id": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8",
                    },
                    "quantity": 2,
                    "total": 20.24
                },
                {
                    "id": "01961b9b-08fd-76a5-8508-cdb6cd5c27ab",
                    "product": {
                        "id": "0195abfe-e416-7052-be3b-27cdaf12a984",
                    },
                    "quantity": 10,
                    "total": 6.2
                }
            ],
            "total": 26.44
        }
        ```
        ```bash
        Response code: 200 (ok)
        Response code: 404 (not found), if the order does not belong to the current user.
        ```