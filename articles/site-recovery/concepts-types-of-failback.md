---
title: Failback no Azure Site Recovery | Microsoft Docs
description: Este artigo fornece uma visão geral de vários tipos de failback e advertências a serem consideradas enquanto executa failback para o local no serviço do Azure Site Recovery.
services: site-recovery
author: rajani-janaki-ram
manager: guaravd
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: rajanki
ms.openlocfilehash: 2a9ee380fc16c4088d98875dd465509c4023d037
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/09/2018
ms.locfileid: "37920382"
---
# <a name="overview-of-failback"></a>Visão geral do failback

Após a realização do failover para o Azure, será possível executar o failback para seu site local. Há dois tipos diferentes de failback que são possíveis com o Azure Site Recovery: 

- Failback para o local original 
- Failback para um local alternativo

Se você fez failover de uma máquina virtual VMware, você pode executar failback para a mesma máquina de virtual do local de origem se ele ainda existe. Nessa situação, apenas as alterações passarão por failback. Esse cenário é conhecido como **recuperação no local original**. Se não houver máquina virtual local, o cenário será uma **recuperação de local alternativo**.

> [!NOTE]
> Somente será possível executar failback para o servidor de Configuração e vCentre original. Você não poderá implantar um novo servidor de Configuração e usá-lo em failback. Além disso, não será possível adicionar um novo vCenter ao servidor de Configuração existente e o failback no novo vCenter.

## <a name="original-location-recovery-olr"></a>OLR (Recuperação no Local Original)
Se você optar por executar failback para a máquina virtual original, as condições a seguir deverão ser atendidas:

* Se a máquina virtual for gerenciada por um servidor vCenter, o host do ESX de Destino Mestre deve ter acesso ao repositório de dados das máquinas virtuais.
* Se a máquina virtual estiver em um host ESX, mas não é gerenciada pelo vCenter, o disco rígido da máquina virtual deve ser em um repositório de dados que o host do destino mestre pode acessar.
* Se sua máquina virtual estiver em um host ESX e não usar o vCenter, complete a descoberta do host ESX do destino mestre antes de protegê-la novamente. Isso se aplicará se você também estiver realizando o failback de servidores físicos.
* Você pode executar failback para uma rede de área de armazenamento virtual (vSAN) ou um disco com base no mapeamento (RDM) se os discos já existem e estão conectados à máquina virtual local de dispositivo bruto.

> [!IMPORTANT]
> É importante habilitar disk.enableUUID= TRUE para que, durante o failback, o serviço do Azure Site Recovery seja capaz de identificar o VMDK original na máquina virtual para qual as alterações pendentes serão gravadas. Se esse valor não for configurado como TRUE, o serviço tentará identificar o VMDK local apropriado no melhor esforço. Se o VMDK correto não for localizado, ele criará um disco adicional onde os dados serão gravados.

## <a name="alternate-location-recovery-alr"></a>ALR (Recuperação de Local Alterativo)
Se a máquina virtual no local não existe antes de proteger novamente a máquina virtual, o cenário é chamado uma recuperação em local alternativo. O fluxo de trabalho Proteja cria novamente a máquina virtual no local. Isso também fará um download de dados completo.

* Ao realizar failback para um local alternativo, a máquina virtual é recuperada para o mesmo host ESX no qual o servidor de destino principal está implantado. O armazenamento de dados que é usado para criar o disco será o mesmo armazenamento de dados que foi selecionado quando proteger novamente a máquina virtual.
* Você pode executar failback somente para um repositório de dados de sistema de arquivo de máquina virtual (VMFS) ou vSAN. Se você tiver um RDM, nova proteção e failback não funcionarão.
* O proteger novamente envolve uma grande transferência inicial de dados seguida pelas alterações. Esse processo existe porque a máquina virtual não existe no local. Os dados completos deverão ser replicados de volta. O proteger novamente levará mais tempo do que em uma recuperação no local original.
* Não é possível executar failback para discos baseados em RDM. Somente novos discos de máquina virtual (VMDKs) podem ser criados em um repositório de dados VMFS/vSAN.

> [!NOTE]
> Um computador físico, após failover no Azure, somente poderá executar failback como uma máquina virtual VMware. Isso segue o mesmo fluxo de trabalho que a recuperação de local alternativo. Descubra, pelo menos, um servidor de destino mestre juntamente com os hosts ESX/ESXi necessários para os quais você precisa fazer failback.

## <a name="next-steps"></a>Próximas etapas

Siga as etapas para realizar a [operação de failback](vmware-azure-failback.md).

