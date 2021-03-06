---
title: Singletons para Funções Duráveis – Azure
description: Como usar singletons para a extensão de Funções Duráveis do Azure Functions.
services: functions
author: cgillum
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/29/2017
ms.author: azfuncdf
ms.openlocfilehash: 8177fb6e8dea83ab2b8b12183cdcca6e43f92ade
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/07/2018
ms.locfileid: "44095089"
---
# <a name="singleton-orchestrators-in-durable-functions-azure-functions"></a>Orquestradores singleton em Funções Duráveis (Azure Functions)

Para trabalhos em segundo plano ou orquestrações com estilo de ator, normalmente você precisa garantir que apenas uma instância de um determinado orquestrador seja executada por vez. Isso pode ser feito nas [Funções Duráveis](durable-functions-overview.md) atribuindo uma ID de instância específica a um orquestrador ao criá-lo.

## <a name="singleton-example"></a>Exemplo de singleton

O exemplo em C# a seguir mostra uma função de gatilho HTTP que cria uma orquestração singleton de trabalho em segundo plano. O código garante que apenas uma instância exista para uma ID de instância especificada.

```cs
[FunctionName("HttpStartSingle")]
public static async Task<HttpResponseMessage> RunSingle(
    [HttpTrigger(AuthorizationLevel.Function, methods: "post", Route = "orchestrators/{functionName}/{instanceId}")] HttpRequestMessage req,
    [OrchestrationClient] DurableOrchestrationClient starter,
    string functionName,
    string instanceId,
    TraceWriter log)
{
    // Check if an instance with the specified ID already exists.
    var existingInstance = await starter.GetStatusAsync(instanceId);
    if (existingInstance == null)
    {
        // An instance with the specified ID doesn't exist, create one.
        dynamic eventData = await req.Content.ReadAsAsync<object>();
        await starter.StartNewAsync(functionName, instanceId, eventData);
        log.Info($"Started orchestration with ID = '{instanceId}'.");
        return starter.CreateCheckStatusResponse(req, instanceId);
    }
    else
    {
        // An instance with the specified ID exists, don't create one.
        return req.CreateErrorResponse(
            HttpStatusCode.Conflict,
            $"An instance with ID '{instanceId}' already exists.");
    }
}
```

Por padrão, IDs de instância são GUIDs gerados aleatoriamente. Mas, nesse caso, a ID de instância é passada nos dados da rota da URL. O código chama [GetStatusAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_GetStatusAsync_) para verificar se uma instância com a ID especificada já está em execução. Se não estiver, uma instância será criada com essa ID.

> [!NOTE]
> Há uma condição de corrida potencial neste exemplo. Se duas instâncias de **HttpStartSingle** executarem simultaneamente, o resultado poderia ser duas diferentes criadas instâncias de singleton, que substituem um o outro. Dependendo dos seus requisitos, isso pode ter efeitos colaterais indesejáveis. Por esse motivo, é importante garantir que nenhum duas solicitações podem executar essa função de gatilho simultaneamente.

Os detalhes da implementação da função de orquestrador, na verdade, não importam. Poderia ser uma função de orquestrador regular que é iniciada e concluída ou poderia ser uma que é executada eternamente (ou seja, um [Orquestração Eterna](durable-functions-eternal-orchestrations.md)). O aspecto importante é que há somente uma instância em execução por vez.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Saiba como chamar suborquestrações](durable-functions-sub-orchestrations.md)
