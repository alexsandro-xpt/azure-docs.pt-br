---
title: Configurar notificações e modelos de email no Gerenciamento de API do Azure | Microsoft Docs
description: Saiba como configurar notificações e modelos de email no Gerenciamento de API do Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/02/2018
ms.author: apimpm
ms.openlocfilehash: 60788f76dac58ead10e43e892d587a86bdd3fcad
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/10/2018
ms.locfileid: "33934277"
---
# <a name="how-to-configure-notifications-and-email-templates-in-azure-api-management"></a>Como configurar notificações e modelos de email no Gerenciamento de API do Azure
O Gerenciamento de API possibilita configurar notificações de eventos específicos e modelos dos emails que são usados para se comunicar com os administradores e desenvolvedores de uma instância do Gerenciamento de API. Este artigo mostra como configurar notificações de eventos disponíveis e fornece uma visão geral da configuração dos modelos dos emails usados desses eventos.

## <a name="prerequisites"></a>pré-requisitos

Se você não tiver uma instância do serviço de Gerenciamento de API, conclua o seguinte guia de início rápido: [Criar uma instância de Gerenciamento de API do Azure](get-started-create-service-instance.md).

## <a name="publisher-notifications"> </a>Configurar notificações

1. Selecione a instância **GERENCIAMENTO DE API**.
2. Clique em **Notificações** para exibir as notificações disponíveis.

    ![Notificações do editor][api-management-publisher-notifications]

    Os eventos na lista a seguir podem ser configurados para enviar notificações.

    * **Solicitações de assinatura (que precisam de aprovação)** - Os usuários e destinatários de email especificados receberão notificações por email sobre solicitações de assinaturas de produtos de API que precisarem de aprovação.
    * **Novas assinaturas** - Os usuários e destinatários de email especificados receberão notificações por email sobre novas assinaturas de produtos de API.
    * **Solicitações da galeria de aplicativos** - Os usuários e destinatários de email especificados receberão notificações por email quando novos aplicativos forem enviados à galeria de aplicativos.
    * **CCO** - Os usuários e destinatários de email especificados receberão cópias ocultas de todos os emails enviados a desenvolvedores.
    * **Novo problema ou comentário** - Os usuários e destinatários de email especificados receberão notificações por email quando um novo problema ou comentário for enviado no portal do desenvolvedor.
    * **Mensagem de conta encerrada** - Os usuários e destinatários de email especificados receberão notificações por email quando uma conta for encerrada.
    * **Limite de cota de assinatura se aproximando** - Os usuários e destinatários de email a seguir receberão notificações por email quando o uso de assinaturas estiver se aproximando da cota.

    Para cada evento, você pode especificar destinatários de email usando a caixa de texto de endereço de email ou selecionando os usuários em uma lista.

3. Para especificar os endereços de email a serem notificados, insira-os na caixa de texto de endereço de email. Se tiver diversos endereços de email, separe-os por vírgula.

    ![Destinatários da notificação][api-management-email-addresses]
4. Pressione **Adicionar**.

## <a name="email-templates"> </a>Configurar modelos de notificação
O Gerenciamento de API fornece modelos de notificação para mensagens de email que são enviadas no decorrer da administração e da utilização do serviço. Os seguintes modelos de email são fornecidos.

* Envio à galeria de aplicativos aprovado
* Carta de despedida do desenvolvedor
* Notificação de limite de cota se aproximando
* Convidar usuário
* Novo comentário adicionado a um problema
* Novo problema recebido
* Nova assinatura ativada
* Confirmação de renovação de assinatura
* Solicitação de assinatura negada
* Solicitação de assinatura recebida

Esses modelos podem ser modificados da forma desejada.

Para exibir e configurar os modelos de email para sua instância de Gerenciamento de API, clique em **Modelos de notificações**.

![Modelos de email][api-management-email-templates]

Cada modelo de email tem um assunto em texto sem formatação e uma definição de corpo em formato HTML. Cada item pode ser personalizado da forma desejada.

![Editor de modelos de email][api-management-email-template]

A lista **Parâmetros** contém parâmetros que, quando inseridos no assunto ou corpo, serão substituídos pelo valor designado quando o email for enviado. Para inserir um parâmetro, posicione o cursor onde quiser que ele seja colocado e clique na seta à esquerda do nome do parâmetro.

> [!NOTE] 
> Os parâmetros não são substituídos por valores reais ao visualizar ou enviar um teste.

Para salvar as alterações feitas no modelo de email, clique em **Salvar** ou, para cancelar as alterações, clique em **Descartar**.
 

[api-management-management-console]: ./media/api-management-howto-configure-notifications/api-management-management-console.png
[api-management-publisher-notifications]: ./media/api-management-howto-configure-notifications/api-management-publisher-notifications.png
[api-management-email-addresses]: ./media/api-management-howto-configure-notifications/api-management-email-addresses.png


[api-management-email-templates]: ./media/api-management-howto-configure-notifications/api-management-email-templates.png
[api-management-email-templates-list]: ./media/api-management-howto-configure-notifications/api-management-email-templates-list.png
[api-management-email-template]: ./media/api-management-howto-configure-notifications/api-management-email-template.png


[Configure publisher notifications]: #publisher-notifications
[Configure email templates]: #email-templates

[How to create and use groups]: api-management-howto-create-groups.md
[How to associate groups with developers]: api-management-howto-create-groups.md#associate-group-developer

[Get started with Azure API Management]: get-started-create-service-instance.md
[Create an API Management service instance]: get-started-create-service-instance.md
