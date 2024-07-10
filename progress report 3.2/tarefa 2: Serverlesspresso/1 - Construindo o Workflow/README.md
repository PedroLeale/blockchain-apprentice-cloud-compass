# Construindo o Workflow

Cada pedido vai seguir os seguintes passos:

* O scan código QR inicia o processo de pedido.
* A aplicação verifica se a loja está aberta e se a fila do barista não está cheia. O barista pode lidar com até 20 bebidas de cada vez. Se a loja estiver fechada ou a fila estiver cheia, o processo de pedido é interrompido.
* A aplicação aguarda 5 minutos para o cliente especificar o pedido da bebida. Se nada acontecer após 5 minutos, o pedido expira.
* A aplicação aguarda 15 minutos para o barista preparar a bebida. Se nada acontecer após 15 minutos, o pedido expira.
* O pedido é finalmente concluído ou cancelado pelo barista.

Este tipo de lógica pode ser representado por uma máquina de estados, podemos usar AWS Step Functions para criar essa máquina.

## Sumário

- [Criando Step Functions](#criando-step-functions)
- [Verificar se a cafeteria está aberta](#verificar-se-a-cafeteria-está-aberta)
- [Diferentes fluxos de execução](#diferentes-fluxos-de-execução)
- [Checar capacidade](#checar-capacidade)
- [Esperar e enumerar pedidos](#esperar-e-enumerar-pedidos)
- [Workflow completo](#workflow-completo)
- [Teste](#teste)

## Criando Step Functions

1. No console de Step Functions na AWS, selecione Criar máquina de estado.

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/criar%20maquina%20de%20estado.png)

2. Escolha a opção "Blank".

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/escolher%20modelo.png)

3. Essa será a OrderProcessorWorkflow

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/order%20processor%20workflow.png)
Essa é apenas a base, podemos adicionar diversos estados que realizam diferentes operações de acordo com os dados de entrada.

## Verificar se a cafeteria está aberta

Faremos uma integração na Step Function para buscar um item em uma tabela DynamoDB.

1. Na máquina de estados, no menu a esquerda busque por DynamoDB GetItem
2. Arraste pra depois do start

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/get%20item%201.png)

1. No nome, coloque ```DynamoDB Get Shop status```
2. Nos parâmetros de API, na aba configurações, coloque:

```JSON
{
  "TableName": "serverlesspresso-config-table",
  "Key": {
    "PK": {
      "S": "config"
    }
  }
}
```

5. Na aba saída:
   1. Marque a opção: Adicione a entrada original à saída usando ResultPath - opcional
   2. Combine original input with result
   3. Na caixa de texto: ```$.GetStore```

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/get%20item%202.png)

## Diferentes fluxos de execução

Se a cafeteria estiver aberta, seguiremos um fluxo, se não, seguiremos outro.

1. No menu a esquerda, procure pela aba fluxo
2. Coloque um estado do tipo Choice depois do GetItem

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/is%20shop%20open.png)

3. Em uma das ramificações, coloque um EventBridge para quando estiver fechado, e apenas um "Pass" por enquanto na outra ramificação. O diagrama deve se parecer com este:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/is%20shop%20open%202.png)

4. Caso execute a máquina, com o JSON contendo ```"storeOpen": { "BOOL": true }``` é esperado o seguinte resultado:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/execucao%201.png)

5. Renomeie o EventBridge para "Shop not Ready" e cole o seguinte JSON no parâmetro de API

```JSON
{
  "Entries": [
    {
      "Detail": {
        "Message": "The Step functions workflow checks if the shop is open and has capacity to serve a new order by invoking a Lambda function that queries the Shop config service. The shop was not ready, and so a 'not ready' event is emitted to cancel the current order.",
        "userId.$": "$.detail.userId"
      },
      "DetailType": "OrderProcessor.ShopUnavailable",
      "EventBusName": "Serverlesspresso",
      "Source": "awsserverlessda.serverlesspresso"
    }
  ]
}
```

## Checar capacidade

1. Busque por ListExecutions no menu a esquerda
2. Caso a cafeteria esteja aberta, este será o novo fluxo de execução

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/list%20executions.png)

3. Nos parâmetros da API, cole o seguinte JSON, substituindo o ARN como necessário

```JSON
{
  "StateMachineArn": "YOUR_STATE_MACHINE_ARN",
  "MaxResults": 100,
  "StatusFilter": "RUNNING"
}
```

4. Na aba saída, marque de acordo com a imagem, inserindo ```$.isCapacityAvailable``` na caixa de texto

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/list%20executions%202.png)

5. Adicione mais um estado de escolha, após ListExecutions
6. Ramificaremos de acordo com o estado passado:
   1. Se estiver cheio, ou seja, o barista já tem 20 pedidos na fila ```$.isCapacityAvailable.Executions[20]```, redirecione para "Shop not Ready"
   2. Caso não esteja cheio, o fluxo segue normal para o próximo estado que será feito em seguida

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/is%20capacity%20available.png)

## Esperar e enumerar pedidos

Faremos agora os estados responsáveis por aguardar o cliente fazer o pedido e os enumeraremos.

1. Na ramificação em que a cafeteria está aberta e tem capacidade de realizar pedidos, adicione mais um EventBridge: PutEvents e o nomeie Emit - Workflow Started TT
2. Nas configurações:
   1. Selecione o checkbox Aguardar retorno de chamada - opcional
   2. Nos parâmetros de API cole o seguinte JSON:
