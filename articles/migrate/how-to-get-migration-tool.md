---
title: Migrar máquinas após a avaliação com Migrações para Azure | Microsoft Docs
description: Descreve como obter recomendações para a migração de máquinas depois de executar uma avaliação com o serviço Migrações para Azure.
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: article
ms.date: 09/17/2018
ms.author: raynew
ms.openlocfilehash: 0b02ae4b75426b379ad7c124f5ddeb053c142ce6
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/17/2018
ms.locfileid: "45730287"
---
# <a name="migrate-machines-after-assessment"></a>Migrar máquinas após a avaliação


As [Migrações para Azure](migrate-overview.md) avalia as máquinas locais para verificar se elas são adequados para a migração para o Azure e fornece estimativas de dimensionamento e custo para executar a máquina no Azure. Atualmente, as Migrações para Azure só avaliam máquinas para migração. A migração em si é executada usando outros serviços do Azure.

Este artigo descreve como obter sugestões para uma ferramenta de migração depois de executar uma avaliação de migração.

## <a name="migration-tool-suggestion"></a>Sugestão de ferramenta de migração

Para obter sugestões sobre ferramentas de migração, você precisa fazer uma descoberta profunda do ambiente local. A descoberta profunda é feita pela instalação de agentes nos computadores locais.  

1. Crie um projeto de Migrações para Azure, descubra máquinas locais e crie uma avaliação de migração. [Saiba mais](tutorial-assessment-vmware.md).
2. Baixe e instale os agentes de Migrações para Azure em cada máquina local para a qual você deseja ver um método de migração recomendado. [Siga este procedimento](how-to-create-group-machine-dependencies.md#prepare-for-dependency-visualization) para instalar os agentes.
2. Identifique as máquinas locais adequadas para a migração lift-and-shift. Essas são as VMs que não exigem nenhuma mudança para os aplicativos executados nelas, e podem ser migradas como estão.
3. Para a migração lift-and-shift, sugerimos o uso do Azure Site Recovery. [Saiba mais](../site-recovery/tutorial-migrate-on-premises-to-azure.md). Como alternativa, você pode usar ferramentas de terceiros que ofereçam suporte à migração para o Azure.
4. Se você tiver máquinas locais que não sejam adequadas para uma migração lift-and-shift, ou seja, se você quiser migrar o aplicativo específico em vez de uma VM inteira, poderá usar outras ferramentas de migração. Por exemplo, sugerimos o [serviço Migração de Banco de Dados do Azure](https://azure.microsoft.com/campaigns/database-migration/) se você quiser migrar bancos de dados locais, como um SQL Server, um MySQL ou um Oracle para o Azure.


## <a name="review-suggested-migration-methods"></a>Examinar os métodos de migração sugeridos

1. Antes de poder obter um método de migração sugerido, você precisa criar um projeto de Migrações para Azure, descobrir as máquinas locais e executar uma avaliação de migração. [Saiba mais](tutorial-assessment-vmware.md).
2. Após a criação da avaliação, exiba-a no projeto > **Visão geral** > **Painel**. Clique em **Preparação para Avaliação**

    ![Preparação para avaliação](./media/tutorial-assessment-vmware/assessment-report.png)  

3. Em **Ferramenta Sugerido**, examine as sugestões de ferramentas que você pode usar para a migração.

    ![Ferramenta sugerida](./media/tutorial-assessment-vmware/assessment-suitability.png)




## <a name="next-steps"></a>Próximas etapas

[Saiba mais](concepts-assessment-calculation.md) sobre como as avaliações são calculadas.
