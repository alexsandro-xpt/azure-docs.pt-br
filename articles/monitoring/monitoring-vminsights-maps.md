---
title: Como exibir dependências de aplicativo com o Azure Monitor para VMs | Microsoft Docs
description: O Mapa é um recurso do Azure Monitor para VMs que descobre automaticamente os componentes do aplicativo nos sistemas Windows e Linux e mapeia a comunicação entre os serviços. Este artigo fornece detalhes sobre como usá-lo em uma variedade de cenários.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/17/2018
ms.author: magoedte
ms.openlocfilehash: 30f9a874853f4cb492d7a562d0d2439b73cf62dc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989967"
---
# <a name="using-azure-monitor-for-vms-map-to-understand-application-components"></a>Usando o Azure Monitor para o Mapa de VMs para entender os componentes do aplicativo
A exibição dos componentes de aplicativo descobertos em máquinas virtuais do Windows e do Linux em execução no Azure em seu ambiente pode ser feita de duas maneiras com o Azure Monitor para VMs: em uma máquina virtual diretamente ou entre grupos de VMs do Azure Monitor. 

Este artigo ajudará você a entender a experiência entre as duas perspectivas e como usar o recurso Mapa. Para obter informações sobre como configurar o Azure Monitor para VMs, confira [Habilitar o Azure Monitor para VMs](monitoring-vminsights-onboard.md).

