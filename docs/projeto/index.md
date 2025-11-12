# Teste de carga - 20%

Vamos ver se nossa aplicação é capaz de aguentar com um trafego que aumenta cada vez mais.
> Abaixo a um GIF mostrando como o nosso serviço é capaz de escalar conforme a demanda.

![GIF.AQUI](<Vídeo sem título ‐ Feito com o Clipchamp (4).gif>)

O que estamos fazendo nesse vídeo criamos três terminais: 

1. Cria o HPA (Horizontal Pod Autoscaler) para o ```gateway```. Ele vai aumentar o num de **pods** baseado em quanto de CPU está sendo usado.
2. Esse terminal serve para monitorar os **pods** (ou seja, para vermos se está sendo criado mais)
3. Nesse que está sendo rodado o teste de carga, mandando bastante trafégo para nossa aplicação ```gateway```