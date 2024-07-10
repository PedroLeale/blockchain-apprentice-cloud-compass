# Configurando os frontends

Os frontends são aplicações Vue.js que foram implantadas com AWS Amplify.

## Sumário

- [Cloudshell](#cloudshell)
- [Aplicativo de Display](#aplicativo-de-display)
- [Aplicativo de Barista](#aplicativo-de-barista)
- [Aplicativo de Cliente](#aplicativo-de-cliente)
- [Teste](#teste)

## CloudShell

Usaremos o CloudShell para pegar algumas informações, anote-as

1. Abra o terminal e digite: ```aws cognito-identity list-identity-pools --max-results 10```

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/poolID.png)

2. Anote o valor __poolID__.
3. Agora digite: ```aws iot describe-endpoint --endpoint-type iot:Data-ATS```

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/host.png)

4. Anote o valor __endpointAddress__, este será seu Host.

## Aplicativo de Display

1. Vá até o console CloudFormation
2. Selecione a stack do workshop e procure nos outputs a URL do aplicativo de display

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/displayapp.png)

3. Clicando na URL, será redirecionado a uma página com formulário. A preencha com os valores que buscamos no CloudShell e clique em Save and Reload

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/displayapp2.png)

4. Agora crie uma conta, mas não faça login ainda.
5. Vá até o console Cognito na AWS. Procure por ServerlesspressoUserPool

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/displayapp3.png) 

6. Crie um grupo com o nome admin e adicione o seu usuário a ele
7. Agora faça login na sua conta

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/displayapp4.png) 

## Aplicativo de Barista

1. Na página do aplicativo de Display, selecione "Configure barista app"

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/baristaapp.png)

2. Mesmo procedimento de antes, clique em Save and Reload

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/baristaapp2.png)

3. Faça login na mesma conta do aplicativo de Display

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/customerapp.png)


## Aplicativo de Cliente

1. No aplicativo de display, selecione Configure order app

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/customerapp.png)

2. Irá abrir um popup contendo um QR code

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/customerapp2.png)

3. Ao escanear, será levado para uma página similar a dos outros aplicativos em que devemos clicar em Save and Reload

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/customerapp3.png)

## Teste

1. Com todos os aplicativos abertos, faça um pedido qualquer pelo aplicativo do cliente

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/test%20order%20a%20coffee.png)

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/test%20order%20in%20queue.png)

2. No aplicativo do barista, podemos aceitar ou recusar o pedido, além de controlar o status dele

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/test%20barista%20app%201.png)
![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/test%20barista%20app%202.png)
![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/test%20barista%20app%203.png)

3. Além disso, o aplicativo de display mostra os pedidos com o número de acordo com a ordem

![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/test%20display%20app%201.png)
![](/progress%20report%203.2/tarefa%202:%20Serverlesspresso/images/frontends/test%20display%20app%202.png)

