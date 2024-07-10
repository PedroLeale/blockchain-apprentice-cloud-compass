# Serverlesspresso

Workshop para construir uma aplicação serverless de uma cafeteria. Foi utilizada a stack Serverlesspresso disponibilizada no workshop.

## Sumário

- [Como funciona a cafeteria?](#como-funciona-a-cafeteria)
- [Estrutura da aplicação](#estrutura-da-aplicação)
  - [Frontends](#frontends)
  - [Backend](#backend)
- [Diagrama da arquitetura](#diagrama-da-arquitetura)
- [Módulos](#módulos)
- [Atenção](#atenção)

## Como funciona a cafeteria?

O processo de pedido na cafeteria é o seguinte:

* Temos um código QR que muda a cada 5 minutos. Os clientes escaneiam esse código QR para fazer um pedido usando seu dispositivo móvel. O código QR é válido para 10 bebidas no período de 5 minutos e desaparece da tela quando não há mais bebidas disponíveis. Isso ajuda a evitar que os baristas fiquem sobrecarregados com pedidos!
* O cliente faz o pedido no aplicativo web carregado pelo código QR. O backend valida o pedido, cria um número de pedido e o disponibiliza para os baristas.
* Os baristas veem o pedido aparecer em seu próprio aplicativo. Eles podem alterar o status do pedido para indicar quando está sendo preparado, quando está concluído ou se precisam cancelar o pedido.
* O cliente vê todas as atualizações do barista em seu dispositivo móvel. Também exibimos o status dos pedidos futuros e concluídos.

## Estrutura da aplicação

### Frontends

Os frontends já estão implantados. Depois de construir o backend, você fornecerá variáveis de ambiente para os frontends para permitir que eles se conectem. Os três frontends são:

* __Aplicativo de exibição__: Este é exibido nos monitores suspensos (no contexto do evento). Ele fornece um código de barras para os clientes escanearem para fazer um pedido e mostra uma fila em tempo real de pedidos de bebidas futuras e concluídas.
* __Aplicativo de barista__: Este é executado em tablets usados pelos baristas. O aplicativo permite que os baristas alterem o status de um pedido de bebida ou cancelem o pedido, se necessário. As atualizações deste aplicativo são propagadas para os outros aplicativos.
* __Aplicativo de pedido__: Este é usado pelos clientes para fazer um pedido. Ele é projetado para ser executado em dispositivos móveis. Durante o teste de hoje, você usará seu dispositivo móvel com este aplicativo para fazer pedidos.

### Backend

A arquitetura da aplicação backend usa AWS Step Functions, Amazon EventBridge, AWS Lambda, Amazon API Gateway, Amazon S3, Amazon DynamoDB e Amazon Cognito. <br>
O JavaScript é executado no aplicativo do navegador frontend, enviando e recebendo dados de uma API backend construída usando API Gateway. O DynamoDB fornece uma camada de armazenamento de dados persistente usada pela API. Os eventos são roteados de volta para os aplicativos frontend usando AWS IoT Core e Lambda.

## Diagrama da arquitetura

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/diagrama.png)

## Módulos

| Módulo       | Feature                        | Descrição                                                                                            |
|---------------|--------------------------------|-----------------------------------------------------------------------------------------------------|
| 1            | [Construindo o Workflow](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/1%20-%20Construindo%20o%20Workflow/README.md) | Construa o Workflow com Step Functions                                                                       |
| 2             | [Roteamento de eventos com EventBridge](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/2%20-%20Roteamento%20de%20eventos%20com%20EventBridge/README.md) | Usando eventos para se comunicar entre diferentes microsserviços.                            |
| 3             | [Configurando os frontends](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/3%20-%20Configurando%20os%20frontends/README.md)       | Construa um serviço que envie mensagens de volta para os frontends com uma conexão websocket aberta. |


## Atenção

Após terminar este workshop não se esqueça de fazer um cleanup, isto é, desativar e/ou apagar os serviços utilizados para evitar custos desnecessários na sua conta AWS.