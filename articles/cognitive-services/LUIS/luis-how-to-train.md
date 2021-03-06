---
title: Treinar seu aplicativo de LUIS
titleSuffix: Azure Cognitive Services
description: O treinamento é o processo de ensinar seu aplicativo de LUIS (Serviço Inteligente de Reconhecimento Vocal) para melhorar seu reconhecimento vocal de idioma natural. Treine seu aplicativo de LUIS após atualizações ao modelo, como adicionar, editar, rotular ou excluir entidades, intenções ou enunciados.
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/06/2018
ms.author: diberry
ms.openlocfilehash: 42cff3dd8237598da5aa71ed1a4d6462c5b4c25d
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/07/2018
ms.locfileid: "44049124"
---
# <a name="train-your-luis-app"></a>Treinar seu aplicativo de LUIS

O treinamento é o processo de ensinar seu aplicativo de LUIS (Serviço Inteligente de Reconhecimento Vocal) para melhorar seu reconhecimento vocal de idioma natural. Treine seu aplicativo de LUIS após atualizações ao modelo, como adicionar, editar, rotular ou excluir entidades, intenções ou enunciados. 

<!--
When you train a LUIS app by example, LUIS generalizes from the examples you have labeled, and it learns to recognize the relevant intents and entities. This teaches LUIS to improve classification accuracy in the future. -->

Treinar e [testar](luis-concept-test.md) um aplicativo é um processo iterativo. Após treinar seu aplicativo de LUIS, teste-o com enunciados de amostra para ver se as intenções e as entidades são corretamente reconhecidas. Caso não sejam, faça atualizações ao aplicativo de LUIS, treine e teste novamente. 

## <a name="train-your-app"></a>Treinar seu aplicativo
Para começar o processo iterativo, primeiro você precisa treinar seu aplicativo de LUIS pelo menos uma vez. Verifique se que cada intenção tem pelo menos um enunciado antes do treinamento.

1. Acesse seu aplicativo selecionando seu nome na página **Meus Aplicativos**. 

2. Em seu aplicativo, selecione **Treinar** no painel superior. 

3. Quando o treinamento estiver concluído, uma barra verde de notificação aparecerá na parte superior do navegador.

<!-- The following note refers to what might cause the error message "Training failed: FewLabels for model: <ModelName>" -->

>[!NOTE]
>Se você tiver uma ou mais intenções no seu aplicativo que não contenham enunciados de exemplo, você não poderá treinar seu aplicativo. Adicione enunciados para todas as suas intenções. Para obter mais informações, veja [Adicionar enunciados de exemplo](luis-how-to-add-example-utterances.md).

## <a name="next-steps"></a>Próximas etapas

* [Rotular enunciados sugeridos com o LUIS](luis-how-to-review-endoint-utt.md) 
* [Usar recursos para melhorar o desempenho do aplicativo de LUIS](luis-how-to-add-features.md) 