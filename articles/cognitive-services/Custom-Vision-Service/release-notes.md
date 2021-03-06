---
title: Notas de versão do Serviço de Visão Personalizada
services: cognitive-services
author: anrothMSFT
manager: corncar
ms.service: cognitive-services
ms.component: custom-vision
ms.date: 08/28/2018
ms.author: anroth
ms.openlocfilehash: 8423051fa5169eb8acddc7297e36188e297cb9e4
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/30/2018
ms.locfileid: "43288657"
---
# <a name="custom-vision-service-release-notes"></a>Notas de versão do Serviço de Visão Personalizada

## <a name="august-14-2018"></a>14 de agosto de 2018
- Adicionado widget "Iniciar" ao site customvision.ai para orientar os usuários através do treinamento do projeto. 
- Melhorias adicionais no pipeline de aprendizado de máquina para beneficiar projetos de vários rótulos (nova camada de perda).

## <a name="june-28-2018"></a>28 de junho de 2018
- Correções de bugs e melhorias de back-end.
- Habilitada classificação Multiclasse, para projetos em que as imagens possuem exatamente um rótulo. Em Previsões para o modo multiclasse, as Probabilidades serão somadas a uma (todas as imagens são classificadas entre as Marcas especificadas).

## <a name="june-13-2018"></a>13 de junho de 2018
- Atualização UX, focada na facilidade de uso e acessibilidade. 
- Melhorias no pipeline de aprendizado de máquina para beneficiar projetos de vários rótulos com um grande número de marcas.
- Corrigido o bug na exportação do TensorFlow. Habilitado o controle de versão do modelo exportado, para que as iterações possam ser exportadas mais de uma vez. 

## <a name="may-7-2018"></a>7 de maio de 2018
- Introdução da versão prévia do recurso de Detecção de Objetos para projetos de Avaliação Limitada.
- Atualização para APIs 2.0
- Camada S0 expandida para até 250 marcas e 50.000 imagens. 
- Melhorias significativas de back-end para o pipeline de aprendizado de máquina para projetos de classificação de imagem. Projetos treinados depois de 27 de abril de 2018 irão se beneficiar dessas atualizações.
- Adicionada a exportação de modelo para ONNX, para uso com o Windows ML.
- Adicionada a exportação de modelo para o Dockerfile. Isso permite que você baixe os artefatos para criar seus próprios contêineres do Windows ou Linux, incluindo um DockerFile, o modelo do TensorFlow e o código de serviço. 
- Para modelos recentemente treinados exportados para TensorFlow nos domínios Geral (Compacto) e Ponto de referência (Compacto), os [Valores Médios agora são (0, 0,0)](https://github.com/azure-samples/cognitive-services-android-customvision-sample), para manter a consistência em todos os projetos. 

## <a name="march-1-2018"></a>1º de março de 2018
- Inclusão da versão prévia paga e integração no Portal do Azure. Os projetos agora podem ser anexados aos recursos do Azure com a camada F0 (Gratuita) ou S0 (Standard). Introdução dos projetos da camada S0, que permite até 100 marcas e 25.000 imagens. 
- O back-end é alterado para o parâmetro de normalização/pipeline de aprendizado de máquina. Isso dará aos clientes um melhor controle das compensações de precisão e recall ao ajustar o Limite de Probabilidade. Como parte dessas alterações, o Limite de Probabilidade padrão no portal do CustomVision.ai foi definido para ser 50%.

## <a name="december-19-2017"></a>19 de dezembro de 2017

- Adicionada a exportação para o Android (TensorFlow), além de uma exportação lançada anteriormente para iOS (CoreML). Isso permite a exportação de um modelo compacto treinado para ser executado offline em um aplicativo.
- Adicionados os domínios “compactos” Varejo e Ponto de referência para habilitar a exportação de modelo para esses domínios.
- Lançamento das versões [API de Treinamento 1.2](https://southcentralus.dev.cognitive.microsoft.com/docs/services/f2d62aa3b93843d79e948fe87fa89554/operations/5a3044ee08fa5e06b890f11f) e [API de Previsão 1.1](https://southcentralus.dev.cognitive.microsoft.com/docs/services/57982f59b5964e36841e22dfbfe78fc1/operations/5a3044f608fa5e06b890f164). As APIs atualizadas oferecem suporte à exportação de modelo, uma nova operação de Previsão que não salva as imagens em “Previsões” e introdução das operações em lote para a API de Treinamento.
- Ajustes de experiência do usuário, incluindo a capacidade de ver qual domínio foi usado para treinar uma iteração.
- Atualização do [SDK e exemplo de C#](https://github.com/Microsoft/Cognitive-CustomVision-Windows).

