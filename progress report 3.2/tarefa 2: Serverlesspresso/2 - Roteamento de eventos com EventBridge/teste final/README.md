# Teste final

Vamos testar a lógica completa antes de configurar os frontends

## Preparação

Abra em seu navegador as seguintes abas:

* Console de Step Functions na máquina de estados OrderProcessorWorkflow
* Console de Step Functions na máquina de estados OrderManagerStateMachine
* Console EventBridge
* Console DynamoDB

## Criando um pedido simples

1. Vá até os barramentos de evento no EventBridge, escolha Serverlesspresso
2. Crie um novo evento
   1. Para a fonte do evento: ```awsserverlessda.serverlesspresso```
   2. Para o tipo de detalhe: ```Validator.NewOrder```
   3. E o seguinte JSON:
```JSON
{"userId":"1","orderId":"2"}
``` 
3. Na tabela ```serverlesspresso-order-table``` no DynamoDB, o item foi salvo com SK "2".

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/workflow%20started%20dynamoDB.png)

## Adicionando detalhes ao pedido

1. Na máquina de estados OrderManagerStateMachine, inicie a execução com o seguinte JSON

```JSON
{"action":"","body":{"userId":"1","drink":"Cappuccino","modifiers":[],"icon":"barista-icons_cappuccino-alternative"},"orderId":"2","baristaUserId":"3"}
```

2. A execução do OrderManager acontecerá de forma parecida com esta:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/teste%20final%203.png)

3. A execução do OrderProcessor iniciada anteriormente, graças ao evento causado pela OrderManager, está parada esperando que o barista informe que o pedido está pronto.

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/teste%20final%202.png)

4. Para que o barista assuma/de Claim no pedido precisamos executar novamente a máquina de estado OrderManager porém agora com outros dados de entrada

```JSON
{
  "action": "make",
  "body": {},
  "orderId": "2",
  "baristaUserId": "3"
}
```

5. A execução vai seguir este fluxo

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/teste%20final%204.png)

## Completando o pedido

1. Vá para a OrderManager
2. Inicie uma nova execução com este JSON

```JSON
{"action":"complete","body":{"userId":"1","drink":"Cappuccino","modifiers":[],"icon":"barista-icons_cappuccino-alternative"},"orderId":"2","baristaUserId":"3"}
```

3. A execução seguirá este fluxo

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/teste%20final%205.png)

4. A máquina de estados OrderProcessor seguirá este

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/teste%20final%206.png)

5. O item no banco de dados também será atualizado

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/teste%20final%207.png)