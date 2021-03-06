---
title: Limites do QnA Maker – Serviços Cognitivos do Azure | Microsoft Docs
description: Limites do QnA Maker
services: cognitive-services
author: nstulasi
manager: sangitap
ms.service: cognitive-services
ms.component: QnAMaker
ms.topic: article
ms.date: 05/07/2018
ms.author: saneppal
ms.openlocfilehash: 93471faab9aac94616c770cbee21fb0364f73639
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/03/2018
ms.locfileid: "39501853"
---
# <a name="qna-maker-limits"></a>Limites do QnA Maker
Lista abrangente dos limites no QnA Maker.

## <a name="knowledge-bases"></a>Bases de Dados de Conhecimento

* Número máximo de bases de dados de conhecimento com base em [limites de camada do Azure Search](https://docs.microsoft.com/en-us/azure/search/search-limits-quotas-capacity)

|**Camada do Azure Search** | **Gratuito** | **Básico** |**S1** | **S2**| **S3** |**S3 HD**|
|---|---|---|---|---|---|----|
|Número máximo de bases de dados de conhecimento publicadas permitidas (Máx. de índices – 1 (reservado para teste)|2|14|49|199|199|2999|

## <a name="extraction-limits"></a>Limites de extração
* Número máximo de arquivos que podem ser extraídos e o tamanho máximo do arquivo: consulte [Preços de QnAMaker](https://azure.microsoft.com/en-in/pricing/details/cognitive-services/qna-maker/)
* Número máximo de links profundos que podem ser rastreados para extração de QnAs de páginas HTML de perguntas Frequentes: 20

## <a name="metadata-limits"></a>Limites de metadados
* Número máximo de campos de metadados por base de dados de conhecimento com base nos [limites de camada do Azure Search](https://docs.microsoft.com/en-us/azure/search/search-limits-quotas-capacity)

|**Camada do Azure Search** | **Gratuito** | **Básico** |**S1** | **S2**| **S3** |**S3 HD**|
|---|---|---|---|---|---|----|
|Máximo de campos de metadados por serviço QnA Maker (entre todas as bases de dados de conhecimento)|1000|100*|1000|1000|1000|1000|

## <a name="knowledge-base-content-limits"></a>Limites de conteúdo da Base de Dados de Conhecimento
Limites gerais sobre o conteúdo na base de dados de conhecimento:
* Tamanho do texto de resposta: 250.000
* Tamanho do texto da pergunta: 1.000
* Tamanho do texto de chave/valor dos metadados: 100
* Caracteres com suporte para nome de metadados: alfabéticos, dígitos e _  
* Caracteres com suporte para valor de metadados: todos, exceto : e | 
* Tamanho do nome do arquivo: 200
* Formatos de arquivo com suporte: ".tsv", ".pdf", ".txt", ".docx", ".xlsx".
* Número máximo de perguntas alternativas: 100
* Número máximo de pares de pergunta-resposta: depende da [camada do Azure Search](https://docs.microsoft.com/en-in/azure/search/search-limits-quotas-capacity#document-limits) escolhida 

## <a name="create-knowledge-base-call-limits"></a>Criar limites de chamada da base de dados de conhecimento:
Eles representam os limites de cada ação de criação da base de dados de conhecimento; ou seja, clicar em *Criar KB* ou chamar a API CreateKnowledgeBase.
* Número máximo de perguntas alternativas por resposta: 100
* Número máximo de URLs: 10
* Número máximo de arquivos: 10

## <a name="update-knowledge-base-call-limits"></a>Atualizar limites de chamada da base de dados de conhecimento
Eles representam os limites de cada ação de atualização; ou seja, clique em *Salvar e treinar* ou chame a API UpdateKnowledgeBase.
* Tamanho de cada nome de origem: 300
* Número máximo de perguntas alternativas adicionadas ou excluídas: 100
* Número máximo de campos de metadados adicionados ou excluídos: 10
* Número máximo de URLs que podem ser atualizadas: 5
