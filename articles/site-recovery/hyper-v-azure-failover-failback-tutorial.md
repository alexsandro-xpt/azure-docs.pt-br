---
title: Executar failover e failback de VMs do Hyper-V replicadas no Azure com o Site Recovery | Microsoft Docs
description: Saiba como executar failover de VMs do Hyper-V no Azure e failback no site local com o Azure Site Recovery
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: raynew
ms.openlocfilehash: f758939964045ed373703a211d4cbef00f0e42e7
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/09/2018
ms.locfileid: "37919539"
---
# <a name="failover-and-failback-hyper-v-vms-replicated-to-azure"></a>Fazer failover e failback em VMs do Hyper-V replicadas no Azure

Este tutorial descreve como fazer failover de uma VM do Hyper-V no Azure. Após fazer failover, faça failback para o site local quando ele estiver disponível. Neste tutorial, você aprenderá como:

> [!div class="checklist"]
> * Verificar as propriedades da VM Hyper-V para saber se elas estão em conformidade com os requisitos do Azure
> * Executar um failover para o Azure
> * Failback do Azure para o local
> * Faça replicação inversa de VMs locais para iniciar a replicação para o Azure novamente

Este tutorial é o quinto tutorial de uma série. Presume-se que você já tenha concluído as tarefas nos tutoriais anteriores.    

1. [Preparar o Azure](tutorial-prepare-azure.md)
2. [Preparar Hyper-V local](tutorial-prepare-on-premises-hyper-v.md)
3. Configurar a recuperação de desastres para as [VMs do Hyper-V](tutorial-hyper-v-to-azure.md) ou para [VMs do Hyper-V gerenciadas em nuvens do System Center VMM](tutorial-hyper-v-vmm-to-azure.md)
4. [Realizar uma simulação de recuperação de desastre](tutorial-dr-drill-azure.md)

## <a name="prepare-for-failover-and-failback"></a>Preparar-se para failover e failback

Certifique-se de que não haja instantâneos na VM e que a VM local esteja desativada durante o failback. Isso ajuda a garantir a consistência dos dados durante a replicação. Não ative a VM local durante o failback. 

O failover e o failback têm três estágios:

1. **Failover para Azure**: fazer failover de VMs do Hyper-V do site local para Azure.
2. **Failback para local**: fazer failover de VMs do Azure para o site local, quando disponível. Ele inicia a sincronização de dados do Azure para o local e, após a conclusão, ele abre as VMs localmente.  
3. **Replicação inversa de VMs locais**: após o failback local, replique inversamente as VMs locais para começar a replicá-las no Azure.

## <a name="verify-vm-properties"></a>Verificar as propriedades da VM

Antes do failover, verifique as propriedades da VM e verifique se a VM atende aos [requisitos do Azure](hyper-v-azure-support-matrix.md#replicated-vms).

Em **Itens Protegidos**, clique em **Itens Replicados** > VM.

2. No painel **Item Replicado**, há um resumo das informações da VM, o status de integridade e os últimos pontos de recuperação disponíveis. Clique em **Propriedades** para exibir mais detalhes.

3. Em **Computação e Rede**, você pode modificar o nome do Azure, o grupo de recursos, o tamanho do destino, o [conjunto de disponibilidade](../virtual-machines/windows/tutorial-availability-sets.md) e as configurações de disco gerenciado.

4. Você pode exibir ou modificar as configurações de rede, incluindo a rede/sub-rede na qual a VM do Azure estará localizada após o failover e o endereço IP que será atribuído a ela.

5. Em **Discos**, é possível visualizar as informações sobre o sistema operacional e os discos de dados na VM.

## <a name="failover-to-azure"></a>Failover para o Azure

1. Em **Configurações** > **Itens replicados**, clique na VM > **Failover**.
2. Em **Failover**, selecione o **Último** ponto de recuperação. 
3. Selecione **Desligar o computador antes do início do failover**. O Site Recovery tenta desligar as VMs de origem antes de acionar o failover. O failover continuará mesmo o desligamento falhar. Você pode acompanhar o progresso do failover na página **Trabalhos** .
4. Depois de verificar o failover, clique em **Confirmar**. Exclui todos os pontos de recuperação disponíveis.

> [!WARNING]
> **Não cancele um failover em andamento**: se você cancelar em andamento, o failover será interrompido, mas a VM não será replicada novamente.

## <a name="failback-azure-vm-to-on-premises-and-reverse-replicate-the-on-premises-vm"></a>Fazer failback de VM do Azure para local e replicação inversa de VM local

A operação de failback é basicamente um failover do Azure para o site local e, na replicação inversa, ele inicia novamente a replicação de VMs do site local para o Azure.

1. Em **Configurações** > **Itens Replicados**, clique na VM > **Failover Planejado**.
2. Em **Confirmar Failover Planejado**, verifique a direção do failover (do Azure) e selecione os locais de origem e de destino.
3. Selecione **Sincronizar os dados antes do failover (sincronizar apenas alterações delta)**. Essa opção minimiza o tempo de inatividade da VM porque sincroniza sem desligar a VM.
4. Inicie o failover. Você pode acompanhar o progresso do failover na guia **Trabalhos** .
5. Depois que a sincronização de dados inicial for concluída e você estiver pronto para desligar as VMs no Azure, clique em **Trabalhos** > nome do trabalho de failover planejado > **Concluir Failover**. Ele desliga a VM do Azure, transfere as alterações mais recentes para o local e inicia a VM local.
6. Faça logon na VM local para verificar se ela está disponível conforme o esperado.
7. A VM local agora está em um estado de **Confirmação Pendente**. Clique em **Confirmar**. Ele exclui as VMs do Azure e seus discos e prepara a VM local para a replicação inversa.
Para iniciar a replicação da VM local para o Azure, habilite a **Replicação Inversa**. Ele dispara a replicação de alterações delta ocorridas desde que a VM do Azure foi desativada.  
