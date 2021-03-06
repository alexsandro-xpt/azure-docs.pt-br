---
title: Referência do desenvolvedor de JavaScript do Azure Functions | Microsoft Docs
description: Entenda como desenvolver funções usando JavaScript.
services: functions
documentationcenter: na
author: ggailey777
manager: cfowler
editor: ''
tags: ''
keywords: azure functions, functions, processamento de eventos, webhooks, computação dinâmica, arquitetura sem servidor
ms.assetid: 45dedd78-3ff9-411f-bb4b-16d29a11384c
ms.service: functions
ms.devlang: nodejs
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 03/04/2018
ms.author: glenga
ms.openlocfilehash: 6099a818651cf75a75159f43748720b3eb01e4de
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287814"
---
# <a name="azure-functions-javascript-developer-guide"></a>Guia do desenvolvedor de JavaScript do Azure Functions

A experiência de JavaScript para o Azure Functions torna mais fácil exportar uma função que é passada como um objeto `context` para se comunicar com o tempo de execução e para receber e enviar dados por meio de associações.

Este artigo pressupõe que você já tenha lido a [referência do desenvolvedor do Azure Functions](functions-reference.md).

## <a name="exporting-a-function"></a>Exportando uma função
Cada função JavaScript deve exportar um único `function` via `module.exports` para o tempo de execução para localizar a função e executá-la. Esta função deve sempre ter um objeto `context` como o primeiro parâmetro.

```javascript
// You must include a context, other arguments are optional
module.exports = function(context, myTrigger, myInput, myOtherInput) {
    // function logic goes here :)
    context.done();
};
// You can also use 'arguments' to dynamically handle inputs
module.exports = function(context) {
    context.log('Number of inputs: ' + arguments.length);
    // Iterates through trigger and input binding data
    for (i = 1; i < arguments.length; i++){
        context.log(arguments[i]);
    }
    context.done();
};
```

