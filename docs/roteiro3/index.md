### Exchange API

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
        gateway e1@==> exchange:::red
        gateway --> product
        gateway --> order
        product --> db
        order --> db
        order --> product
    end
    exchange e3@==> 3partyapi:::green@{label: "3rd-party API"}
    internet e2@==>|request| gateway
    e1@{ animate: true }
    e2@{ animate: true }
    e3@{ animate: true }
    classDef red fill:#fcc
    classDef green fill:#cfc
    click product "#product-api" "Product API"
```

## Tarefas:

Implementar um microserviço implementado em **FastAPI**, que consulta um provedor externo de câmbio e aplica regras de spread.

- essa API usa o [ExchangeRate-API](https://www.exchangerate-api.com/) para pegar o preço atual das trocas entre moedas (câmbio)

## Estrutura do projeto

### Exchange-Service

```tree
api/
    exchange-service/
        app/
            __init__.py
            main.py
            auth.py
            config.py
            models.py
            clients/
                __init__.py
                rates.py
        requirements.txt
        Dockerfile
```

## Endpoint:

!!! info "GET/exchange/{from}/{to}"

    Pega a taxa de câmbio de uma moeda para outro.

    === "Response"

        ``` { .json .copy .select linenums='1' }
            {
                "sell": 0.82,
                "buy": 0.80,
                "date": "2021-09-01 14:23:42",
                "id-account": "0195ae95-5be7-7dd3-b35d-7a7d87c404fb"
            }
        ```
        ```bash
        Response code: 200 (ok)
        ```

## Repositorios:

[Exchange-Service](https://github.com/pma2025/pma252.exchange-service)

## Conclusão

Cada enxadada uma minhoca.