## <a name="sign-in-to-azure"></a>Entrar no Azure
Entre no Portal do Azure em [https://portal.azure.com](https://portal.azure.com).

## <a name="introduction-to-map-experience"></a>Introdução à experiência de Mapa
Antes de se aprofundar na exibição do Mapa para uma máquina virtual individual ou um grupo de VMs, é importante fornecermos uma breve introdução ao recurso para entender como as informações são apresentadas e o que as visualizações representam.  

Se você selecionar o recurso Mapa diretamente em uma VM ou no Azure Monitor, ele apresentará uma experiência consistente.  A única diferença é que no Azure Monitor você poderá ver todos os membros de um aplicativo de várias camadas ou um cluster em um mapa.

Os Mapas visualizam as dependências de VMs com a descoberta dos processos em execução com conexões de rede ativas entre servidores, da latência de conexão de entrada e saída e das portas em qualquer arquitetura conectada a TCP em um intervalo de tempo especificado.  A expansão de uma VM mostra os detalhes do processo e somente os processos que se comunicam com a VM são exibidos. A contagem de clientes front-end que se conectam à máquina virtual é indicada com o Grupo de Clientes. A contagem de servidores back-end aos quais a VM se conecta é indicada nos Grupos de Portas do Servidor. Expanda um Grupo de Portas do Servidor para ver uma lista detalhada de servidores conectados por meio dessa porta.  

Quando você clica na máquina virtual, o painel **Propriedades** é expandido no lado direito para mostrar as propriedades do item selecionado, como informações do sistema relatadas pelo sistema operacional, propriedades da VM do Azure e um gráfico de rosca que resume as conexões descobertas. 

![Propriedades do sistema do computador](./media/monitoring-vminsights-maps/properties-pane-01.png)

No lado direito do painel, clique no ícone **Eventos de Log** para alternar o foco do painel para mostrar uma lista de tabelas cujos dados coletados da VM foram enviados ao Log Analytics e estão disponíveis para consulta.  Ao clicar em um dos tipos de registro listados, a página **Logs** será aberta para exibir os resultados desse tipo com uma consulta pré-configurada filtrada na máquina virtual específica.  

![Lista de pesquisa de logs no painel Propriedades](./media/monitoring-vminsights-maps/properties-pane-logs-01.png)

Feche *Logs** e retorne ao painel **Propriedades** e selecione **Alertas** para exibir os alertas gerados pelos alertas para a VM do critério de integridade. O Mapa é integrado aos Alertas do Azure para mostrar os alertas acionados para o servidor selecionado no intervalo de tempo selecionado. O servidor exibirá um ícone, se houver alertas atuais, e o painel Alertas do Computador listará os alertas. 

![Alertas do computador no painel Propriedades](./media/monitoring-vminsights-maps/properties-pane-alerts-01.png)

Para habilitar o recurso Mapa a exibir alertas relevantes, crie uma regra de alerta que é acionada para um computador específico. Para criar alertas apropriadas:
- Inclua uma cláusula para agrupar por computador (por exemplo, **intervalo de 1 minuto por Computador**).
- Escolha o alerta com base na medida métrica.

Para obter mais informações sobre os Alertas do Azure e como criar regras de alerta, confira [Alertas unificados no Azure Monitor](../monitoring-and-diagnostics/monitoring-overview-unified-alerts.md)

A opção **Legenda** no canto superior direito descreve as funções e os símbolos em um mapa.  Para ampliar para examinar mais detalhadamente o mapa e movê-lo, os controles de Zoom no lado inferior direito da página definem o nível de zoom e ajustam a página ao tamanho da página atual.  

## <a name="connection-metrics"></a>Métricas de conexão
O painel **Conexões** exibe as métricas de conectividade padrão para a conexão selecionada da VM pela porta TCP. As métricas incluem tempo de resposta, solicitações por minuto, taxa de transferência de tráfego e links.  

![Exemplo de painel de gráficos de conectividade de rede](./media/monitoring-vminsights-maps/map-group-network-conn-pane-01.png)  

### <a name="failed-connections"></a>Conexões com falha
As conexões com falha são mostradas no mapa para processos e computadores, com uma linha vermelha tracejada indicando que um sistema cliente não consegue acessar um processo ou uma porta. As conexões com falha serão relatadas por meio de qualquer sistema com o Dependency Agent se esse sistema estiver tentando estabelecer a conexão com falha. O Mapa mede esse processo observando os soquetes TCP que falham ao estabelecer uma conexão. Isso pode ocorrer devido a um firewall, uma configuração incorreta no cliente ou servidor ou um serviço remoto não disponível.

![Exemplo de conexão com falha no mapa](./media/monitoring-vminsights-maps/map-group-failed-connection-01.png)

O entendimento das conexões com falha pode ajudar com solução de problemas, validação da migração, análise de segurança e noções básicas sobre a arquitetura geral do serviço. Às vezes, as conexões com falhas são inofensivas, mas frequentemente apontam diretamente para um problema, como um ambiente de failover tornando-se inacessível repentinamente ou duas camadas de aplicativos sem comunicação após uma migração na nuvem.

### <a name="client-groups"></a>Grupos de Clientes
Os Grupos de Clientes no mapa representam os computadores cliente que têm conexões no computador mapeado. Um Grupo de Clientes único representa os clientes para um computador ou processo individual.

![Exemplo de Grupos de Clientes no mapa](./media/monitoring-vminsights-maps/map-group-client-groups-01.png)

Para ver os clientes monitorados e os endereços IP dos sistemas em um Grupo de Clientes, selecione o grupo. O conteúdo do grupo é listado abaixo do grupo.  

![Exemplo de lista de endereços IP do grupo de clientes no mapa](./media/monitoring-vminsights-maps/map-group-client-group-iplist-01.png)

Se o grupo incluir clientes monitorados e não monitorados, selecione a seção apropriada do gráfico de rosca no grupo para filtrar os clientes.

### <a name="server-port-groups"></a>Grupos de Portas do Servidor
Os Grupos de Porta do Servidor representam portas de servidor em servidores que têm conexões de entrada no computador mapeado. O grupo contém a porta do servidor e uma contagem do número de servidores com conexões com essa porta. Selecione o grupo para ver os servidores individuais e as conexões listadas. 

![Exemplo de Grupo de Portas do Servidor no mapa](./media/monitoring-vminsights-maps/map-group-server-port-groups-01.png)  

Se o grupo incluir servidores monitorados e não monitorados, selecione a seção apropriada do gráfico de rosca no grupo para filtrar os servidores.

## <a name="view-map-directly-from-a-virtual-machine"></a>Exibir o Mapa diretamente em uma máquina virtual 

Para acessar o Azure Monitor para VMs diretamente em uma máquina virtual, realize o procedimento a seguir.

1. No Portal do Azure, selecione **Máquinas Virtuais**. 
2. Na lista, escolha uma VM e, na seção **Monitoramento**, escolha **Insights (versão prévia)**.  
3. Selecione a guia **Mapa**.

O Mapa visualiza as dependências de VMs, ou seja, os grupos de processos e os processos em execução com conexões de rede ativas, em um intervalo de tempo especificado.  Por padrão, o mapa mostra os últimos 30 minutos.  Usando o seletor **TimeRange** no canto superior esquerdo, você pode consultar intervalos de tempo históricos de até uma hora para mostrar como as dependências eram no passado (por exemplo, durante um incidente ou antes de uma alteração).  

![Visão geral do mapa de VM direta](./media/monitoring-vminsights-maps/map-direct-vm-01.png)

## <a name="view-map-from-azure-monitor"></a>Exibir o Mapa no Azure Monitor
No Azure Monitor, o recurso Mapa fornece uma exibição global das máquinas virtuais e suas dependências.  Para acessar o recurso Mapa no Azure Monitor, realize o procedimento a seguir. 

1. No portal do Azure, selecione **Monitor**. 
2. Escolha **Máquinas Virtuais (versão prévia)** na seção **Insights**.
3. Selecione a guia **Mapa**.

![Visão geral do mapa de várias VMs do Azure Monitor](./media/monitoring-vminsights-maps/map-multivm-azure-monitor-01.png)

No seletor **Workspace** na parte superior da página, se você tiver mais de um workspace do Log Analytics, escolha aquele que está integrado à solução e que tem máquinas virtuais subordinadas a ele.  Em seguida, no seletor **Grupo**, selecione uma assinatura ou um grupo de recursos para exibir um conjunto de VMs e suas dependências que correspondem ao grupo, em um período especificado.  Por padrão, o mapa mostra os últimos 30 minutos.  Usando o seletor **TimeRange**, você pode consultar intervalos de tempo históricos de até uma hora para mostrar como as dependências eram no passado (por exemplo, durante um incidente ou antes de uma alteração).   

## <a name="next-steps"></a>Próximas etapas
Para saber como usar o recurso de integridade, confira [Exibir a Integridade da VM do Azure](monitoring-vminsights-health.md), ou para identificar gargalos e a utilização geral com o desempenho das VMs, confira [Exibir o Azure Monitor para o desempenho das VMs](monitoring-vminsights-performance.md). 