```JSON
{
  "Entries": [
    {
      "Detail": {
        "Message": "The workflow waits for your order to be submitted. It emits an event with a unique 'task token'. The token is stored in an Amazon DynamoDB table, along with your order ID.",
        "TaskToken.$": "$$.Task.Token",
        "orderId.$": "$.detail.orderId",
        "userId.$": "$.detail.userId"
      },
      "DetailType": "OrderProcessor.WorkflowStarted",
      "EventBusName": "Serverlesspresso",
      "Source": "awsserverlessda.serverlesspresso"
    }
  ]
}
```

3. Na saída marque "Adicionar a entrada original à saída usando ResultPath - opcional" e escolha "Discard result and keep original input"
4. Na parte tratamento de erros, adicione um agente de captura/Catch da seguinte forma:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/tratamento%20de%20erros%20workflow%20started.png)

5. Escolha para inserir heartbeat em segundos, para cada 900 segundos (15 minutos)
6. Caso caia no catch, adicione um estado pass com a seguinte saída:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/CustomerTimeout.png)

7. Este estado levará para um EventBridge PutEvents, chamado Emit - error timeout, com o seguinte parâmetro de API:

```JSON
{
  "Entries": [
    {
      "Detail": {
        "Message": "The order timed out. Step Functions waits a set amount of time (5 minutes for a customer, 15 minutes for a barista), no action was taken and so the order is ended.",
        "userId.$": "$.detail.userId",
        "orderId.$": "$.detail.orderId",
        "cause.$": "$.cause"
      },
      "DetailType": "OrderProcessor.OrderTimeOut",
      "EventBusName": "Serverlesspresso",
      "Source": "awsserverlessda.serverlesspresso"
    }
  ]
}
```

8. Voltando no Emit - Workflow Started TT, caso não caia no Catch, seguiremos para o fluxo normal. Crie um novo estado DynamoDB: UpdateItem
9. O nomeie "Generate Order Number" e coloque o seguinte JSON nos parâmentros de API:

```JSON
{
  "TableName": "serverlesspresso-counting-table",
  "Key": {
    "PK": {
      "S": "orderID"
    }
  },
  "UpdateExpression": "set IDvalue = IDvalue + :val",
  "ExpressionAttributeValues": {
    ":val": {
      "N": "1"
    }
  },
  "ReturnValues": "UPDATED_NEW"
}
```

10. Nas opções de saída deste estado, teremos algo um pouco diferente:
    1. Coloque o seguinte JSON na opção de transformar o resultado com ResultSelector
    2. Combine o input original com o resultado: ```$.Order.Payload```
```JSON
{
  "orderNumber.$": "$.Attributes.IDvalue.N"
}
``` 

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/generate%20order%20number%20output.png)

11. De acordo com o retorno do Generate Order Number, emitiremos um evento novamente com EventBridge: PutEvents
    1. O nomeie Emit - Awaiting Completion TT
    2. Clique para Aguardar o retorno de chamada
    3. Na saída marque Adicione a entrada original à saída usando ResultPath - opcional
    4. Também na saída, escolha "Combine original input with result" e coloque ```$.order``` na caixa de texto
    5. Coloque o seguinte JSON nos parâmetros da API

```JSON
{
  "Entries": [
    {
      "Detail": {
        "Message": "You pressed 'submit order'. The workflow resumes using the stored 'task token', it generates your order number. It then pauses again, emitting an event with a new 'task token'.",
        "TaskToken.$": "$$.Task.Token",
        "orderId.$": "$.detail.orderId",
        "orderNumber.$": "$.Order.Payload.orderNumber",
        "userId.$": "$.detail.userId"
      },
      "DetailType": "OrderProcessor.WaitingCompletion",
      "EventBusName": "Serverlesspresso",
      "Source": "awsserverlessda.serverlesspresso"
    }
  ]
}
```

12. Assim como o PutEvents passado, este também terá tratamento de erros e heartbeat de 900 segundos

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/awaiting%20completion.png)

13. O catch deste estado será o Barista Timeout, que é bem parecido com o Customer Timeout, até leva no mesmo evento de erro

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/barista%20timeout.png)

14. Para o fluxo sem erros, apenas um estado de passagem que leva a um último evento "Emit - order Finished"
    1.  Neste não precisa marcar o checkbox de aguardar retorno de chamada
    2.  Cole o seguinte JSON nos parâmetros de API

```JSON
{
  "Entries": [
    {
      "Detail": {
        "Message": "The order has reached the end of the workflow, and so a final event is emitted to alert other services to this.",
        "userId.$": "$.detail.userId",
        "orderId.$": "$.detail.orderId"
      },
      "DetailType": "OrderProcessor.orderFinished",
      "EventBusName": "Serverlesspresso",
      "Source": "awsserverlessda.serverlesspresso"
    }
  ]
}
```

## Workflow completo

O workflow completo por enquanto deve estar se parecendo com este:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/workflow%20completo.png)

## Teste

Vamos realizar um teste do fluxo de execução para cafeteria fechada. Para a cafeteria aberta ainda precisamos implementar mais integrações com outros serviços.

1. Na página das tabelas do DynamoDB, selecione serverlesspresso-config-table, e abra o item config
2. Marque como falso o atributo storeOpen

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/dynamoDB%20loja%20fechada.png)

3. Agora inicie a execução da máquina de estado que criamos e coloque o seguinte JSON

```JSON
{
    "detail": {
      "orderId": "1",
      "userId": "testuser"
    }
}
```

4. O espero é a execução ser parecida com esta:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/workflow/execucao%20cafeteria%20fechada.png)