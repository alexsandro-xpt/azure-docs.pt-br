---
title: Conectar-se ao Trello por meio do Aplicativo Lógico do Azure | Microsoft Docs
description: Automatizar tarefas e fluxos de trabalho que monitoram e gerenciam listas, quadros e cartões em seus projetos do Trello usando Aplicativo Lógico do Azure
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: fe7a4377-5c24-4f72-ab1a-6d9d23e8d895
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: 4ae8d3dff108f14844c31d7b9d0b0871326832a3
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046142"
---
# <a name="monitor-and-manage-trello-with-azure-logic-apps"></a>Monitorar e gerenciar o Trello com Aplicativo Lógico do Azure

Com o Aplicativo Lógico do Azure e o conector do Trello, você pode criar tarefas automatizadas e fluxos de trabalho que monitoram e gerenciam listas, cartões, quadros e membros da equipe do Trello e assim por diante, por exemplo:

* Monitore quando novos cartões são adicionados a listas e quadros. 
* Crie, obtenha e gerencie listas, cartões e quadros.
* Adicione comentários e membros aos cartões.
* Liste quadros, rótulos de quadro, cartões em quadros, comentários de cartão, membros de cartão, membros da equipe e equipes em que você é um membro. 
* Obtenha as equipes.

Você pode usar gatilhos para obter respostas da sua conta do Trello e disponibilizar a saída para outras ações. Você pode usar ações que executam tarefas com sua conta do Trello. Você também pode fazer com que outras ações usem a saída das ações do Trello. Por exemplo, quando um novo cartão é adicionado ao painel ou à lista, você pode enviar mensagens com o conector do Slack. Se ainda não estiver familiarizado com os aplicativos lógicos, veja [O que é o Aplicativo Lógico do Azure?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Pré-requisitos

* Uma assinatura do Azure. Se você não tiver uma assinatura do Azure, <a href="https://azure.microsoft.com/free/" target="_blank">inscreva-se em uma conta gratuita do Azure</a>. 

* Suas credenciais de usuário e conta do Trello

  Suas credenciais autorizam o aplicativo lógico a criar uma conexão e acessar sua conta do Trello.

* Conhecimento básico sobre [como criar aplicativos lógicos](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* O aplicativo lógico no qual você deseja acessar a conta do Trello. Para iniciar com um gatilho do Trello, [crie um aplicativo lógico em branco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar uma ação do Trello, inicie seu aplicativo lógico com um gatilho, por exemplo, o gatilho **Recorrência**.

## <a name="connect-to-trello"></a>Conectar-se ao Trello

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Entre no [portal do Azure](https://portal.azure.com) e abra seu aplicativo lógico no Designer de Aplicativo Lógico, se ele ainda não estiver aberto.

1. Para aplicativos lógicos em branco, na caixa de pesquisa, insira "trello" como o filtro. Na lista de gatilhos, selecione o gatilho desejado. 

   -ou-

   Para aplicativos lógicos existentes, na última etapa em que deseja adicionar uma ação, escolha **Nova etapa**. 
   Na caixa de pesquisa, insira "trello" como o filtro. 
   Na lista de ações, selecione a ação desejada.

   Para adicionar uma ação entre as etapas, mova o ponteiro sobre a seta entre as etapas. 
   Escolha o sinal de adição (**+**) que aparece e, em seguida, selecione **Adicionar uma ação**.

1. Se você for solicitado a entrar no Trello, autorize o acesso ao aplicativo lógico e entre.

1. Forneça os detalhes necessários para o gatilho ou a ação selecionada e continue criando o fluxo de trabalho do aplicativo lógico.

## <a name="connector-reference"></a>Referência de conector

Para obter detalhes técnicos sobre gatilhos, ações e limites, que são explicados na descrição da OpenAPI do conector (anteriormente conhecido como Swagger), veja a [página de referência](/connectors/trello/) do conector.

## <a name="get-support"></a>Obtenha suporte

* Em caso de dúvidas, visite o [Fórum dos Aplicativos Lógicos do Azure](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps).
* Para enviar ou votar em ideias de recurso, visite o [site de comentários do usuário de Aplicativos Lógicos](http://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Próximas etapas

* Saiba mais sobre outros [conectores de Aplicativos Lógicos](../connectors/apis-list.md)