---
title: Conectar-se à conta do SharePoint do Aplicativo Lógico do Azure | Microsoft Docs
description: Automatize tarefas e fluxos de trabalho que monitoram e gerenciam recursos no SharePoint Online ou SharePoint Server local usando o Aplicativo Lógico do Azure
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: e0ec3149-507a-409d-8e7b-d5fbded006ce
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: 5a3968e1aec98092767712220d8aaf676aba89cd
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/27/2018
ms.locfileid: "43053313"
---
# <a name="monitor-and-manage-sharepoint-resources-with-azure-logic-apps"></a>Monitorar e gerenciar recursos do SharePoint com o Aplicativo Lógico do Azure

Com o Aplicativo Lógico do Azure e o conector do SharePoint, você pode criar tarefas e fluxos de trabalho automatizados que monitoram e gerenciam recursos, como arquivos, pastas, listas, itens, pessoas e assim por diante, no SharePoint Online ou no SharePoint Server local, por exemplo:

* Monitorar quando arquivos ou itens são criados, alterados ou excluídos.
* Criar, obter, atualizar ou excluir itens.
* Adicionar, obter ou excluir anexos. Obter o conteúdo de anexos.
* Criar, copiar, atualizar ou excluir arquivos. 
* Atualizar propriedades do arquivo. Obter o conteúdo, metadados ou propriedades de um arquivo.
* Listar ou extrair pastas.
* Obter listas ou exibições de lista.
* Definir o status de aprovação de conteúdo.
* Resolver pessoas.
* Enviar solicitações HTTP para o SharePoint.
* Obter valores de entidade.

Você pode usar gatilhos que obtêm respostas do SharePoint e disponibilizar a saída para outras ações. Você pode usar ações nos aplicativos lógicos para executar tarefas no SharePoint. Você também pode fazer com que outras ações usem a saída das ações do SharePoint. Por exemplo, se você buscar regularmente arquivos do SharePoint, poderá enviar mensagens para a sua equipe usando o conector do Slack.
Se ainda não estiver familiarizado com os aplicativos lógicos, veja [O que é o Aplicativo Lógico do Azure?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Pré-requisitos

* Uma assinatura do Azure. Se você não tiver uma assinatura do Azure, <a href="https://azure.microsoft.com/free/" target="_blank">inscreva-se em uma conta gratuita do Azure</a>. 

* Suas credenciais de usuário e endereço de site do SharePoint

  Suas credenciais autorizam o aplicativo lógico a criar uma conexão e acessar sua conta do SharePoint. 

* Antes de poder conectar aplicativos lógicos a sistemas locais, como o SharePoint Server, você precisará [instalar e configurar o gateway de dados local](../logic-apps/logic-apps-gateway-install.md). Dessa forma, você pode especificar para usar sua instalação de gateway quando criar a conexão do SharePoint Server com seu aplicativo lógico.

* Conhecimento básico sobre [como criar aplicativos lógicos](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* O aplicativo lógico no qual você deseja acessar a conta do SharePoint. Para começar com um gatilho do SharePoint, [crie um aplicativo lógico em branco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar uma ação do SharePoint, inicie o aplicativo lógico com um gatilho, como um gatilho do Salesforce se você tiver uma conta do Salesforce.

  Por exemplo, você pode iniciar o aplicativo lógico com o gatilho **Quando um registro é criado** do Salesforce. 
  Esse gatilho é acionado sempre que um novo registro, como um lead, for criado no Salesforce. 
  Então, você pode seguir este gatilho com a ação **Criar arquivo** do SharePoint. Dessa forma, quando o novo registro é criado, seu aplicativo lógico cria um arquivo no SharePoint com informações sobre o novo registro.

## <a name="connect-to-sharepoint"></a>Conectar-se ao SharePoint

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Entre no [portal do Azure](https://portal.azure.com) e abra seu aplicativo lógico no Designer de Aplicativo Lógico, se ele ainda não estiver aberto.

1. Para aplicativos lógicos em branco, na caixa de pesquisa, insira "sharepoint" como o filtro. Na lista de gatilhos, selecione o gatilho desejado. 

   -ou-

   Para aplicativos lógicos existentes, na última etapa em que deseja adicionar uma ação do SharePoint, escolha **Nova etapa**. 
   Na caixa de pesquisa, insira “sharepoint” como filtro. 
   Na lista de ações, selecione a ação desejada.

   Para adicionar uma ação entre as etapas, mova o ponteiro sobre a seta entre as etapas. 
   Escolha o sinal de adição (**+**) que aparece e, em seguida, selecione **Adicionar uma ação**.

1. Quando for solicitado que você entre, forneça as informações de conexão necessárias. Se você estiver usando o SharePoint Server, verifique se selecionou **Conectar via gateway de dados local**. Quando terminar, escolha **Criar**.

1. Forneça os detalhes necessários para o gatilho ou a ação selecionada e continue criando o fluxo de trabalho do aplicativo lógico.

## <a name="connector-reference"></a>Referência de conector

Para obter detalhes técnicos sobre gatilhos, ações e limites, que são explicados na descrição da OpenAPI do conector (anteriormente conhecido como Swagger), veja a [página de referência](/connectors/sharepoint/) do conector.

## <a name="get-support"></a>Obtenha suporte

* Em caso de dúvidas, visite o [Fórum dos Aplicativos Lógicos do Azure](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps).
* Para enviar ou votar em ideias de recurso, visite o [site de comentários do usuário de Aplicativos Lógicos](http://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Próximas etapas

* Saiba mais sobre outros [conectores de Aplicativos Lógicos](../connectors/apis-list.md)
