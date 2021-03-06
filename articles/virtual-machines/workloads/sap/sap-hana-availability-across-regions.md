---
title: Disponibilidade do SAP HANA em regiões do Azure | Microsoft Docs
description: Uma visão geral das considerações de disponibilidade ao executar o SAP HANA em VMs do Azure em várias regiões do Azure.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/26/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: edbd1885dd529e4ccd38f2012d56865a2147f64d
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/05/2018
ms.locfileid: "30842264"
---
# <a name="sap-hana-availability-across-azure-regions"></a>Disponibilidade do SAP HANA entre regiões do Azure

Este artigo descreve os cenários relacionados à disponibilidade do SAP HANA em diferentes regiões do Azure. Devido à distância entre regiões do Azure, a configuração da disponibilidade do SAP HANA em várias regiões do Azure envolve considerações especiais.

## <a name="why-deploy-across-multiple-azure-regions"></a>Por que implantar em várias regiões do Azure?

As regiões do Azure geralmente são separadas por grandes distâncias. Dependendo da região geopolítica, a distância entre as regiões do Azure pode ser de centenas de quilômetros ou mesmo milhares de quilômetros, como nos Estados Unidos. Devido à distância, o tráfego entre os recursos implantados em duas regiões do Azure diferentes passa por uma latência na viagem de ida e volta de rede significativa. A latência é significativa o suficiente para excluir troca de dados síncrona entre duas instâncias do SAP HANA sob cargas de trabalhos típicas do SAP. 

