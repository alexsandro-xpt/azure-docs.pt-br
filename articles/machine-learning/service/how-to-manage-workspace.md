---
title: Criar e gerenciar Workspaces do Azure Machine Learning
description: Saiba como criar, exibir e excluir Workspaces do Azure Machine Learning no portal do Azure.
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: shipatel
author: shivp950
ms.date: 09/24/2018
ms.openlocfilehash: 7d01a2e3ebd46315966c82a43a17ffc5b329b829
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954340"
---
# <a name="create-and-manage-azure-machine-learning-workspaces"></a>Criar e gerenciar Workspaces do Azure Machine Learning

Neste artigo, você vai criar, exibir e excluir [**Workspaces do Azure Machine Learning**](concept-azure-machine-learning-architecture.md#workspace) no portal do Azure para o [serviço do Azure Machine Learning](overview-what-is-azure-ml.md).  Também é possível criar e excluir espaços de trabalho [usando a CLI](reference-azure-machine-learning-cli.md) ou [com o código Python](http://aka.ms/aml-sdk).

Para criar um espaço de trabalho, você precisa de uma assinatura do Azure. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

## <a name="create-a-workspace"></a>Criar um espaço de trabalho 

[!INCLUDE [aml-create-portal](../../../includes/aml-create-in-portal.md)]

## <a name="view-a-workspace"></a>Exibir um espaço de trabalho

1. No canto superior esquerdo do portal, selecione **Todos os serviços**. 

1. No campo de filtro em **Todos os serviços**, digite **Workspace do Machine Learning**.  

   ![procure o espaço de trabalho do Azure Machine Learning](media/how-to-manage-workspace/allservices-search1.png)

1. Nos resultados do filtro, selecione **Workspace do Machine Learning** para exibir uma lista dos espaços de trabalho. 

   ![procure o espaço de trabalho do Azure Machine Learning](media/how-to-manage-workspace/allservices-search.PNG)

1. Examine a lista de espaços de trabalho encontrada. É possível filtrar com base na assinatura, grupos de recursos e locais.  

   ![Lista de espaços de trabalho do Azure Machine Learning](media/how-to-manage-workspace/allservices_view_workspace.PNG)

1. Selecione o espaço de trabalho que você acabou de criar para exibir suas propriedades.

   ![png](media/how-to-manage-workspace/allservices_view_workspace_full.PNG)

## <a name="delete-a-workspace"></a>Excluir um espaço de trabalho

Use o botão Excluir na parte superior do espaço de trabalho que você deseja excluir.

  ![png](media/how-to-manage-workspace/delete-workspace.png)


## <a name="clean-up-resources"></a>Limpar recursos 

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>Próximas etapas

Siga o tutorial completo para aprender a usar um espaço de trabalho para criar, treinar e implantar modelos com o serviço do Azure Machine Learning.

> [!div class="nextstepaction"]
> [Tutorial: Treinar modelos](tutorial-train-models-with-aml.md)
