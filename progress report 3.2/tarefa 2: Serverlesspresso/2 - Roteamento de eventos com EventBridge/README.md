# Roteamento de eventos com EventBridge

Eventos são sinais de que o estado do sistema mudou de alguma forma. Na AWS, são representados como JSONs, neste módulo iremos aprender a como rotear eventos através de diferentes microserviços.

## Sumário

- [Log All](#log-all)
- [New Order](#new-order)
- [Workflow Started](#workflow-started)
- [Waiting Completion](#waiting-completion)
- [Teste final](#teste-final)

## Log All

Vamos criar e testar uma regra que registre todos os eventos que passam pelo barramento da aplicação no CloudWatch Logs.

### Criando a regra
1. Vá até o console EventBridge, selecione regras, então criar regra
2. Para o nome, ```logAll``` e para o Barramento de eventos ```Serverlesspresso```
3. Para a fonte do evento escolha "Outros" e para o Padrão de eventos o seguinte Json. Ignore o campo de evento de exemplo

```JSON
{
  "source": ["awsserverlessda.serverlesspresso"]
}
``` 

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/log%20all%202.png)

4. Para os destinos, selecione Serviço da AWS e o Grupo de logs do CloudWatch da seguinte maneira:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/log%20all%203.png)

5. Não se preocupe com as configurações de etiquetas
6. Ao revisar, tenha certeza que o barramento de eventos é o ```Serverlesspresso```

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/log%20all%204.png)

### Testando

1. Execute a máquina de estados criada no workflow com o seguinte JSON

```JSON
{
    "detail": {
      "orderId": "2",
      "userId": "testuser2"
    }
}
```

2. No console de execução, o status deve ser algo parecido com o da seguinte imagem

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/execucao%20logAll.png)

3. No grupo de logs do CloudWatch, ao buscar pelo serverlesspressoEventBus podemos achar detalhes do evento, como o token e o userId

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/grupo%20de%20logs%20cloudwatch.png)
<br><br>
![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/grupo%20de%20logs%20cloudwatch%202.png)


## New Order

Criaremos uma regra que espera pelo evento ```Validator.NewOrder``` e passa para o workflow alvo. Porém não a testaremos por agora

#### Criando a regra

1. Vá para o console EventBridge e selecione a opção de criar regra
2. Para o nome, ```NewOrder``` e para o barramento de eventos ```Serverlesspresso```
3. Para a fonte do evento, escolha Outros e cole o seguinte JSON personalizado:

```JSON
{
  "detail-type": ["Validator.NewOrder"],
  "source": ["awsserverlessda.serverlesspresso"]
}
``` 

4. Nos destinos, escolha o tipo Serviço da AWS, Máquina de estado do Step Functions e escolha a máquina de estado que criamos no WorkFlow

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/newOrder%201.png)

5. Pode ignorar as etiquetas
6. Tenha certeza que o barramento de eventos é o correto na revisão

## Workflow Started

Vamos criar e testar uma regra que recebe o evento ```WorkflowStarted``` contendo um token. Iremos rotear este evento para uma função Lambda que interage com uma tabela DynamoDB.

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/workflow%20started%200.png)

### Criando a regra

1. Vá até o console EventBridge e selecione criar regra
2. Para o nome, ```WorkflowStarted``` e para o barramento de eventos ```Serverlesspresso```
3. Para a fonte do evento escolha Outros e cole o seguinte JSON personalizado

```JSON
{
  "detail-type": ["OrderProcessor.WorkflowStarted"],
  "source": ["awsserverlessda.serverlesspresso"]
}
```

4. Para os destinos escolha serviço AWS, Função Lambda e procure pela função que tenha WorkFlowStarted no nome (o nome completo pode estar um pouco diferente dependendo de quando você estiver fazendo este workshop).

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/workflowstarted%201.png)

5. Ignore as etiquetas
6. Na revisão verifique o barramento de eventos.

### Testando

1. No EventBridge, procure pelo barramento de Eventos Serverlesspresso

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/newOrder%202.png)

2. Selecione Enviar eventos e o preencha da seguinte maneira:
   1. Para a fonte do evento: ```awsserverlessda.serverlesspresso```
   2. Para o tipo de detalhe: ```Validator.NewOrder```
   3. E o seguinte JSON:
```JSON
{"userId":"1","orderId":"1"}
``` 

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/newOrder%203.png)

3. Envie o evento
4. No console CloudWatch procure pelo grupo de logs serverlesspressoEventBus
5. O log mais recente deve se parecer com este

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/workflow%20started%202.png)

6. Vá até o console DynamoDB, procure pela tabela de nome serverlesspresso-order-table

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/workflow%20started%20dynamoDB.png)

7. Selecione o item mais recente, será o item criado neste teste

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/workflow%20started%203.png)


## Waiting Completion

Uma última regra antes de irmos para os testes finais.
Essa regra roteia o evento WaitingCompletion para uma função Lambda que atualiza a tabela ```serverlesspresso-order-table``` com o novo Token, número do pedido e estado.

### Criando a regra

1. Vá até o console EventBridge e selecione criar regra
2. Para o nome, ```WaitingCompletion``` e para o barramento de eventos ```Serverlesspresso```
3. Para a fonte do evento escolha Outros e cole o seguinte JSON personalizado

```JSON
{
  "detail-type": ["OrderProcessor.WaitingCompletion"],
  "source": ["awsserverlessda.serverlesspresso"]
}
```

4. Para os destinos escolha serviço AWS, Função Lambda e procure pela função que tenha WaitingCompletion no nome (o nome completo pode estar um pouco diferente dependendo de quando você estiver fazendo este workshop).

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/waiting%20completion.png)

5. Ignore as etiquetas
6. Na revisão verifique o barramento de eventos.
7. Vá até o EventBridge novamente, procure pelo barramento de eventos Serverlesspresso e certifique de que foram criadas essas 4 regras:

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/event%20routing/waiting%20completion%202.png)

## Teste final

O teste final será feito em outro arquivo markdown, devido ao tamanho desde arquivo. [Clique aqui](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/2%20-%20Roteamento%20de%20eventos%20com%20EventBridge/teste%20final/README.md)