Por outro lado, as organizações geralmente têm um requisito de distância entre o local do datacenter primário e um datacenter secundário. Um requisito de distância ajuda a fornecer disponibilidade se ocorrer um desastre natural em um local geográfico mais amplo. São exemplos os furacões que atingiram o Caribe e a Flórida em setembro e outubro de 2017. Sua organização pode ter pelo menos um requisito de distância mínima. Para a maioria dos clientes do Azure, essa definição de distância mínima requer a criação de disponibilidade em [Regiões do Azure](https://azure.microsoft.com/regions/). Como a distância entre duas regiões do Azure é muito grande para usar o modo de replicação síncrona do HANA, os requisitos de RTO e RPO poderão forçá-lo a implantar configurações de disponibilidade dentro de uma região e, em seguida, complementar com implantações adicionais em uma segunda região.

Outro aspecto a considerar nesse cenário é o failover e o redirecionamento de cliente. A suposição é que um failover entre instâncias do SAP HANA em duas regiões do Azure diferentes é sempre um failover manual. Como o modo de replicação de sistema do SAP HANA é configurado como assíncrono, há um potencial de que os dados confirmados na instância do HANA primário ainda não tenham feito a instância do HANA secundário. Portanto, o failover automático não é uma opção para configurações onde a replicação for assíncrona. Mesmo com failover controlado manualmente, como em um exercício de failover, será necessário tomar medidas para certificar-se de que todos os dados confirmados no lado primário chegaram à instância secundária antes de movimentar manualmente para a outra região do Azure.
 
A Rede Virtual do Azure usa um intervalo de endereço IP diferente. Os endereços IP são implantados na segunda região do Azure. Por isso, você o precisa alterar a configuração de cliente do SAP HANA ou, preferencialmente, precisa criar as etapas para alterar a resolução de nome. Dessa forma, os clientes esão redirecionados para o endereço IP do servidor do novo site secundário. Para obter mais informações, consulte o artigo do SAP sobre a [Recuperação da conexão do cliente após tomada de controle](https://help.sap.com/doc/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/c93a723ceedc45da9a66ff47672513d3.html).   

## <a name="simple-availability-between-two-azure-regions"></a>Disponibilidade simples entre duas regiões do Azure

Você pode optar por não implementar nenhuma configuração de disponibilidade dentro de uma única região, mas ainda ter a demanda para que a carga de trabalho seja atendida se ocorrer um desastre. Casos típicos para sistemas como esse são sistemas que não são de produção. Embora seja sustentável ter o sistema inoperante por meio dia ou mesmo por um dia, não é possível permitir que o sistema fique indisponível por 48 horas ou mais. Para tornar a instalação mais barata, execute outro sistema que seja ainda menos importante na VM. O outro sistema funciona como destino. Você pode dimensionar a VM na região secundária para que seja menor e optar por não pré-carregar os dados. Como o failover é manual e implica muitas outras etapas para failover na pilha de aplicativo completa, o tempo adicional para encerrar a VM, redimensioná-la e então iniciá-la novamente, é aceitável.

> [!NOTE]
> Mesmo que você não use o pré-carregamento de dados no destino de replicação do sistema do HANA, é necessário pelo menos de 64 GB de memória. Você também precisa de memória suficiente além de 64 GB para manter os dados de rowstore na memória da instância de destino.

![Diagrama de duas VMs em duas regiões](./media/sap-hana-availability-two-region/two_vm_HSR_async_2regions_nopreload.PNG)

> [!NOTE]
> Nessa configuração, não é possível fornecer um RPO=0, pois o modo de replicação de sistema do HANA é assíncrono. Se for necessário fornecer um RPO=0, essa configuração não será a configuração preferencial.

Uma pequena alteração que você pode fazer na configuração pode ser configurar os dados como pré-carregamento. No entanto, dada a natureza manual do failover e o fato de que as camadas de aplicativo também precisam se mover para a segunda região, não é adequado pré-carregar dados. 

## <a name="combine-availability-within-one-region-and-across-regions"></a>Combinar disponibilidade dentro de uma região e entre regiões 

Uma combinação de disponibilidade dentro e entre regiões pode ser conduzida por estes fatores:

- Um requisito para RPO=0 dentro de uma região do Azure.
- A organização não quer ou não pode ter operações globais sendo afetadas por uma grande catástrofe natural que afeta uma região maior. Esse foi o caso de alguns furacões que atingiram o Caribe nos últimos anos.
- Regulamentos que exigem distâncias entre sites primários e secundários que estão claramente além do que as áreas de disponibilidade do Azure podem fornecer.

Nesses casos, você pode configurar para que o SAP chame uma [Configuração de replicação de sistema multicamadas do SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/ca6f4c62c45b4c85a109c7faf62881fc.html) usando a replicação do sistema do HANA. A arquitetura seria semelhante a:

![Diagrama de três VMs em duas regiões](./media/sap-hana-availability-two-region/three_vm_HSR_async_2regions_ha_and_dr.PNG)

Essa configuração fornece um RPO=0, com baixo RTO, dentro da região primária. A configuração também fornece o RPO razoável se uma mudança para a segunda região estiver envolvida. Os tempos do RTO na segunda região dependem de se os dados são ou não pré-carregados. Muitos clientes usam a VM na região secundária para executar um sistema de teste. Nesse caso de uso, os dados não podem ser pré-carregados.

> [!NOTE]
> Como você está utilizando o modo de operação **logreplay** para replicação de sistema do SAP HANA indo do nível 1 ao nível 2 (replicação síncrona na região primária), a replicação entre o nível 2 e nível 3 (replicação para o site secundário) não pode estar no modo de operação **delta_datashipping**. Para obter detalhes sobre os modos de operação e algumas restrições, consulte o artigo [Modos de operação para replicação de sistema do SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/627bd11e86c84ec2b9fcdf585d24011c.html). 

## <a name="next-steps"></a>Próximas etapas

Para obter orientação passo a passo sobre como definir essas configurações no Azure, consulte:

- [Configurar a replicação de sistema do SAP HANA em VMs do Azure](sap-hana-high-availability.md)
- [Alta disponibilidade para SAP HANA usando replicação de sistema](https://blogs.sap.com/2018/01/08/your-sap-on-azure-part-4-high-availability-for-sap-hana-using-system-replication/)

 



 
  