Associações de gatilho e entrada (associações de `direction === "in"`) podem ser passadas para a função como parâmetros. Eles são passados para a função na mesma ordem em que são definidos *function.json*. Você pode manipular dinamicamente entradas usando o objeto JavaScript [`arguments`](https://msdn.microsoft.com/library/87dw3w1k.aspx). Por exemplo, se tiver `function(context, a, b)` e alterá-lo para `function(context, a)`, você ainda poderá obter o valor de `b` no código de função, fazendo referência ao `arguments[2]`.

Todas as associações, independentemente da direção, também são passadas no objeto `context` usando a propriedade `context.bindings`.

## <a name="context-object"></a>objeto de contexto
O tempo de execução usa um objeto `context` para passar dados de/para sua função e permitir que você se comunique com o tempo de execução.

O objeto `context` sempre é o primeiro parâmetro de uma função e deve ser incluído porque tem métodos como `context.done` e `context.log`, que são necessários para usar o tempo de execução corretamente. Você pode nomear o objeto de acordo com a sua preferência (por exemplo, `ctx` ou `c`).

```javascript
// You must include a context, but other arguments are optional
module.exports = function(context) {
    // function logic goes here :)
    context.done();
};
```

### <a name="contextbindings-property"></a>Propriedade context.bindings

```
context.bindings
```
Retorna um objeto nomeado que contém todos os dados de entrada e saída. Por exemplo, a seguinte definição de associação em seu *function.json*, permite que você acesse o conteúdo da fila por meio do objeto `context.bindings.myInput`. 

```json
{
    "type":"queue",
    "direction":"in",
    "name":"myInput"
    ...
}
```

```javascript
// myInput contains the input data, which may have properties such as "name"
var author = context.bindings.myInput.name;
// Similarly, you can set your output data
context.bindings.myOutput = { 
        some_text: 'hello world', 
        a_number: 1 };
```

### <a name="contextdone-method"></a>Método context.done
```
context.done([err],[propertyBag])
```

Informa ao tempo de execução que seu código terminou. Se sua função usa a `async function` declaração (disponível usando o Nó 8 + no Functions versão 2.x), você não precisará usar `context.done()`. O `context.done` retorno de chamada é chamado implicitamente.

Se sua função não é uma função assíncrona, **você deve chamar `context.done`** para informar o tempo de execução que sua função é concluída. A execução atingirá o tempo limite se ele estiver ausente.

O método `context.done` permite que você passe um erro definido pelo usuário de volta ao tempo de execução, e um recipiente de propriedades que substitui as propriedades no objeto `context.bindings`.

```javascript
// Even though we set myOutput to have:
//  -> text: hello world, number: 123
context.bindings.myOutput = { text: 'hello world', number: 123 };
// If we pass an object to the done function...
context.done(null, { myOutput: { text: 'hello there, world', noNumber: true }});
// the done method will overwrite the myOutput binding to be: 
//  -> text: hello there, world, noNumber: true
```

### <a name="contextlog-method"></a>Método context.log  

```
context.log(message)
```
Permite que você grave nos logs do console de streaming no nível de rastreamento padrão. Há métodos adicionais de registro em log disponíveis no `context.log` que permitem a gravação no log de console em outros níveis de rastreamento:


| Método                 | DESCRIÇÃO                                |
| ---------------------- | ------------------------------------------ |
| **error(_message_)**   | Grava no registro em log no nível do erro, ou em um nível inferior.   |
| **warn(_message_)**    | Grava no registro em log no nível do aviso, ou em um nível inferior. |
| **info(_message_)**    | Grava no registro em log no nível da informação, ou em um nível inferior.    |
| **verbose(_message_)** | Grava no registro em log no nível detalhado.           |

O exemplo a seguir grava no console no nível de rastreamento de aviso:

```javascript
context.log.warn("Something has happened."); 
```
Você pode definir o limite do nível de rastreamento para registrar em log no arquivo host.json, ou desativá-lo.  Para saber mais sobre como gravar nos logs, veja a próxima seção.

## <a name="binding-data-type"></a>Tipo de dados de associação

Para definir o tipo de dados para uma associação de entrada, use a propriedade `dataType` na definição de associação. Por exemplo, para ler o conteúdo de uma solicitação HTTP em formato binário, use o tipo `binary`:

```json
{
    "type": "httpTrigger",
    "name": "req",
    "direction": "in",
    "dataType": "binary"
}
```

Outras opções para `dataType` são `stream` e `string`.

## <a name="writing-trace-output-to-the-console"></a>Gravar a saída de rastreamento no console 

No Functions, use os métodos `context.log` para gravar a saída de rastreamento no console. Neste ponto, não é possível usar `console.log` para gravar no console.

Quando você chama `context.log()`, sua mensagem é gravada no console no nível de rastreamento padrão, que é o nível de rastreamento de _informações_. O código a seguir grava no console no nível de rastreamento de informações:

```javascript
context.log({hello: 'world'});  
```

O código anterior é o equivalente ao código a seguir:

```javascript
context.log.info({hello: 'world'});  
```

O código a seguir grava no console no nível do erro:

```javascript
context.log.error("An error has occurred.");  
```

Como _erro_ é o nível de rastreamento mais alto, esse rastreamento é gravado na saída em todos os níveis de rastreamento enquanto o registro em log estiver habilitado.  


Todos os métodos `context.log` dão suporte ao mesmo formato de parâmetro que o [método util.format](https://nodejs.org/api/util.html#util_util_format_format) de Node.js. Considere o código a seguir, que grava no console usando o nível de rastreamento padrão:

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=' + req.originalUrl);
context.log('Request Headers = ' + JSON.stringify(req.headers));
```

Você também pode escrever o mesmo código no formato a seguir:

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);
context.log('Request Headers = ', JSON.stringify(req.headers));
```

### <a name="configure-the-trace-level-for-console-logging"></a>Configurar o nível de rastreamento para o registro em log no console

O Functions permite a definição do nível de rastreamento de limite para gravar no console, o que facilita o controle do modo de gravação dos rastreamentos no console a partir de suas funções. Para definir o limite para todos os rastreamentos gravados no console, use a propriedade `tracing.consoleLevel` no arquivo host.json. Essa configuração se aplica a todas as funções em seu aplicativo de função. O exemplo a seguir define o limite de rastreamento para habilitar o registro em log detalhado:

```json
{ 
    "tracing": {      
        "consoleLevel": "verbose"     
    }
}  
```

Os valores de **consoleLevel** correspondem aos nomes dos métodos `context.log`. Para desabilitar todo o registro em log do rastreamento no console, defina **consoleLevel** como _off_. Para obter mais informações, consulte a [referência para host.json](functions-host-json.md).

## <a name="http-triggers-and-bindings"></a>Gatilhos e associações HTTP

HTTP e gatilhos de webhook e associações de saída HTTP usam objetos de solicitação e resposta para representar as mensagens HTTP.  

### <a name="request-object"></a>Objeto da solicitação

O objeto `request` tem as seguintes propriedades:

| Propriedade      | DESCRIÇÃO                                                    |
| ------------- | -------------------------------------------------------------- |
| _body_        | Um objeto que contém o corpo da solicitação.               |
| _headers_     | Um objeto que contém os cabeçalhos da solicitação.                   |
| _method_      | O método HTTP da solicitação.                                |
| _originalUrl_ | A URL da solicitação.                                        |
| _params_      | Um objeto que contém os parâmetros de roteamento da solicitação. |
| _query_       | Um objeto que contém os parâmetros da consulta.                  |
| _rawBody_     | O corpo da mensagem como uma cadeia de caracteres.                           |


### <a name="response-object"></a>Objeto de resposta

O objeto `response` tem as seguintes propriedades:

| Propriedade  | DESCRIÇÃO                                               |
| --------- | --------------------------------------------------------- |
| _body_    | Um objeto que contém o corpo da resposta.         |
| _headers_ | Um objeto que contém os cabeçalhos da resposta.             |
| _isRaw_   | Indica que a formatação foi ignorada para a resposta.    |
| _status_  | O código de status HTTP da resposta.                     |

### <a name="accessing-the-request-and-response"></a>Acessar a solicitação e a resposta 

Ao trabalhar com gatilhos HTTP, há três maneiras de acessar os objetos de solicitação e resposta HTTP:

+ A partir das associações de entrada e saída nomeadas. Dessa forma, o gatilho e as associações de HTTP funcionam da mesma forma que qualquer outra associação. O exemplo a seguir define o objeto de resposta usando uma associação chamada `response`: 

    ```javascript
    context.bindings.response = { status: 201, body: "Insert succeeded." };
    ```

+ Das propriedades `req` e `res` no objeto `context`. Dessa forma, você pode usar o padrão convencional para acessar os dados HTTP a partir do objeto de contexto, em vez de usar o padrão `context.bindings.name` completo. O exemplo a seguir mostra como acessar os objetos `req` e `res` no `context`:

    ```javascript
    // You can access your http request off the context ...
    if(context.req.body.emoji === ':pizza:') context.log('Yay!');
    // and also set your http response
    context.res = { status: 202, body: 'You successfully ordered more coffee!' }; 
    ```

+ Chamando `context.done()`. Um tipo especial de associação HTTP que retorna a resposta passada ao método `context.done()`. A seguinte associação de saída HTTP define um parâmetro de saída `$return`:

    ```json
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
    ``` 
    Essa associação de saída espera que você forneça a resposta ao chamar `done()` da seguinte maneira:

    ```javascript
     // Define a valid response object.
    res = { status: 201, body: "Insert succeeded." };
    context.done(null, res);   
    ```  

## <a name="node-version-and-package-management"></a>Versão do Node e gerenciamento de pacote

A tabela a seguir mostra a versão do Node.js usada por cada versão principal do tempo de execução do Functions:

| Versão do Functions | Versão do Node.js | 
|---|---|
| 1.x | 6.11.2 (bloqueada pelo tempo de execução) |
| 2. x  | _Active Directory LTS_ e _atual_ versões de Node. js (8.11.1 e 10.6.0 recomendado). Defina a versão usando a [configuração do aplicativo](functions-how-to-use-azure-function-app-settings.md#settings) WEBSITE_DEFAULT_NODE_VERSION.|

Você pode ver a versão atual usada pelo tempo de execução imprimindo `process.version` de qualquer função.

As etapas a seguir permitem que você inclua pacotes em seu aplicativo de função: 

1. Vá para `https://<function_app_name>.scm.azurewebsites.net`.

2. Clique em **Console de Depuração** > **CMD**.

3. Acesse `D:\home\site\wwwroot`e arraste o arquivo package.json para a pasta **wwwroot** na metade superior da página.  
    Também há outras maneiras de carregar arquivos em seu aplicativo de função. Para saber mais, confira [Como atualizar os arquivos do aplicativo de função](functions-reference.md#fileupdate). 

4. Depois que o arquivo package.json é carregado, execute o comando `npm install` no **console de execução remota do Kudu**.  
    Essa ação baixa os pacotes indicados no arquivo package.json e reinicia o aplicativo de função.

Após a instalação dos pacotes necessários, você os importa para sua função chamando `require('packagename')`, como no exemplo a seguir:

```javascript
// Import the underscore.js library
var _ = require('underscore');
var version = process.version; // version === 'v6.5.0'

module.exports = function(context) {
    // Using our imported underscore.js library
    var matched_names = _
        .where(context.bindings.myInput.names, {first: 'Carla'});
```

Você deve definir um arquivo `package.json` na raiz de seu aplicativo de função. A definição de arquivo permite que todas as funções no aplicativo compartilhem os mesmos pacotes armazenados em cache, o que oferece o melhor desempenho. Se houver conflitos de versão, você poderá resolver o conflito adicionando um arquivo `package.json` na pasta de uma função específica.  

## <a name="environment-variables"></a>Variáveis de ambiente
Para obter uma variável de ambiente ou um valor de configuração do aplicativo, use `process.env`, conforme mostrado na função `GetEnvironmentVariable`:

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    context.log('Node.js timer trigger function ran!', timeStamp);   
    context.log(GetEnvironmentVariable("AzureWebJobsStorage"));
    context.log(GetEnvironmentVariable("WEBSITE_SITE_NAME"));

    context.done();
};

function GetEnvironmentVariable(name)
{
    return name + ": " + process.env[name];
}
```
## <a name="considerations-for-javascript-functions"></a>Considerações para funções em JavaScript

Ao trabalhar com funções em JavaScript, lembre-se das considerações nas duas seções a seguir.

### <a name="choose-single-vcpu-app-service-plans"></a>Escolher Planos do Serviço de Aplicativo de vCPU único

Ao criar um aplicativo de funções que usa o Plano do Serviço de Aplicativo, recomendamos que você selecione um plano de vCPU único em vez de um plano com vários vCPUs. Atualmente, o Functions executa funções em JavaScript com mais eficiência em VMs de vCPU único, e o uso de VMs maiores não produz os aprimoramentos de desempenho esperados. Quando for necessário, você poderá escalar horizontalmente manualmente adicionando mais instâncias de VM de vCPU único ou poderá habilitar o dimensionamento automático. Para saber mais, confira [Dimensionar a contagem de instâncias manual ou automaticamente](../monitoring-and-diagnostics/insights-how-to-scale.md?toc=%2fazure%2fapp-service-web%2ftoc.json).    

### <a name="typescript-and-coffeescript-support"></a>Suporte a TypeScript e CoffeeScript
Como ainda não há suporte direto para compilação automática de TypeScript ou CoffeeScript por meio do tempo de execução, esse suporte precisa ser manipulado fora do tempo de execução, no tempo de implantação. 

## <a name="next-steps"></a>Próximas etapas
Para saber mais, consulte os recursos a seguir:

* [Práticas recomendadas para o Azure Functions](functions-best-practices.md)
* [Referência do desenvolvedor do Azure Functions](functions-reference.md)
* [Gatilhos e associações de Azure Functions](functions-triggers-bindings.md)

