---
title: Chamar ponto de extremidade usando Node.js - Pesquisa Personalizada do Bing - Serviços Cognitivos da Microsoft
description: Este início rápido mostra como solicitar resultados de pesquisa da instância de pesquisa personalizada usando Node.js para chamar o ponto de extremidade da Pesquisa Personalizada do Bing.
services: cognitive-services
author: brapel
manager: ehansen
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: article
ms.date: 05/07/2018
ms.author: v-brapel
ms.openlocfilehash: 73c31c7175bd4dfcb182fb76784937c176ac7702
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977866"
---
# <a name="call-bing-custom-search-endpoint-nodejs"></a>Chamar ponto de extremidade da Pesquisa Personalizada do Bing (Node.js)

Este início rápido mostra como solicitar resultados de pesquisa de sua instância de pesquisa personalizada usando o Node.js para chamar o ponto de extremidade da Pesquisa Personalizada do Bing. 

## <a name="prerequisites"></a>Pré-requisitos

Para concluir este início rápido, você precisa de:

- Uma instância de pesquisa personalizada pronta para uso. Consulte [Criar a primeira instância da Pesquisa Personalizada do Bing](quick-start.md).
- [Node. js](https://www.nodejs.org/) instalado.
- Uma chave de assinatura. Você pode obter uma chave de assinatura quando ativar sua [avaliação gratuita](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search) ou pode usar uma chave de assinatura paga no painel do Azure (consulte [Conta da API de Serviços Cognitivos](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)).    

## <a name="run-the-code"></a>Executar o código

Para executar esse exemplo, siga estas etapas:

1. Crie uma pasta para o código.  
  
2. Em um prompt de comando ou terminal, navegue até a pasta que você acabou de criar.  
  
3. Instale o módulo do nó de **solicitação**:
    <pre>
    npm install request
    </pre>  
    
4. Crie um arquivo chamado BingCustomSearch.js na pasta que você criou e copie o seguinte código nele. Substitua **YOUR-SUBSCRIPTION-KEY** e **YOUR-CUSTOM-CONFIG-ID** pela sua chave de assinatura e ID de configuração.  
  
    ``` javascript
    var request = require("request");
    
    var subscriptionKey = 'YOUR-SUBSCRIPTION-KEY';
    var customConfigId = 'YOUR-CUSTOM-CONFIG-ID';
    var searchTerm = 'microsoft';
    
    var options = {
        url: 'https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/search?' + 
          'q=' + searchTerm + 
          '&customconfig=' + customConfigId,
        headers: {
            'Ocp-Apim-Subscription-Key' : subscriptionKey
        }
    }
    
    request(options, function(error, response, body){
        var searchResponse = JSON.parse(body);
        for(var i = 0; i < searchResponse.webPages.value.length; ++i){
            var webPage = searchResponse.webPages.value[i];
            console.log('name: ' + webPage.name);
            console.log('url: ' + webPage.url);
            console.log('displayUrl: ' + webPage.displayUrl);
            console.log('snippet: ' + webPage.snippet);
            console.log('dateLastCrawled: ' + webPage.dateLastCrawled);
            console.log();
        }
    })
    ```  
  
6. Execute o código usando o seguinte comando:  
  
    ```    
    node BingCustomSearch.js
    ``` 

## <a name="next-steps"></a>Próximas etapas
- [Configurar a experiência de interface do usuário hospedada](./hosted-ui.md)
- [Usar marcadores de decoração para realçar texto](./hit-highlighting.md)
- [Paginar páginas da Web](./page-webpages.md)
