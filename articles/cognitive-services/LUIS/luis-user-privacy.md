---
title: Exportação e a exclusão de dados do cliente – LUIS – Serviços Cognitivos do Azure | | Microsoft Docs
description: Referência de exportação e exclusão de dados do cliente do LUIS (Serviço Inteligente de Reconhecimento Vocal).
services: cognitive-services
author: v-geberr
manager: kaiqb
ms.service: cognitive-services
ms.technology: luis
ms.topic: article
ms.date: 05/23/2018
ms.author: diberry
ms.openlocfilehash: c6fba28ea3a4b24f02b62b6c3f124569378e5bc8
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/28/2018
ms.locfileid: "43130519"
---
# <a name="export-and-delete-your-customer-data-in-language-understanding-luis-in-cognitive-services"></a>Exportar e excluir dados do cliente no LUIS (Serviço Inteligente de Reconhecimento Vocal) nos Serviços Cognitivos

## <a name="summary-of-customer-data-request-features"></a>Resumo dos recursos de solicitação de dados do cliente
O LUIS (Serviço Inteligente de Reconhecimento Vocal) preserva o conteúdo do cliente para operar o serviço, mas o usuário do LUIS tem controle total sobre a exibição, a exportação e a exclusão dos próprios dados. Isso pode ser feito por meio do [portal](luis-reference-regions.md) da Web do LUIS ou de [APIs Programáticas do LUIS](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

O conteúdo do cliente foi armazenado criptografado no armazenamento do Azure regional da Microsoft e inclui:

- Conteúdo da conta de usuário coletado no momento do registro
- Dados de treinamento necessários para compilar modelos (ou seja, intenção e entidades)
- Consultas do usuário registradas em log em tempo de execução para ajudar a melhorar os modelos do usuário
  - Os usuários podem desligar o registro em log de consulta anexando `&log=false` à solicitação; detalhes [aqui](luis-resources-faq.md#how-can-i-disable-the-logging-of-utterances)

## <a name="deleting-customer-data"></a>Excluindo os dados do cliente
Os usuários do LUIS têm controle total para excluir qualquer conteúdo de usuário, seja por meio do portal da Web do LUIS, seja pelas APIs Programáticas do LUIS. A tabela a seguir exibe links para ajudar a ajudar com ambos:

| | **Conta do Usuário** | **Aplicativo** | **Enunciado(s)** | **Consultas do usuário final** |
| --- | --- | --- | --- | --- |
| **Portal** | [Link](luis-how-to-account-settings.md) | [Link](luis-how-to-start-new-app.md#delete-app) | [Link](luis-how-to-start-new-app.md#delete-app) | [Link](luis-how-to-start-new-app.md#delete-app) |
| **APIs** | [Link](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c4c) | [Link](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c39) | [Link](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0b) | [Link](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/58b6f32139e2bb139ce823c9) |


## <a name="exporting-customer-data"></a>Exportando os dados do cliente
Os usuários do LUIS têm controle total para exibir os dados no portal, no entanto, eles devem ser exportados por meio das APIs Programáticas do LUIS. A tabela a seguir exibe links para ajudar com exportações de dados por meio das APIs Programáticas do LUIS:

| | **Conta do Usuário** | **Aplicativo** | **Enunciado(s)** | **Consultas do usuário final** |
| --- | --- | --- | --- | --- |
| **APIs** | [Link](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c48) | [Link](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c40) | [Link](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0a) | [Link](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c36) |


## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Referência de regiões do LUIS](./luis-reference-regions.md)
