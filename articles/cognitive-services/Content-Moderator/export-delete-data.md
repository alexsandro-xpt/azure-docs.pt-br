---
title: Exportar ou excluir seus dados no Content Moderator – Serviços Cognitivos do Azure | Microsoft Docs
description: Saiba como exportar ou excluir seus dados no Content Moderator.
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: article
ms.date: 05/25/2018
ms.author: v-jaswel
ms.openlocfilehash: fb496837560fe28f1a2e53d8c4ca67e23ada8f64
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576792"
---
# <a name="export-or-delete-user-data-in-content-moderator"></a>Exportar ou excluir dados do usuário no Content Moderator

O Content Moderator coleta dados de usuário para operar o serviço, mas os clientes precisam ter controle total sobre a exibição, a exportação e a exclusão dos dados usando a [Interface do Usuário de Análise](http://contentmoderator.cognitive.microsoft.com/) e as [APIs](https://docs.microsoft.com/azure/cognitive-services/content-moderator/api-reference).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

Para obter mais informações sobre como exportar e excluir dados do usuário em Content Moderator, veja a tabela a seguir.

| Dados | Operação de exportar | Operação de Exclusão |
| ---- | ---------------- | ---------------- |
| Informações da Conta (Chaves de Assinatura) | N/D | Exclua usando o portal do Azure (Assinaturas do Azure). Ou use o botão **Excluir Equipe** na página de configurações da Equipe de [Examinar Interface do Usuário](http://contentmoderator.cognitive.microsoft.com/). |
| Imagens para correspondência personalizada | [Obter IDs de imagem](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f676). As imagens são armazenadas em um formato de hash proprietário unidirecional e não é possível extrair as imagens reais. | [Excluir todas as imagens](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f686). Ou exclua o recurso Content Moderator usando o portal do Azure. |
| Termos para correspondência personalizada | [Obter todos os termos](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67e) | [Excluir todos os termos](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67d). Ou exclua o recurso Content Moderator usando o portal do Azure. |
| Marcas | N/D | Use o ícone **Excluir** disponível para cada marcação na página de configurações de Marcação de Interface do Usuário de Análise. Ou use o botão **Excluir Equipe** na página de configurações da Equipe de [Examinar Interface do Usuário](http://contentmoderator.cognitive.microsoft.com/). |
| Análises | [Obter análise](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2) | Use o botão **Excluir Equipe** na página de configurações da Equipe de [Interface do Usuário de Análise](http://contentmoderator.cognitive.microsoft.com/).
| Usuários | N/D | Use o ícone **Excluir** disponível para cada usuário na página de configurações de Equipe [Interface do Usuário de Análise](http://contentmoderator.cognitive.microsoft.com/). Ou use o botão **Excluir Equipe** na página de configurações da Equipe de [Examinar Interface do Usuário](http://contentmoderator.cognitive.microsoft.com/). |

