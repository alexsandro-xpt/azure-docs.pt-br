---
title: Alta disponibilidade e recuperação de desastre do SAP HANA no Azure (instâncias grandes) | Microsoft Docs
description: Estabeleça a alta disponibilidade e planeje a recuperação de desastre do SAP HANA no Azure (instâncias grandes)
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/27/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 554991c7c0f11a095a11ae24dbb693a1a3ba50fd
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/02/2018
ms.locfileid: "39430113"
---
# <a name="sap-hana-large-instances-high-availability-and-disaster-recovery-on-azure"></a>Alta disponibilidade e recuperação de desastre de instâncias grandes do SAP HANA no Azure 

>[!IMPORTANT]
>Esta documentação não substitui a documentação de administração do SAP HANA ou as Notas do SAP. Espera-se que o leitor tenha um sólido reconhecimento e experiência em administração e operações do SAP HANA, especialmente com os tópicos de backup, restauração, alta disponibilidade e recuperação de desastre. Nesta documentação, são mostradas capturas de tela do SAP HANA Studio. O conteúdo, a estrutura e a natureza das telas das ferramentas de administração do SAP e as próprias ferramentas podem mudar de uma versão do SAP HANA para outra versão.

É importante que você execute as etapas e os processos realizados em seu ambiente e com as liberações e versões do HANA. Alguns processos descritos nesta documentação foram simplificados para um melhor entendimento geral e não devem ser usados como etapas detalhadas para eventuais manuais de operação. Se quiser criar manuais de operação para suas configurações, será necessário testar e executar os processos e documentar os processos relacionados às suas configurações específicas. 


Alta disponibilidade e recuperação de desastre (DR) são aspectos críticos da execução do SAP HANA no servidor do Azure (Instâncias Grandes). É importante trabalhar com o SAP, seu integrador de sistemas ou com a Microsoft para arquitetar e implementar adequadamente as estratégias corretas de alta disponibilidade e recuperação de desastre. Também é importante considerar o RPO (objetivo do ponto de recuperação) e o objetivo do tempo de recuperação, específicos para o ambiente.

A Microsoft dá suporte a alguns recursos de alta disponibilidade do SAP HANA com instâncias grandes do HANA. Esses recursos incluem:

- **Replicação de armazenamento**: a capacidade do sistema de armazenamento de replicar todos os dados para outro selo de instância grande do HANA em outra região do Azure. O SAP HANA funciona independentemente desse método. Essa funcionalidade é o mecanismo de recuperação de desastre padrão oferecido para o SAP HANA em Instâncias Grandes.
- **Replicação do sistema HANA**: a [replicação de todos os dados no SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html) para um sistema SAP HANA separado. O objetivo de tempo de recuperação é minimizado por meio da replicação de dados em intervalos regulares. O SAP HANA dá suporte a modos síncronos, síncronos na memória e assíncronos. O modo síncrono é utilizado apenas para sistemas SAP HANA que estão dentro do mesmo datacenter ou a menos de 100 km de distância. Com o design atual dos carimbos do SAP HANA em Instâncias Grandes, a replicação do sistema HANA pode ser utilizada para alta disponibilidade apenas dentro de uma região. A replicação do sistema HANA requer um componente roteamento ou de proxy reverso de terceiros para usar as configurações de recuperação de desastre em outra região do Azure. 
- **Failover automático de host**: uma solução local de recuperação de falhas para o SAP HANA que é uma alternativa à replicação do sistema HANA. Se o nó mestre ficar não disponível, configure um ou mais nós SAP HANA em espera no modo de escala horizontal e o SAP HANA fará automaticamente o failover para um nó em espera.

O SAP HANA (Instâncias Grandes) do Azure é oferecido em duas regiões do Azure, em quatro áreas geopolíticas (EUA, Austrália, Europa e Japão). Duas regiões dentro de uma área geopolítica que hospedam os carimbos do SAP HANA em Instâncias Grandes são conectadas a circuitos de rede dedicados separados. Eles são utilizados para replicar instantâneos de armazenamento para fornecer métodos de recuperação de desastre. A replicação não é estabelecida por padrão, mas é configurada para clientes que solicitam a funcionalidade de recuperação de desastre. A replicação de armazenamento depende do uso de instantâneos de armazenamento para instâncias grandes HANA. Não será possível escolher uma região do Azure como uma região de DR se estiver em uma área geopolítica diferente. 

A tabela a seguir mostra os métodos e combinações de alta disponibilidade e recuperação de desastre atualmente com suporte:

| Cenário com suporte em Instâncias Grandes HANA | Opção de alta disponibilidade | Opção de recuperação de desastre | Comentários |
| --- | --- | --- | --- |
| Nó único | Não disponível. | Instalação da recuperação de desastre dedicada.<br /> Instalação da recuperação de desastre multipropósito. | |
| Failover automático de host: expansão (com ou sem estado de espera)<br /> incluindo 1 + 1 | Possível com em espera usando a função ativa.<br /> O HANA controla a opção de função. | Instalação da recuperação de desastre dedicada.<br /> Instalação da recuperação de desastre multipropósito.<br /> Sincronização da recuperação de desastre usando a replicação de armazenamento. | Conjuntos de volumes do HANA são conectados a todos os nós.<br /> O local de recuperação de desastre deve ter o mesmo número de nós. |
| Replicação do sistema HANA | Possível com a instalação primária ou secundária.<br /> A secundária move para a função primária em um caso de failover.<br /> Replicação do sistema HANA e controle de failover do SO. | Instalação da recuperação de desastre dedicada.<br /> Instalação da recuperação de desastre multipropósito.<br /> Sincronização da recuperação de desastre usando a replicação de armazenamento.<br /> A recuperação de desastre usando replicação do sistema HANA ainda não é possível sem componentes de terceiros. | Conjunto separado de volumes de disco são anexados a cada nó.<br /> Somente os volumes de disco de réplica secundária no local de produção são replicados para o local da recuperação de desastre.<br /> Um conjunto de volumes é necessário no local da recuperação de desastre. | 

É em uma configuração de recuperação de desastre dedicada que a unidade da Instância Grande do HANA no local da recuperação de desastre não é usada para executar nenhuma outra carga de trabalho ou sistema de não produção. A unidade é passiva e é implantada apenas se um failover de desastre é executado. No entanto, essa configuração não é uma opção preferencial para muitos clientes.

Consulte os [ cenários suportados pelo HLI ](hana-supported-scenario.md) para aprender o layout de armazenamento e os detalhes de ethernet para sua arquitetura.

> [!NOTE]
> [Implantações do SAP HANA MCOD](https://launchpad.support.sap.com/#/notes/1681092) (várias instâncias do HANA em uma unidade) como cenários de sobreposição funcionam com os métodos de alta disponibilidade e recuperação de desastres listados na tabela. Uma exceção é o uso da Replicação do Sistema do HANA com um cluster de failover automático baseado no Pacemaker. Esse caso permite somente uma instância do HANA por unidade. Para implantações de [SAP HANA MDC](https://launchpad.support.sap.com/#/notes/2096000), somente métodos de HA e DR não baseados em armazenamento funcionarão se mais de um locatário for implantado. Com um locatário implantado, todos os métodos listados são válidos.  

É em uma configuração de recuperação de desastre multipropósito que a unidade de Instância Grande do HANA no local da recuperação de desastre executa uma carga de trabalho de não produção. Em caso de desastre, desligue o sistema de não produção, monte os conjuntos de volume de armazenamento replicado (adicional) e, em seguida, inicie a instância do HANA de produção. A maioria dos clientes que usam a funcionalidade de recuperação de desastre do SAP HANA em Instâncias Grandes utiliza essa configuração. 


É possível encontrar mais informações sobre a alta disponibilidade do SAP HANA nos seguintes artigos do SAP: 

- [Whitepaper de Alta Disponibilidade do SAP HANA](http://go.sap.com/documents/2016/05/f8e5eeba-737c-0010-82c7-eda71af511fa.html)
- [Guia de administração do SAP HANA](http://help.sap.com/hana/SAP_HANA_Administration_Guide_en.pdf)
- [Vídeo do SAP HANA Academy sobre Replicação do Sistema SAP HANA](http://scn.sap.com/community/hana-in-memory/blog/2015/05/19/sap-hana-system-replication)
- [SAP Note de Suporte nº 1999880 – perguntas frequentes sobre replicação do sistema SAP HANA](https://apps.support.sap.com/sap/support/knowledge/preview/en/1999880)
- [SAP Note de Suporte SAP nº 2165547 – Fazer backup do SAP HANA e restaurá-lo no ambiente de replicação do sistema SAP HANA](https://websmp230.sap-ag.de/sap(bD1lbiZjPTAwMQ==)/bc/bsp/sno/ui_entry/entry.htm?param=69765F6D6F64653D3030312669765F7361706E6F7465735F6E756D6265723D3231363535343726)
- [SAP Note de Suporte SAP nº 1984882 – uso da replicação do sistema SAP HANA para troca de hardware com tempo de inatividade mínimo/zero](https://websmp230.sap-ag.de/sap(bD1lbiZjPTAwMQ==)/bc/bsp/sno/ui_entry/entry.htm?param=69765F6D6F64653D3030312669765F7361706E6F7465735F6E756D6265723D3139383438383226)

## <a name="network-considerations-for-disaster-recovery-with-hana-large-instances"></a>Considerações de rede para recuperação de desastre com Instâncias Grandes HANA

Para aproveitar a funcionalidade da recuperação de desastre do SAP HANA em Instâncias Grandes, é necessário começar a projetar a conectividade de rede para as duas regiões do Azure. É necessária uma conexão de circuito do Azure ExpressRoute dos locais em sua região principal do Azure e outra conexão de circuito com os locais em sua região de recuperação de desastre. Essa medida abrange uma situação na qual há um problema em uma região do Azure, incluindo um local do MSEE (Microsoft Enterprise Edge Router).

Como segunda medida, é possível conectar todas as redes virtuais do Azure que conectam-se ao SAP HANA (Instâncias Grandes) do Azure em uma região a um circuito de ExpressRoute que conecta o SAP HANA em Instâncias Grandes na outra região. Com essa *conexão cruzada* os serviços em execução em uma rede virtual do Azure na Região 1 podem conectar-se a unidades do SAP HANA em Instâncias Grandes na Região 2 e vice-versa. Essa medida resolve um caso em que apenas um dos locais do MSEE conecta-se a um ponto local com o Azure estando offline.

O gráfico a seguir ilustra uma configuração flexível para casos de recuperação de desastre:

![Configuração ideal para a recuperação de desastre](./media/hana-overview-high-availability-disaster-recovery/image1-optimal-configuration.png)



## <a name="other-requirements-with-hana-large-instances-storage-replication-for-disaster-recovery"></a>Outros requisitos com replicação de armazenamento do SAP HANA em Instâncias Grandes para recuperação de desastre

Além dos requisitos anteriores para uma configuração de recuperação de desastre com o SAP HANA em Instâncias Grandes, é necessário:

- Ordenar SKUs do SAP HANA (Instâncias Grandes) do Azure do mesmo tamanho que SKUs de produção e implantá-las na região de recuperação de desastre. Nas implantações atuais do cliente, essas instâncias são usadas para executar instâncias HANA de não produção. Essas configurações são chamadas de *configurações de recuperação de desastres multipropósito*.   
- Ordenar armazenamento adicional no site de DR para cada uma das SKUs do SAP HANA (Instâncias Grandes) do Azure que você deseja recuperar no site de recuperação de desastre. Comprar armazenamento adicional permite que você aloque os volumes de armazenamento. É possível alocar os volumes que são o destino da replicação de armazenamento da sua região do Azure de produção para a região de recuperação de desastre do Azure.
- No caso de ter configurado HSR primário e configurar a replicação baseada em armazenamento para o site de DR, você deverá adquirir armazenamento adicional no site de DR para que os dados de nós primários e secundários sejam replicados para o site de DR.

 

## <a name="backup-and-restore"></a>Backup e restauração

Um dos aspectos mais importantes para a operação de bancos de dados é protegê-los contra eventos catastróficos. A causa desses eventos pode ser qualquer coisa, desde desastres naturais a simples erros de usuário.

O backup de um banco de dados, com a capacidade de restaurá-lo para qualquer ponto no tempo (como antes de alguém excluir dados críticos), permite a restauração para um estado o mais próximo possível à forma como estava antes da interrupção.

Dois tipos de backup devem ser executados para obter melhores resultados:

- Backups de banco de dados: backups completos, incrementais ou diferenciais
- Backups do log de transações

Além de backups completos do banco de dados realizados em um nível de aplicativo, é possível realizar backups com instantâneos de armazenamento. Os instantâneos de armazenamento não substituem backups de log de transações. Os backups de log de transações continuam sendo importantes para restaurar o banco de dados para um determinado ponto no tempo ou para esvaziar os logs de transações já confirmadas. No entanto, instantâneos de armazenamento podem acelerar a recuperação fornecendo rapidamente uma imagem de roll forward do banco de dados. 

O SAP HANA no Azure (Instâncias Grandes) oferece duas opções de backup e de restauração:

- DIY (Faça Você Mesmo). Após calcular para garantir que haja espaço em disco suficiente, execute backups completos de banco de dados e de log usando um dos métodos de backup em disco a seguir. É possível fazer backup diretamente para volumes anexados às unidades do SAP HANA em Instâncias Grandes ou para NFS (Compartilhamentos de Arquivos de Rede) configurados em uma VM (máquina virtual) do Azure. No último caso, os clientes configuram uma VM do Linux no Azure, anexam o Armazenamento do Azure à VM e compartilham o armazenamento por meio de um servidor NFS configurado na VM. Se você realizar o backup em volumes anexados diretamente às unidades de Instância Grande do HANA, será necessário copiar os backups para uma conta de armazenamento do Azure (depois de configurar uma VM do Azure que exporta compartilhamentos NFS com base no Armazenamento do Microsoft Azure). Também é possível usar um cofre de backup do Azure ou um armazenamento frio do Azure. 

   Outra opção é usar uma ferramenta de proteção de dados de terceiros para armazenar os backups depois que eles são copiados em uma conta de armazenamento do Azure. A opção de backup DIY também pode ser necessária para dados que precisam ser armazenados por períodos mais longos devido à conformidade e para fins de auditoria. Em todos os casos, os backups são copiados para compartilhamentos NFS representados por meio de uma VM e do Armazenamento do Azure.

- Funcionalidade de backup e restauração de infraestrutura. Também é possível utilizar a funcionalidade de backup e restauração que a infraestrutura subjacente do SAP HANA (Instâncias Grandes) do Azure fornece. Essa opção atende a necessidade de backups e restaurações rápidas. O restante desta seção aborda a funcionalidade de backup e de restauração oferecida com Instâncias Grandes HANA. Esta seção também aborda a restauração e o backup de relação e a restauração para a funcionalidade de recuperação de desastre oferecida pelo SAP HANA em Instâncias Grandes.

>   [!NOTE]
>   A tecnologia de instantâneo que é usada pela infraestrutura subjacente das Instâncias Grandes HANA tem uma dependência de instantâneos do SAP HANA. Neste ponto, os instantâneos do SAP HANA não funcionam junto com os múltiplos locatários dos contêineres de banco de dados multilocatário do SAP HANA. Se apenas um locatário for implantado, os instantâneos do SAP HANA funcionarão e esse método poderá ser utilizado.

### <a name="using-storage-snapshots-of-sap-hana-on-azure-large-instances"></a>Usando instantâneos de armazenamento do SAP HANA no Azure (Instâncias Grandes)

A infraestrutura de armazenamento subjacente ao SAP HANA no Azure (Instâncias Grandes) dá suporte a instantâneos de armazenamento de volumes. Há suporte para o backup e para a restauração de volumes, com as seguintes considerações:

- Em vez de backups de banco de dados completos, instantâneos de volume de armazenamento são executados com frequência.
- Ao disparar um instantâneo sobre os volumes /hana/data e /hana/shared (inclui /usr/sap), a tecnologia de instantâneos iniciará um instantâneo do SAP HANA antes de executar o instantâneo de armazenamento. Esse instantâneo SAP HANA é o ponto de instalação para restaurações de log eventual após a recuperação do instantâneo de armazenamento. Para que o instantâneo do HANA seja bem-sucedido, é preciso uma instância ativa do HANA.  No cenário de HSR, não há suporte para o instantâneo de armazenamento no nó secundário atual em que o instantâneo do HANA não pode ser executado.
- Após o instantâneo de armazenamento ser executado com êxito, o instantâneo do SAP HANA será excluído.
- Os backups de log de transações são usados frequentemente e armazenados no volume /hana/logbackups ou no Azure. É possível disparar o volume /hana/logbackups que contém os backups de log de transações para usar um instantâneo separadamente. Nesse caso, não é necessário executar um instantâneo do HANA.
- Se você precisar restaurar um banco de dados para um determinado ponto no tempo, solicite suporte do Microsoft Azure (para uma interrupção de produção) ou do Gerenciamento de Serviços do SAP HANA do Azure para restaurar para um determinado instantâneo de armazenamento. Um exemplo é uma restauração planejada de um sistema de área restrita para seu estado original.
- O instantâneo SAP HANA incluído no instantâneo de armazenamento é um ponto de deslocamento para a aplicação de backups de log de transações que foram executados e armazenados após o instantâneo de armazenamento ter sido tirado.
- Esses backups de log de transações são aplicados para restaurar o banco de dados para um determinado ponto no tempo.

É possível executar instantâneos de armazenamento, direcionando três classes de volumes:

- Um instantâneo combinado em /hana/data e /hana/shared (inclui /usr/sap). Esse instantâneo exige a criação de um instantâneo do SAP HANA como preparação para o instantâneo de armazenamento. O instantâneo do SAP HANA garante que o banco de dados esteja em um estado consistente do ponto de vista de armazenamento. Ele garante também que haja um ponto de configuração para o processo de restauração.
- Um instantâneo separado em /hana/logbackups.
- Uma partição do sistema operacional.

Obtenha os scripts de instantâneo e a documentação mais recentes do [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts). 

### <a name="storage-snapshot-considerations"></a>Considerações sobre o instantâneo de armazenamento

>[!NOTE]
>Instantâneos de armazenamento consomem o espaço de armazenamento que foi alocado para as unidades de Instância Grande do HANA. É necessário considerar os seguintes aspectos de programação de instantâneos de armazenamento e quantos instantâneos de armazenamento devem ser mantidos. 

A mecânica específica de instantâneos de armazenamento para SAP HANA no Azure (Instâncias Grandes) inclui:

- Um instantâneo de armazenamento específico (no ponto no tempo em que é obtido) consome pouco de armazenamento.
- Como as alterações de conteúdo de dados e o conteúdo nos arquivos de dados do SAP HANA mudam no volume de armazenamento, o instantâneo precisa armazenar o conteúdo do bloco original, assim como as alterações de dados.
- Como resultado, o instantâneo de armazenamento aumenta de tamanho. Quanto mais tempo o instantâneo existe, quanto maior o instantâneo de armazenamento ficar.
- Quanto mais alterações são feitas no volume de banco de dados do SAP HANA durante o tempo de vida de um instantâneo de armazenamento, maior o consumo de espaço do instantâneo de armazenamento.

O SAP HANA no Azure (Instâncias Grandes) vem com tamanhos de volume fixo para o volume de dados e de log do SAP HANA. A execução de instantâneos desses volumes consome seu espaço de volume. É necessário determinar quando agendar instantâneos de armazenamento. Também é necessário monitorar o consumo de espaço dos volumes de armazenamento, assim como gerenciar o número de instantâneos que você armazena. É possível desabilitar os instantâneos de armazenamento quando você importa muitos dados ou realiza outras alterações significativas no banco de dados do HANA. 


As seções a seguir fornecem informações para executar esses instantâneos incluindo recomendações gerais:

- Embora o hardware possa suportar 255 instantâneos por volume, é recomendável que permaneça bem abaixo desse número. A recomendação é 250 ou menos.
- Antes de executar instantâneos de armazenamento, monitore e mantenha o controle do espaço livre.
- Reduza o número de instantâneos de armazenamento com base em espaço livre. É possível reduzir o número de instantâneos que você mantém ou aumentar os volumes. É possível solicitar armazenamento adicional em unidades de 1 terabyte.
- Durante atividades como a movimentação de dados em SAP HANA com ferramentas de migração da plataforma SAP (R3load) ou a restauração de bancos de dados do SAP HANA com base em backups, desabilite instantâneos de armazenamento no volume /hana/data. 
- Durante maiores reorganizações de tabelas do SAP HANA, deve-se evitar os instantâneos de armazenamento, se possível.
- Os instantâneos de armazenamento são um pré-requisito para usar os recursos de recuperação de desastre do SAP HANA (Instâncias Grandes) do Azure.

### <a name="prerequisites-for-using-self-service-storage-snapshots"></a>Pré-requisitos para usar os instantâneos de armazenamento de autoatendimento

Para garantir que o script do instantâneo execute com êxito, verifique se o Perl está instalado no sistema operacional Linux no servidor do SAP HANA em Instâncias Grandes. O Perl vem pré-instalado em sua unidade do HANA em Instâncias Grandes. Para verificar a versão do Perl, use a linha de comando a seguir:

`perl -v`

![A chave pública é copiada executando este comando](./media/hana-overview-high-availability-disaster-recovery/perl_screen.png)


### <a name="set-up-storage-snapshots"></a>Configurar instantâneos de armazenamento

Para configurar instantâneos de armazenamento com SAP HANA em Instâncias Grandes, siga estas etapas:
1. Verifique se o Perl está instalado no sistema operacional Linux no servidor das Instâncias Grandes HANA.
1. Modifique /etc/ssh/ssh\_config para adicionar a linha _MACs hmac-sha1_.
1. Crie uma conta de usuário de backup do SAP HANA no nó mestre para cada instância do SAP HANA que está em execução, se aplicável.
1. Instale o cliente SAP HANA HDB em todos os servidores de Instâncias Grandes do SAP HANA.
1. No primeiro servidor de Instâncias Grandes do SAP HANA de cada região, crie uma chave pública para acessar a infraestrutura de armazenamento subjacente que controla a criação de instantâneos.
1. Copie os scripts e o arquivo de configuração do [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts) para o local do **hdbsql** na instalação do SAP HANA.
1. Modifique o arquivo *HANABackupDetails.txt* conforme necessário para as especificações apropriadas do cliente.

Obtenha os scripts de instantâneo e a documentação mais recentes do [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts). 

### <a name="consideration-for-mcod-scenarios"></a>Consideração para cenários de MCOD
Se estiver executando um [cenário de MCOD](https://launchpad.support.sap.com/#/notes/1681092) com várias instâncias do SAP HANA em uma unidade do SAP HANA em Instâncias Grandes, você terá volumes de armazenamento separados provisionados para cada uma das instâncias do SAP HANA. Na versão atual da automação de instantâneo de autoatendimento, não é possível iniciar instantâneos separados em cada SID (ID do sistema) da instância do HANA. A funcionalidade fornece verificações das instâncias do SAP HANA registradas do servidor no arquivo de configuração (veja mais adiante neste artigo) e executa um instantâneo simultâneo dos volumes de todas as instâncias registradas na unidade.
 

### <a name="step-1-install-the-sap-hana-hdb-client"></a>Etapa 1: Instalar o cliente do SAP HANA HDB

O sistema operacional Linux instalado no SAP HANA (Instâncias Grandes) do Azure inclui as pastas e os scripts necessários para executar instantâneos de armazenamento SAP HANA para fins de backup e de recuperação de desastre. Procure versões mais recentes em [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts). A versão mais recente dos scripts é a 3.x. Scripts diferentes podem ter diferentes versões secundárias dentro da mesma versão principal.

>[!IMPORTANT]
>Ao passar da versão 2.1 para a versão 3.x dos scripts, observe que a estrutura do arquivo de configuração e alguma sintaxe foi alterada. Consulte os textos explicativos nas seções específicas. 

No entanto, é sua responsabilidade instalar o cliente do SAP HANA HDB nas unidades de SAP HANA em Instâncias Grandes durante a instalação do SAP HANA.

### <a name="step-2-change-the-etcsshsshconfig"></a>Etapa 2: Alterar o /etc/ssh/ssh\_config

Altere `/etc/ssh/ssh_config` adicionando a linha _MACs hmac-sha1_ conforme mostrado aqui:
```
#   RhostsRSAAuthentication no
#   RSAAuthentication yes
#   PasswordAuthentication yes
#   HostbasedAuthentication no
#   GSSAPIAuthentication no
#   GSSAPIDelegateCredentials no
#   GSSAPIKeyExchange no
#   GSSAPITrustDNS no
#   BatchMode no
#   CheckHostIP yes
#   AddressFamily any
#   ConnectTimeout 0
#   StrictHostKeyChecking ask
#   IdentityFile ~/.ssh/identity
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   Port 22
Protocol 2
#   Cipher 3des
#   Ciphers aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-cbc,3des-cbc
#   MACs hmac-md5,hmac-sha1,umac-64@openssh.com,hmac-ripemd160
MACs hmac-sha1
#   EscapeChar ~
#   Tunnel no
#   TunnelDevice any:any
#   PermitLocalCommand no
#   VisualHostKey no
#   ProxyCommand ssh -q -W %h:%p gateway.example.com
```

### <a name="step-3-create-a-public-key"></a>Etapa 3: Criar uma chave pública

Para habilitar o acesso às interfaces de instantâneo de armazenamento do locatário do SAP HANA em Instâncias Grandes, é necessário estabelecer um procedimento de conexão por meio de uma chave pública. No primeiro servidor do SAP HANA (Instâncias Grandes) do Azure no locatário, crie uma chave pública a ser usada para acessar a infraestrutura de armazenamento. A chave pública garante que uma senha não será necessária para entrar nas interfaces do instantâneo de armazenamento. A criação de uma chave pública também significa que não é necessário manter credenciais de senha. No Linux no servidor de Instâncias Grandes do SAP HANA, execute o seguinte comando para gerar a chave pública:
```
  ssh-keygen –t dsa –b 1024
```
O novo local é **_/root/.ssh/id\_dsa.pub**. Não insira uma senha real; caso contrário, será necessário inserir a senha todas as vezes que entrar. Em vez disso, selecione **Enter** duas vezes para remover o requisito “inserir senha” para entrar.

Certifique-se de que a chave pública foi corrigida conforme o esperado alterando as pastas para **/root/.ssh/** e executando o comando `ls`. Se a chave estiver presente, você poderá copiá-la executando o seguinte comando:

![A chave pública é copiada executando este comando](./media/hana-overview-high-availability-disaster-recovery/image2-public-key.png)

Neste ponto, entre em contato com o Gerenciamento de Serviços do SAP HANA no Azure e informe a chave pública. O representante de serviço usa uma chave pública para registrá-la na infraestrutura de armazenamento subjacente que está gravada para seu locatário de Instância Grande HANA.

### <a name="step-4-create-an-sap-hana-user-account"></a>Etapa 4: Criar uma conta de usuário do SAP HANA

Para iniciar a criação de instantâneos do SAP HANA, é necessário criar uma conta de usuário no SAP HANA que os scripts de instantâneo de armazenamento possam usar. Crie uma conta de usuário do SAP HANA no SAP HANA Studio para essa finalidade. O usuário deve ser criado no SYSTEMDB, e NÃO no banco de dados do SID para MDC. No ambiente de contêiner único, o usuário é configurado no banco de dados de locatário. Essa conta deve ter os seguintes privilégios: **Administrador de Backup** e **Leitura de Catálogo**. Neste exemplo, o nome de usuário é **SCADMIN**. O nome da conta de usuário criada no HANA Studio diferencia maiúsculas de minúsculas. Certifique-se de selecionar **Não** para exigir que o usuário altere a senha na próxima conexão.

![Criando um usuário no Studio HANA](./media/hana-overview-high-availability-disaster-recovery/image3-creating-user.png)

Se você utilizar implantações de MCOD com várias instâncias do SAP HANA em uma unidade, será necessário repetir essa etapa para cada instância do SAP HANA.

### <a name="step-5-authorize-the-sap-hana-user-account"></a>Etapa 5: Autorizar a conta de usuário do SAP HANA

Nesta etapa, autorize a conta de usuário do SAP HANA que você criou para que os scripts não precisem enviar senhas em tempo de execução. O comando `hdbuserstore` do SAP HANA permite criar uma chave de usuário do SAP HANA, armazenada em um ou mais nós do SAP HANA. A chave de usuário permite que ele acesse o SAP HANA sem precisar gerenciar senhas de dentro do processo de script. O processo de script será abordado posteriormente neste artigo.

>[!IMPORTANT]
>Execute o seguinte comando no usuário sob o qual os scripts foram planejados para ser executados. Caso contrário, o script não funcionará corretamente.

Insira o comando `hdbuserstore` da seguinte maneira:

**Para a instalação do HANA não MDC**
```
hdbuserstore set <key> <host>:<3[instance]15> <user> <password>
```

**Para a instalação do HANA MDC**
```
hdbuserstore set <key> <host>:<3[instance]13> <user> <password>
```

No exemplo a seguir, o usuário é **SCADMIN01**, o nome do host é **lhanad01** e o número da instância é **01**:
```
hdbuserstore set SCADMIN01 lhanad01:30115 <backup username> <password>
```
Se utilizar uma implantação do HANA MCOD com várias instâncias do SAP HANA em uma unidade, será necessário repetir a etapa para cada instância do SAP HANA e o usuário de backup associado na unidade.

Se tiver uma configuração de expansão do SAP HANA, será necessário gerenciar todos os scripts de um único servidor. Neste exemplo, a chave **SCADMIN01** do SAP HANA deve ser alterada para cada host de maneira que mostre qual host está relacionado à chave. Corrija a conta de backup do SAP HANA com o número da instância do HANA DB. A chave deve ter privilégios administrativos no host ao qual está atribuída e o usuário de backup para configurações de expansão deve ter direitos de acesso a todas as instâncias do SAP HANA. Supondo que três nós de expansão tenham os nomes **lhanad01**, **lhanad02** e **lhanad03**, a sequência de comandos tem essa aparência:

```
hdbuserstore set SCADMIN01 lhanad01:30115 SCADMIN <password>
hdbuserstore set SCADMIN01 lhanad02:30115 SCADMIN <password>
hdbuserstore set SCADMIN01 lhanad03:30115 SCADMIN <password>
```

### <a name="step-6-get-the-snapshot-scripts-configure-the-snapshots-and-test-the-configuration-and-connectivity"></a>Etapa 6: Obter os scripts de instantâneo, configurar os instantâneos e testar a configuração e a conectividade

Baixe a versão mais recente dos scripts no [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts). Copie os scripts baixados e o arquivo de texto no diretório de trabalho para **hdbsql**. Para instalações atuais do HANA, esse diretório está no seguinte formato: /hana/shared/D01/exe/linuxx86\_64/hdb. 
``` 
azure_hana_backup.pl 
azure_hana_replication_status.pl 
azure_hana_snapshot_details.pl 
azure_hana_snapshot_delete.pl 
testHANAConnection.pl 
testStorageSnapshotConnection.pl 
removeTestStorageSnapshot.pl
azure_hana_dr_failover.pl
azure_hana_test_dr_failover.pl 
HANABackupCustomerDetails.txt 
``` 

Ao lidar com scripts Perl: 

- Nunca modifique os scripts, a menos que seja instruído pelo Microsoft Operations.
- Quando for solicitado a modificar o script ou um arquivo de parâmetro, sempre use o editor de texto do Linux, como "vi", e não os editores do Windows, como o Bloco de Notas. Usar o editor do Windows pode corromper o formato de arquivo.
- Sempre use os scripts mais recentes. Você pode baixar a versão mais recente no GitHub.
- Use a mesma versão de script em toda a paisagem.
- Teste os scripts e fique à vontade com os parâmetros necessários e a saída do script antes de usá-los diretamente no sistema de produção.
- Não altere o nome do ponto de montagem do servidor provisionado pelo Microsoft Operations. Esses scripts contam com a disponibilidade desses pontos de montagem padrão para uma execução bem-sucedida.


A finalidade dos diferentes scripts e arquivos é conforme a seguir:

- **azure\_hana\_backup.pl**: esse script é agendado com o utilitário Linux Cron Scheduling para executar instantâneos de armazenamento nos dados e volumes compartilhados do HANA, no volume /hana/logbackups ou no sistema operacional.
- **azure\_hana\_replication\_status.pl**: esse script fornece os detalhes básicos sobre o status de replicação do site de produção para o site de recuperação de desastre. O script monitora para garantir que a replicação está ocorrendo e mostra o tamanho dos itens que estão sendo replicados. Ele também fornecerá orientação se uma replicação estiver demorando muito ou se o link estiver inativo.
- **azure\_hana\_snapshot\_details.pl**: este script oferece uma lista de detalhes básicos sobre todos os instantâneos, por volume, que existem em seu ambiente. Esse script pode ser executado no servidor primário ou em uma unidade de servidor no local da recuperação de desastre. O script fornece as informações a seguir, divididas por cada volume que contém os instantâneos:
   * O tamanho do total de instantâneos em um volume
   * Os detalhes a seguir em cada instantâneo nesse volume: 
      - Nome do instantâneo 
      - Criar tempo 
      - Tamanho do instantâneo
      - Frequência do instantâneo
      - ID do HANA Backup associada a esse instantâneo, se relevante
- **azure\_hana\_snapshot\_delete.pl**: este script exclui um instantâneo de armazenamento ou um conjunto de instantâneos. É possível usar a ID de backup do SAP HANA como encontrada no HANA Studio, ou o nome do instantâneo de armazenamento. Atualmente, a ID de backup só está vinculada aos instantâneos criados para os volumes data/log/shared HANA. Caso contrário, se a ID de instantâneo for inserida, serão procurados todos os instantâneos que correspondam à ID de instantâneo inserida.  
- **testHANAConnection.pl**: este script testa a conexão com a instância do SAP HANA e é necessário para configurar o instantâneo de armazenamento.
- **testStorageSnapshotConnection.pl**: esse script tem dois propósitos. Primeiro, ele garante que a unidade do SAP HANA em Instâncias Grandes que executa os scripts tenha acesso à máquina virtual de armazenamento atribuído e com a interface de instantâneo de armazenamento do SAP HANA em Instâncias Grandes. A segunda finalidade é criar um instantâneo temporário para a instância do HANA que você está testando. Esse script deve ser executado para cada instância do HANA em um servidor para garantir que os scripts de backup funcionem conforme o esperado.
- **removeTestStorageSnapshot.pl**: esse script exclui o instantâneo de teste criado com o script **testStorageSnapshotConnection.pl**.
- **azure\_hana\_dr\_failover.pl**: esse script inicia um failover de recuperação de desastre em outra região. O script precisa ser executado na unidade do SAP HANA em Instâncias Grande na região de DR ou na unidade para a qual você deseja fazer failover. Esse script interrompe a replicação do armazenamento do lado primário para o lado secundário, restaura o instantâneo mais recente nos volumes de DR e fornece os pontos de montagem para os volumes de DR.
- **azure\_hana\_test\_dr\_failover.pl**: esse script executa um failover de teste no site de DR. Ao contrário do script azure_hana_dr_failover.pl, essa execução não interrompe a replicação do armazenamento do primário para o secundário. Em vez disso, clones dos volumes de armazenamento replicados são criados no lado de DR e os pontos de montagem dos volumes clonados são fornecidos. 
- **HANABackupCustomerDetails.txt**: este arquivo é um arquivo de configuração modificável que você precisa modificar para adaptar-se à sua configuração do SAP HANA. O arquivo *HANABackupCustomerDetails.txt* é o arquivo de controle e configuração do script que executa os instantâneos de armazenamento. Ajuste o arquivo para suas finalidades e configuração. Você receberá o **Nome do Backup de Armazenamento** e o **Endereço IP da Conta de Armazenamento**  do SAP HANA no Gerenciamento de Serviços do Azure quando as instâncias forem implantadas. Não é possível modificar a sequência, ordenação nem o espaçamento de qualquer uma das variáveis nesse arquivo. Se você fizer isso, os scripts não serão executados corretamente. Além disso, você recebe o endereço IP do nó de escala vertical ou do nó mestre (se expansão) do SAP HANA no Gerenciamento de Serviços do Azure. Você também conhece o número de instância do HANA obtido durante a instalação do SAP HANA. Agora, é necessário adicionar um nome de backup ao arquivo de configuração.

Para uma implantação escalável verticalmente ou horizontalmente, o arquivo de configuração seria semelhante ao exemplo a seguir, após o preenchimento do nome do servidor da unidade do SAP HANA em Instâncias Grandes e do endereço IP do servidor. Preencha todos os campos necessários para cada SID do SAP HANA de que você quer fazer backup ou recuperação.

Também é possível comentar linhas de instâncias que você não quer fazer backup por um período de tempo adicionando um "#" na frente de um campo obrigatório. Também não é necessário inserir todas as instâncias do SAP HANA contidas em um servidor, se não houver necessidade de fazer backup ou recuperar essa instância específica. O formato deve ser mantido para todos os campos, ou todos os scripts apresentarão uma mensagem de erro e o script será encerrado. É possível excluir as linhas necessárias adicionais de todos os detalhes de informações do SID que você não estiver utilizando após a última instância do SAP HANA em uso. Todas as linhas devem ser preenchidas, comentadas ou excluídas.

>[!IMPORTANT]
>A estrutura do arquivo foi alterada com a mudança da versão 2.1 para a versão 3.x. Se quiser usar os scripts da versão 3.x, você precisará adaptar a estrutura do arquivo de configuração. 


```
HANA Server Name: testing01
HANA Server IP Address: 172.18.18.50
```

Para cada instância que configurar na unidade do SAP HANA em Instâncias Grandes ou para a configuração de expansão, será necessário definir os dados conforme a seguir:

    
```
######***SID #1 Information***#####
SID1: h01
###Provided by Microsoft Operations###
SID1 Storage Backup Name: clt1h01backup
SID1 Storage IP Address: 172.18.18.11
######     Customer Provided    ######
SID1 HANA instance number: 00
SID1 HANA HDBuserstore Name: SCADMINH01
```
Para configurações de expansão e Replicação do Sistema do HANA, repita essa configuração em cada um dos nós. Essa medida garante que, em casos de falha, os backups e a eventual replicação do armazenamento possam continuar funcionando.   

Após colocar todos os dados de configuração no arquivo *HANABackupCustomerDetails.txt* verifique se as configurações estão corretas para os dados da instância do HANA. Use o script `testHANAConnection.pl`, que é independente de uma configuração de expansão ou escala vertical do SAP HANA.

```
testHANAConnection.pl
```

Se você tiver uma configuração de expansão do SAP HANA, certifique-se de que a instância mestre do HANA tenha acesso a todos os servidores e instâncias necessários do HANA. Não há parâmetros para o script de teste, porém é necessário adicionar seus dados no arquivo de configuração *HANABackupCustomerDetails.txt* para o script executar corretamente. Apenas os códigos de erro de comando shell são retornados, portanto, não é possível para o script verificar os erros de cada instância. Ainda assim, o script ainda oferece alguns comentários úteis para você verificar novamente.

Para executar o script, insira o seguinte comando:
```
 ./testHANAConnection.pl
```
O script exibirá a mensagem de que a conexão do HANA foi bem-sucedida se for bem-sucedido ao obter o status da instância do HANA.


A próxima etapa de teste é verificar a conectividade com o armazenamento com base nos dados que você inseriu no arquivo de configuração *HANABackupCustomerDetails.txt* e, em seguida, executar um instantâneo de teste. Antes de executar o script `azure_hana_backup.pl`, é necessário executar esse teste. Se um volume não contiver instantâneos, será impossível determinar se o volume está vazio ou se há uma falha de SSH para obter os detalhes do instantâneo. Por esse motivo, script é executado em duas etapas:

- Ele verifica se a máquina virtual de armazenamento e as interfaces dos locatários estão acessíveis para os scripts executarem instantâneos.
- Ele cria um instantâneo de teste ou fictício para cada volume por instância do HANA.

Por esse motivo, a instância do HANA é incluída como um argumento. Se a execução falhar, não será possível fornecer a verificação de erros para a conexão de armazenamento. Mesmo se não houver nenhuma verificação de erro, o script fornecerá dicas úteis.

1. Execute a sequência de comandos para executar este teste:

   ```
   ssh <StorageUserName>@<StorageIP>
   ```

   O nome de usuário do armazenamento e o endereço IP do armazenamento foram fornecidos para você na transferência da unidade do HANA em Instâncias Grandes.

1. Executar o script de teste:
   ```
    ./testStorageSnapshotConnection.pl
   ```

O script tenta entrar no armazenamento usando a chave pública fornecida nas etapas de configuração anteriores e com os dados configurados no arquivo *HANABackupCustomerDetails.txt*. Se a conexão for bem-sucedida, o seguinte conteúdo será mostrado:

```
**********************Checking access to Storage**********************
Storage Access successful!!!!!!!!!!!!!!
```

Se ocorrerem problemas na conexão com o console de armazenamento, a saída será semelhante a:

```
**********************Checking access to Storage**********************
WARNING: Storage check status command 'volume show -type RW -fields volume' failed: 65280
WARNING: Please check the following:
WARNING: Was publickey sent to Microsoft Service Team?
WARNING: If passphrase entered while using tool, publickey must be re-created and passphrase must be left blank for both entries
WARNING: Ensure correct IP address was entered in HANABackupCustomerDetails.txt
WARNING: Ensure correct Storage backup name was entered in HANABackupCustomerDetails.txt
WARNING: Ensure that no modification in format HANABackupCustomerDetails.txt like additional lines, line numbers or spacing
WARNING: ******************Exiting Script*******************************
```

Após uma conexão com êxito nas interfaces da máquina virtual de armazenamento, o script continuará com a fase 2 e criará um instantâneo de teste. A saída é mostrada aqui para uma configuração de expansão de três nós do SAP HANA:

```
**********************Creating Storage snapshot**********************
Taking snapshot testStorage.recent for hana_data_hm3_mnt00001_t020_dp ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00001_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00002_t020_dp ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00002_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00003_t020_dp ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00003_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_backups_hm3_t020_dp ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_backups_hm3_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_hm3_mnt00001_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_hm3_mnt00002_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_hm3_mnt00003_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_shared_hm3_t020_vol ...
Snapshot created successfully.
```

Se o instantâneo de teste foi executado com êxito com o script, você poderá continuar com a configuração de instantâneos de armazenamento reais. Se não for executado com êxito, investigue os problemas antes de prosseguir. O instantâneo de teste deve permanecer até que os primeiros instantâneos reais sejam concluídos.


### <a name="step-7-perform-snapshots"></a>Etapa 7: Executar instantâneos

Quando as etapas de preparação estiverem concluídas, você poderá começar a configurar a configuração do instantâneo de armazenamento real. O script a ser agendado funciona com as configurações de expansão e escalável do SAP HANA. Para execução periódica e regular do script de backup, agende o script usando o utilitário cron. 

Você pode criar três tipos de backups de instantâneos:
- **HANA**: um backup de instantâneo combinado no qual os volumes que contêm o /hana/data e /hana/shared (que também contém /usr/sap) são cobertos pelo instantâneo coordenado. Uma restauração de arquivo único é possível desse instantâneo.
- **Logs**: um backup de instantâneo do volume /hana/logbackups. Nenhum instantâneo HANA é acionado para executar esse instantâneo de armazenamento. Esse volume de armazenamento deve conter os backups de log de transações do SAP HANA. Eles são executados com mais frequência para restringir o crescimento de log e evitar possíveis perdas de dados. Uma restauração de arquivo único é possível desse instantâneo. Não diminua a frequência para menos de 3 minutos.
- **Inicialização**: um instantâneo do volume que contém o LUN (número de unidade lógica) de inicialização do SAP HANA em Instâncias Grandes. Esse backup de instantâneo só é possível com os SKUs tipo I das Instâncias Grandes HANA. Você não pode executar restaurações de arquivo único do instantâneo do volume que contém o LUN de inicialização.


>[!NOTE]
> A sintaxe de chamada para esses três tipos de instantâneos foi alterado com a mudança para os scripts da versão 3.x, que dão suporte a implantações de MCOD. Não é mais necessário especificar o SID HANA de uma instância. Você precisará garantir que as instâncias do SAP HANA de uma unidade sejam configuradas no arquivo de configuração *HANABackupCustomerDetails.txt*.

>[!NOTE]
> Quando você executar o script pela primeira vez, ele poderá apresentar alguns erros inesperados no ambiente com vários SIDs. Executar novamente o script corrige o problema.



A nova sintaxe de chamada para executar instantâneos de armazenamento com o script *azure_hana_backup.pl* é semelhante a esta:

```
HANA backup covering /hana/data and /hana/shared (includes/usr/sap)
./azure_hana_backup.pl hana <snapshot_prefix> <snapshot_frequency> <number of snapshots retained>

For /hana/logbackups snapshot
./azure_hana_backup.pl logs <snapshot_prefix> <snapshot_frequency> <number of snapshots retained>

For snapshot of the volume storing the boot LUN
./azure_hana_backup.pl boot <HANA Large Instance Type> <snapshot_prefix> <snapshot_frequency> <number of snapshots retained>

```

Os parâmetros são os seguintes: 

- O primeiro parâmetro caracteriza o tipo de backup de instantâneo. Os valores permitidos são **hana**, **logs** e **boot**. 
- O parâmetro **<HANA Large Instance Type>** é necessário apenas para backups de volumes de inicialização. Há dois valores válidos com "TypeI" ou "TypeII", dependendo da unidade do HANA em Instâncias Grandes. Para localizar o tipo da unidade, consulte a visão geral do [SAP HANA (Instâncias Grandes) e arquitetura do Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture).  
- O parâmetro **<snapshot_prefix>** é um rótulo de backup ou de instantâneo para o tipo de instantâneo. Há duas finalidades: uma é para você dar um nome a ele para que saiba do que se tratam esses instantâneos. A segunda finalidade é para o script *azure\_hana\_backup.pl* determinar o número de instantâneos de armazenamento retidos de acordo com esse rótulo específico. Se você agendar dois backups de instantâneo de armazenamento do mesmo tipo (como **hana**), com dois rótulos diferentes e definir que 30 instantâneos devem ser mantidos para cada um, 60 instantâneos de armazenamento dos volumes serão afetados. Somente caracteres alfanuméricos ("A-Z, a-z, 0-9"), sublinhado ("_") e traço ("-") são permitidos. 
- O parâmetro **< snapshot_frequency >** é reservado para futuros desenvolvimentos e não tem nenhum impacto. Configure para "3min" ao executar backups do tipo **log** e para "15min" ao executar os outros tipos de backup.
- O parâmetro **<number of snapshots retained>** define a retenção dos instantâneos indiretamente, definindo o número de instantâneos com o mesmo prefixo de instantâneo (rótulo). Esse parâmetro é importante para a execução agendada por meio de cron. Se o número de instantâneos com o mesmo snapshot_prefix exceder o número dado por esse parâmetro, o instantâneo mais antigo será excluído antes de executar um novo instantâneo de armazenamento.

No caso de uma expansão, o script executa uma verificação adicional para garantir que você possa acessar todos os servidores do HANA. O script também verifica se todas as instâncias do HANA retornam o status apropriado das instâncias antes de criar um instantâneo do SAP HANA. O instantâneo SAP HANA é seguido por um instantâneo de armazenamento.

A execução do script `azure_hana_backup.pl` cria o instantâneo de armazenamento nas três fases a seguir:

1. Executa um instantâneo SAP HANA
1. Executa um instantâneo de armazenamento
1. Remove a captura instantânea do SAP HANA que foi criada antes da execução do instantâneo de armazenamento

Para executar o script, chame-o da pasta executável HDB para a qual ele foi copiado. 

O período de retenção é administrado com o número de instantâneos enviados como parâmetro ao executar o script. A quantidade de tempo coberta pelos instantâneos de armazenamento é uma função do período de execução e do número de instantâneos enviados como um parâmetro ao executar o script. Se o número de instantâneos que é mantido exceder o número identificado como parâmetro na chamada do script, o instantâneo de armazenamento mais antigo do mesmo rótulo será excluído antes de executar um novo instantâneo. O número que você der como o último parâmetro da chamada é o número que você poderá usar para controlar o número de instantâneos mantidos. Com esse número, também é possível controlar, indiretamente, o espaço em disco usado para instantâneos. 

> [!NOTE]
>Como alterar o rótulo, a contagem começa novamente. Você precisa ser rigoroso quanto aos rótulos para que os instantâneos não sejam excluídos acidentalmente.

### <a name="snapshot-strategies"></a>Estratégias de instantâneo
A frequência dos instantâneos para os diferentes tipos depende do fato de você usar a funcionalidade de recuperação de desastre do SAP HANA em Instâncias Grandes. Essa funcionalidade depende de instantâneos de armazenamento, os quais podem requerer recomendações especiais para os períodos de execução e frequência dos instantâneos de armazenamento. 

Nas considerações e recomendações a seguir, vamos supor que você *não* usa a funcionalidade de recuperação de desastre que o SAP HANA em Instâncias Grandes oferece. Em vez disso, você usa os instantâneos de armazenamento para ter backups e ser capaz de fornecer recuperação pontual nos últimos 30 dias. Devido às limitações do número de instantâneos e de espaço, os clientes consideraram os seguintes requisitos:

- O tempo de recuperação para uma restauração pontual.
- O espaço usado.
- O ponto de recuperação e os objetivos de tempo de recuperação para possível recuperação de um desastre.
- A execução eventual de backups completos de banco de dados do HANA em discos. Sempre que um backup completo do banco de dados em discos ou a interface **backint** for realizado, a execução de instantâneos de armazenamento falhará. Se você planeja executar backups completos do banco de dados sobre instantâneos de armazenamento, verifique se a execução dos instantâneos de armazenamento está desabilitada durante esse momento.
- O número de instantâneos por volume (limitado a 250).


Para clientes que não usam a funcionalidade de recuperação de desastres de Instâncias Grandes HANA, o período de instantâneo é menos frequente. Nesses casos, os clientes executam os instantâneos combinados em /hana/data e /hana/shared (inclui /usr/sap) em períodos de 12 ou 24 horas e mantêm os instantâneos por um mês. O mesmo acontece com os instantâneos do volume de backup de log de transações. No entanto, a execução de backups de log de transações do SAP HANA no volume do backup de log ocorre em períodos de 5 minutos a 15 minutos.

Os instantâneos de armazenamento agendados executam melhor utilizando o cron. Utilize o mesmo script para todos os backups e necessidades de recuperação de desastre e modifique as entradas de script para corresponder aos vários tempos de backup solicitados. Esses instantâneos são todos agendados de forma diferente no cron dependendo de seu tempo de execução: por hora, 12 horas, diária ou semanal. 

A seguir, um exemplo de uma programação cron em /etc/crontab:
```
00 1-23 * * * ./azure_hana_backup.pl hana hourlyhana 15min 46
10 00 * * *  ./azure_hana_backup.pl hana dailyhana 15min 28
00,05,10,15,20,25,30,35,40,45,50,55 * * * *  Perform SAP HANA transaction log backup
22 12 * * *  ./azure_hana_backup.pl log dailylogback 3min 28
30 00 * * *  ./azure_hana_backup.pl boot TypeI dailyboot 15min 28
```
No exemplo anterior, há um instantâneo combinado por hora que aborda os volumes que contêm as localizações /hana/data e /hana/shared (que inclui /usr/sap). Use esse tipo de instantâneo para uma recuperação pontual mais rápida nos últimos dois dias. Além disso, há um instantâneo diário nesses volumes. Assim, você tem dois dias de cobertura por instantâneos por hora além de quatro semanas de cobertura por instantâneos diários. Além disso, o backup de log de transações tem seu backup feito diariamente. Esses backups também são mantidos por quatro semanas. Como você pode ver na terceira linha de crontab, o backup de log de transações do HANA está agendado para ser executado a cada 5 minutos. As horas de início dos trabalhos de cron diferentes que executam os instantâneos de armazenamento são alternados, assim, os instantâneos não são executados simultaneamente em um determinado ponto no tempo. 

No exemplo a seguir, você executa um instantâneo combinado que cobre os volumes que contêm as localizações /hana/data e /hana/shared (que inclui /usr/sap) por hora. Mantenha esses instantâneos por dois dias. Os instantâneos dos volumes de backup de log de transações são executados em uma base de 5 minutos e são mantidos por 4 horas. Como antes, o backup do arquivo de log de transações do HANA está agendado para ser executado a cada 5 minutos. O instantâneo do volume de backup de log de transações é executado com um atraso de 2 minutos após o backup de log de transações ter sido inciado. Dentro desses 2 minutos, o backup de log de transações do SAP HANA deverá ser concluído sob circunstâncias normais. Como antes, o volume que contém o LUN de inicialização tem seu backup feito uma vez por dia por um instantâneo de armazenamento e é mantido por quatro semanas.

```
10 0-23 * * * ./azure_hana_backup.pl hana hourlyhana 15min 48
0,5,10,15,20,25,30,35,40,45,50,55 * * * *  Perform SAP HANA transaction log backup
2,7,12,17,22,27,32,37,42,47,52,57 * * * *  ./azure_hana_backup.pl log logback 3min 48
30 00 * * *  ./azure_hana_backup.pl boot TypeII dailyboot 15min 28
```

O gráfico a seguir ilustra as sequências do exemplo anterior, excluindo o LUN de inicialização:

![Relação entre os backups e os instantâneos](./media/hana-overview-high-availability-disaster-recovery/backup_snapshot_updated0921.PNG)

O SAP HANA executa gravações regulares no volume /hana/log para documentar as alterações confirmadas no banco de dados. Regularmente, o SAP HANA grava um ponto de salvamento para o volume /hana/data. Conforme especificado em crontab, um backup de log de transações do SAP HANA é executado a cada 5 minutos. Você também pode ver que um instantâneo SAP HANA é executado a cada hora, como resultado do acionamento de um instantâneo de armazenamento combinado nos volumes /hana/data e /hana/shared. Depois do êxito do instantâneo do HANA, o instantâneo de armazenamento combinado é executado. Conforme instruído em crontab, o instantâneo de armazenamento no volume /hana/logbackup é executado a cada 5 minutos, aproximadamente 2 minutos após o backup de log de transações do HANA.

> 

>[!IMPORTANT]
> O uso de instantâneos de armazenamento para backups do SAP HANA só é válido quando os instantâneos são executados junto com backups de log de transações do SAP HANA. Esses backups de log de transações precisam abordar os períodos de tempo entre os instantâneos de armazenamento. 

Se você firmou um compromisso com os usuários de uma recuperação pontual de 30 dias, será necessário:

- Em casos extremos, acessar um instantâneo de armazenamento combinado no /hana/data e /hana/shared que tem 30 dias.
- Tenha backups de log de transações contíguos que abrangem o tempo entre todos os instantâneos de armazenamento combinado. Portanto, o instantâneo mais antigo do volume de backup de log de transações deve ser de 30 dias. Esse não será o caso se você copiar os backups de log de transações para outro compartilhamento NFS localizado no armazenamento do Azure. Nesse caso, você pode efetuar pull de backups de log de transações antigos desse compartilhamento NFS.

Para beneficiar-se de instantâneos de armazenamento e replicação de armazenamento eventual dos backups de log de transações, você precisa alterar o local para o qual o SAP HANA grava os backups de log de transações. É possível fazer essa alteração no HANA Studio. Embora o SAP HANA faça backup de segmentos de log completos automaticamente, você deve especificar um intervalo de backup de log para ser determinístico. Isso é especialmente verdadeiro quando você usa a opção de recuperação de desastre, porque geralmente convém executar backups de log com um período determinístico. No caso abaixo, usamos 15 minutos como o intervalo de backup de log.

![Agendar logs de backup SAP HANA no Studio de SAP HANA](./media/hana-overview-high-availability-disaster-recovery/image5-schedule-backup.png)

Você também pode escolher os backups com frequência maior que 15 minutos. Uma configuração mais frequente é frequentemente usada em conjunto com a funcionalidade de recuperação de desastre do SAP HANA nas Instâncias Grandes. Alguns clientes executam backups de log de transações a cada 5 minutos.  

Se o backup do banco de dados nunca foi feito, a etapa final será executar um backup de banco de dados com base no arquivo para criar uma entrada de backup única que deve existir no catálogo de backup. Caso contrário, o SAP HANA não pode iniciar os backups de log indicados.

![Faça um backup baseado em arquivo para criar uma única entrada de backup](./media/hana-overview-high-availability-disaster-recovery/image6-make-backup.png)


Após seus primeiros instantâneos de armazenamento serem executados com êxito, você poderá excluir o instantâneo de teste que foi executado na etapa 6. Para fazer isso, execute o script `removeTestStorageSnapshot.pl`:
```
./removeTestStorageSnapshot.pl
```

A seguir, um exemplo da saída do script:
```
Checking Snapshot Status for h80
**********************Checking access to Storage**********************
Storage Snapshot Access successful.
**********************Getting list of volumes that match HANA instance specified**********************
Collecting set of volumes hosting HANA matching pattern *h80* ...
Volume show completed successfully.
Adding volume hana_data_h80_mnt00001_t020_vol to the snapshot list.
Adding volume hana_log_backups_h80_t020_vol to the snapshot list.
Adding volume hana_shared_h80_t020_vol to the snapshot list.
**********************Adding list of snapshots to volume list**********************
Collecting set of snapshots for each volume hosting HANA matching pattern *h80* ...
**********************Displaying Snapshots by Volume**********************
hana_data_h80_mnt00001_t020_vol
Test_HANA_Snapshot.2018-02-06_1753.3
Test_HANA_Snapshot.2018-02-06_1815.2
….
Command completed successfully.
Exiting with return code: 0
Command completed successfully.
```


### <a name="monitoring-the-number-and-size-of-snapshots-on-the-disk-volume"></a>Monitoramento do número e tamanho dos instantâneos no volume de disco

Em um volume de armazenamento específico, você pode monitorar o número de instantâneos e o consumo de armazenamento desses instantâneos. O comando `ls` não mostra o diretório de instantâneo ou arquivos. No entanto, o comando do sistema operacional Linux `du` mostra detalhes sobre esses instantâneos de armazenamento, porque eles são armazenados nos mesmos volumes. O comando pode ser usado com as seguintes opções:

- `du –sh .snapshot`: essa opção fornece um total de todos os instantâneos no diretório de instantâneos.
- `du –sh --max-depth=1`: essa opção lista todos os instantâneos salvos na pasta **.snapshot** e o tamanho de cada instantâneo.
- `du –hc`: essa opção fornece o tamanho total usado por todos os instantâneos.

Use estes comandos para verificar se os instantâneos foram obtidos e armazenado não estão consumindo todo o armazenamento nos volumes.

>[!NOTE]
>Os instantâneos do LUN de inicialização não são visíveis com os comandos acima.

### <a name="getting-details-of-snapshots"></a>Obter detalhes de instantâneos
Para obter mais detalhes sobre instantâneos, é possível usar o script `azure_hana_snapshot_details.pl`. Esse script poderá ser executado em qualquer local se houver um servidor ativo no local de recuperação de desastres. O script fornece a saída a seguir dividida por cada volume que contém os instantâneos: 
   * O tamanho do total de instantâneos em um volume
   * Os detalhes a seguir em cada instantâneo nesse volume: 
      - Nome do instantâneo 
      - Criar tempo 
      - Tamanho do instantâneo
      - Frequência do instantâneo
      - ID do HANA Backup associada a esse instantâneo, se relevante

A seguir, um exemplo da sintaxe de execução de script:

```
./azure_hana_snapshot_details.pl 
```

Como o script tenta recuperar a ID de backup do HANA, ele precisa se conectar à instância do SAP HANA. Essa conexão requer que o arquivo de configuração *HANABackupCustomerDetails.txt* seja configurado corretamente. Uma saída de dois instantâneos em um volume pode ser semelhante à seguinte:

```
**********************************************************
****Volume: hana_shared_SAPTSTHDB100_t020_vol       ***********
**********************************************************
Total Snapshot Size:  411.8MB
----------------------------------------------------------
Snapshot:   customer.2016-09-20_1404.0
Create Time:   "Tue Sep 20 18:08:35 2016"
Size:   2.10MB
Frequency:   customer 
HANA Backup ID:   
----------------------------------------------------------
Snapshot:   customer2.2016-09-20_1532.0
Create Time:   "Tue Sep 20 19:36:21 2016"
Size:   2.37MB
Frequency:   customer2
HANA Backup ID:   
```


### <a name="file-level-restore-from-a-storage-snapshot"></a>Restauração no nível do arquivo de um instantâneo de armazenamento
Para os tipos de instantâneos **hana** e **logs**, é possível acessar os instantâneos diretamente nos volumes no diretório **.snapshot**. Há um subdiretório para cada um dos instantâneos. É possível copiar cada arquivo no estado em que estava no ponto do instantâneo desse subdiretório para a estrutura de diretório real. Na versão atual do script, não há **NENHUM** script de restauração fornecido para a restauração de instantâneo como um autoatendimento (embora a restauração de instantâneo pode ser executada como parte dos scripts de DR de autoatendimento no site de DR durante o failover). Você deve entrar em contato com a equipe de operações do Microsoft através da abertura de uma solicitação de serviço para restaurar um instantâneo desejado dos instantâneos existentes disponíveis.

>[!NOTE]
>A restauração de arquivo único não funciona para instantâneos do LUN de inicialização, independentemente do tipo de unidade do HANA em Instâncias Grandes. O diretório **.snapshot** não é exposto no LUN de inicialização. 


### <a name="reducing-the-number-of-snapshots-on-a-server"></a>Reduzindo o número de instantâneos em um servidor

Conforme explicado anteriormente, você pode reduzir o número armazenado de certos rótulos de instantâneos. Os dois últimos parâmetros do comando para iniciar um instantâneo são o rótulo e o número de instantâneos que você deseja manter.

```
./azure_hana_backup.pl hana dailyhana 15min 28
```

No exemplo anterior, o rótulo de instantâneo é **dailyhana** e o número de instantâneos com este rótulo a ser retido é **28**. Ao responder ao consumo de espaço em disco, pode ser útil reduzir o número de instantâneos armazenados. A maneira fácil de reduzir o número de instantâneos para 15, por exemplo, é executar o script com o último parâmetro definido como **15**:

```
./azure_hana_backup.pl hana dailyhana 15min 15
```

Se você executar o script com essa configuração, o número de instantâneos, incluindo o novo instantâneo de armazenamento, é 15. Os 15 instantâneos mais recentes são mantidos e os 15 instantâneos mais antigos são excluídos.

 >[!NOTE]
 > Esse script reduzirá o número de instantâneos apenas se houver instantâneos com mais de 1 hora. O script não exclui instantâneos com menos de 1 hora. Essas restrições estão relacionadas à funcionalidade de recuperação de desastre opcional oferecida.

Se você não deseja mais manter um conjunto de instantâneos com um rótulo de backup **hanadaily** nos exemplos de sintaxe, é possível executar o script com **0** como o número de retenção. Todos os instantâneos que correspondem a esse rótulo são removidos. Entretanto, remover todos os instantâneos pode afetar as funcionalidades de recuperação de desastre do SAP HANA nas Instâncias Grandes.

Uma segunda opção para excluir instantâneos específicos é usar o script `azure_hana_snapshot_delete.pl`. Este script foi desenvolvido para excluir um instantâneo ou conjunto de instantâneos usando a ID de backup do HANA conforme encontrada no HANA Studio ou por meio do próprio nome do instantâneo. No momento, a ID de backup só está vinculada aos instantâneos criados para o tipo de instantâneo **hana**. Backups de instantâneos do tipo **logs** e **boot** não executam um instantâneo do SAP HANA e, portanto, não há ID de backup a ser localizada para esses instantâneos. Se o nome do instantâneo for inserido, ele procurará todos os instantâneos em volumes diferentes que correspondam ao nome do instantâneo inserido. 

Chame o script de que você precisa para especificar o SID da instância do HANA usando a sintaxe de chamada do script:

```
./azure_hana_snapshot_delete.pl <SID>

```

Execute o script como usuário **raiz**.

Se selecionar um instantâneo, você poderá excluir cada um deles individualmente. Primeiro, forneça o volume que contém o instantâneo e, em seguida, forneça o nome do instantâneo. Se o instantâneo existir nesse volume e tiver mais de 1 hora, ele será excluído. É possível encontrar os nomes de volume e nomes de instantâneo executando o script `azure_hana_snapshot_details`. 

>[!IMPORTANT]
>Se houver dados que somente existam no instantâneo que você está excluindo, depois que o instantâneo for excluído, esses dados serão perdidos para sempre.

   

### <a name="recover-to-the-most-recent-hana-snapshot"></a>Recuperar para o instantâneo mais recente do HANA

Em um cenário de inatividade na produção, o processo de recuperação de um instantâneo de armazenamento poderá ser iniciado como um incidente do cliente com suporte do Microsoft Azure. É um assunto de alta urgência se os dados foram excluídos em um sistema de produção, e a única maneira de recuperá-los é restaurar o banco de dados de produção.

Em uma situação diferente, uma recuperação pontual pode ter baixa urgência e ser planejada com dias de antecedência. Você pode planejar essa recuperação com o SAP HANA no Gerenciamento de Serviços do Azure, em vez de gerar uma marca de alta prioridade. Por exemplo, você pode planejar atualizar o software SAP aplicando um novo pacote de melhoria. Em seguida, é necessário reverter para um instantâneo que representa o estado antes da atualização de pacote de melhoria.

Antes de enviar a solicitação, é necessário se preparar. A equipe de Gerenciamento de Serviços do SAP HANA no Azure poderá manipular a solicitação e fornecer os volumes restaurados. Depois, restaure o banco de dados do HANA com base nos instantâneos. 

A seguir, é apresentado como preparar a solicitação:

>[!NOTE]
>A interface do usuário pode variar das capturas de tela seguintes, dependendo da versão do SAP HANA usada.

1. Decida qual instantâneo para restaurar. Somente o volume de dados/hana é restaurado, a menos que indicado o contrário. 

1. Desligar a instância do HANA.

 ![Desligar a instância do HANA](./media/hana-overview-high-availability-disaster-recovery/image7-shutdown-hana.png)

1. Desmonte os volumes de dados em cada nó do banco de dados do HANA. Se os volumes de dados ainda estiverem montados para o sistema operacional, a restauração do instantâneo falhará.
 ![Desmontar os volumes de dados em cada nó de banco de dados do HANA](./media/hana-overview-high-availability-disaster-recovery/image8-unmount-data-volumes.png)

1. Abra uma solicitação de suporte do Azure e inclua instruções sobre a restauração de um instantâneo específico.

 - Durante a restauração: o Gerenciamento de Serviços do SAP HANA no Azure pode solicitar que você participe de uma chamada em conferência para garantir a coordenação, a verificação e a confirmação de que o instantâneo de armazenamento correto foi restaurado. 

 - Depois da restauração: o SAP HANA no Gerenciamento de Serviços do Azure será notificado quando o instantâneo de armazenamento tiver sido restaurado.

1. Depois que o processo de restauração for concluído, monte novamente todos os volumes de dados.

 ![Montar novamente todos os volumes de dados](./media/hana-overview-high-availability-disaster-recovery/image9-remount-data-volumes.png)

1. Selecione as opções de recuperação no SAP HANA Studio, caso elas não apareçam automaticamente quando você se reconectar ao HANA DB por meio do SAP HANA Studio. O exemplo a seguir mostra uma restauração para o último instantâneo HANA. Um instantâneo de armazenamento incorpora um instantâneo do HANA. Se você restaurar para o instantâneo de armazenamento mais recente, ele deverá ser o instantâneo mais recente do HANA. (Se você restaurar para um instantâneo de armazenamento mais antigo, será necessário localizar o instantâneo do HANA com base na hora em que o instantâneo de armazenamento foi tirado.)

 ![Selecionar as opções de recuperação no SAP HANA Studio](./media/hana-overview-high-availability-disaster-recovery/image10-recover-options-a.png)

1. Selecione **Recuperar o banco de dados para um instantâneo de backup ou armazenamento de dados específico**.

 ![A janela Especificar o tipo de recuperação](./media/hana-overview-high-availability-disaster-recovery/image11-recover-options-b.png)

1. Selecione **Especificar backup sem catálogo**.

 ![A janela Especificar o local de backup](./media/hana-overview-high-availability-disaster-recovery/image12-recover-options-c.png)

1. Na lista **Tipo de Destino**, selecione **Instantâneo**.

 ![A janela Especificar o Backup a ser Recuperado](./media/hana-overview-high-availability-disaster-recovery/image13-recover-options-d.png)

1. Selecione **Concluir** para iniciar o processo de recuperação.

 ![Selecione “Concluir” para iniciar o processo de recuperação](./media/hana-overview-high-availability-disaster-recovery/image14-recover-options-e.png)

1. O banco de dados do HANA é restaurado e recuperado para o instantâneo HANA incluído pelo instantâneo de armazenamento.

 ![O banco de dados do HANA é restaurado e recuperado para o instantâneo do HANA](./media/hana-overview-high-availability-disaster-recovery/image15-recover-options-f.png)

### <a name="recover-to-the-most-recent-state"></a>Recuperar para o estado mais recente

O processo a seguir restaura um instantâneo HANA que foi incluído no instantâneo de armazenamento. Ele restaura os backups de log de transação para o estado mais recente do banco de dados antes de restaurar o instantâneo de armazenamento.

>[!IMPORTANT]
>Antes de continuar, verifique se você tem uma cadeia completa e contígua de backups de log de transações. Sem esses backups, você não poderá restaurar o estado atual do banco de dados.

1. Conclua as etapas de 1 a 6 em [Recuperar para o instantâneo mais recente do HANA](#recovering-to-the-most-recent-hana-snapshot).

1. Selecione **Recuperar o banco de dados para seu estado mais recente**.

 ![Selecione “Recuperar o banco de dados para seu estado mais recente”](./media/hana-overview-high-availability-disaster-recovery/image16-recover-database-a.png)

1. Especifique o local dos backups de log mais recentes do HANA. O local especificado precisa conter todos os backups de log de transações do instantâneo do HANA até o estado mais recente.

 ![Especifique o local dos backups de log mais recentes do HANA](./media/hana-overview-high-availability-disaster-recovery/image17-recover-database-b.png)

1. Selecione um backup como uma base recuperar o banco de dados. Nesse exemplo, o instantâneo HANA na captura de tela, é o instantâneo HANA que foi incluído no instantâneo de armazenamento. 

 ![Selecione um backup como uma base recuperar o banco de dados](./media/hana-overview-high-availability-disaster-recovery/image18-recover-database-c.png)

1. Desmarque a caixa de seleção **Usar Backups Delta** se não existirem deltas entre a hora do instantâneo do HANA e o estado mais recente.

 ![Desmarque a caixa de seleção “Usar Backups de Delta” se nenhum delta existir](./media/hana-overview-high-availability-disaster-recovery/image19-recover-database-d.png)

1. Na tela de resumo, selecione **Concluir** para iniciar o procedimento de restauração.

 ![Clique em “Concluir” na tela de resumo](./media/hana-overview-high-availability-disaster-recovery/image20-recover-database-e.png)

### <a name="recover-to-another-point-in-time"></a>Recuperar para outro ponto no tempo
Para recuperar para um ponto de tempo entre o instantâneo HANA (incluído no instantâneo de armazenamento) e um posterior àquele da recuperação pontual, execute as seguintes etapas:

1. Certifique-se de que você tenha todos os backups de log de transações do instantâneo do HANA para a hora em que deseja recuperar.
1. Inicie o procedimento em [Recuperar para o estado mais recente](#recovering-to-the-most-recent-state).
1. Na etapa 2 do procedimento, na janela **Especificar o Tipo de Recuperação**, selecione **Recuperar o banco de dados para o seguinte ponto** e, em seguida, especifique o ponto em questão. 
1. Conclua as etapas de 3 a 6.

### <a name="monitor-the-execution-of-snapshots"></a>Monitorar a execução de instantâneos

À medida que você usa instantâneos de armazenamento do SAP HANA nas Instâncias Grandes, você também precisará monitorar a execução desses instantâneos. O script que executa um instantâneo de armazenamento grava a saída em um arquivo e salva no mesmo local que os scripts Perl. Um arquivo separado é criado para cada instantâneo de armazenamento. A saída de cada arquivo mostra as várias fases de execução do script de instantâneo:

1. Localiza os volumes necessários para criar um instantâneo.
1. Localiza os instantâneos tirados desses volumes.
1. Exclui instantâneos existentes eventuais para corresponder ao número de instantâneos que você especificou.
1. Cria um instantâneo do SAP HANA.
1. Cria o instantâneo de armazenamento sobre os volumes.
1. Exclui o instantâneo do SAP HANA.
1. Renomeia o instantâneo mais recente para **.0**.

A parte mais importante do cab de script identificada é esta parte:
```
**********************Creating HANA snapshot**********************
Creating the HANA snapshot with command: "./hdbsql -n localhost -i 01 -U SCADMIN01 "backup data create snapshot"" ...
HANA snapshot created successfully.
**********************Creating Storage snapshot**********************
Taking snapshot hourly.recent for hana_data_lhanad01_t020_vol ...
Snapshot created successfully.
Taking snapshot hourly.recent for hana_log_backup_lhanad01_t020_vol ...
Snapshot created successfully.
Taking snapshot hourly.recent for hana_log_lhanad01_t020_vol ...
Snapshot created successfully.
Taking snapshot hourly.recent for hana_shared_lhanad01_t020_vol ...
Snapshot created successfully.
Taking snapshot hourly.recent for sapmnt_lhanad01_t020_vol ...
Snapshot created successfully.
**********************Deleting HANA snapshot**********************
Deleting the HANA snapshot with command: "./hdbsql -n localhost -i 01 -U SCADMIN01 "backup data drop snapshot"" ...
HANA snapshot deletion successfully.
```
Você pode ver a este exemplo como o script registros a criação do instantâneo HANA. No caso de escala horizontal, esse processo é iniciado no nó principal. O nó mestre iniciará a criação síncrona de instantâneos SAP HANA em cada um de nós de trabalho. O instantâneo de armazenamento é então obtido. Após a execução bem-sucedida de instantâneos de armazenamento, o instantâneo do HANA será excluído. A exclusão do instantâneo do HANA é iniciada com base no nó mestre.


## <a name="disaster-recovery-principles"></a>Princípios da recuperação de desastre
O SAP HANA nas Instâncias Grandes oferece uma funcionalidade de recuperação de desastre entre carimbos do SAP HANA nas Instâncias Grandes em diferentes regiões do Azure. Por exemplo, se você implantar unidades do SAP HANA nas Instâncias Grandes na região Oeste dos EUA do Azure, será possível usar as unidades do SAP HANA nas Instâncias Grandes na região Leste dos EUA como unidades de recuperação de desastre. Como mencionado anteriormente, a recuperação de desastre não está configurada automaticamente, porque requer que você pague por outra unidade de Instância Grande do HANA na região de recuperação de desastre. A configuração de recuperação de desastres funciona para configurações de expansão, bem como escaláveis. 

Nos cenários implantados até agora, os clientes usam a unidade na região de recuperação de desastre para executar os sistemas de não produção que usam uma instância instalada do HANA. A unidade de Instância Grande HANA deve ser da mesma SKU que a SKU usada para fins de produção. A imagem a seguir mostra como é a configuração do disco entre a unidade do servidor na região de produção do Azure e a região de recuperação de desastre:

![Configuração de recuperação de desastre do ponto de vista do disco](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_setup.PNG)

Conforme mostrado neste gráfico de visão geral, é necessário solicitar um segundo conjunto de volumes de disco. Os volumes de disco de destino são do mesmo tamanho que os volumes de produção para a instância de produção em unidades de recuperação de desastre. Esses volumes de disco estão associados com a unidade de servidor da Instância Grande do HANA no local de recuperação de desastre. Os volumes a seguir são replicados da região de produção para o local da recuperação de desastre:

- /hana/data
- /hana/logbackups 
- /hana/shared (inclui /usr/sap)

O volume /hana/log não é replicado, porque o log de transações do SAP HANA não é necessários da maneira como a restauração desses volumes é feita. 

A base da funcionalidade de recuperação de desastre oferecida é a funcionalidade de replicação de armazenamento oferecida pela infraestrutura do SAP HANA nas Instâncias Grandes. A funcionalidade usada no lado do armazenamento não é um fluxo constante de alterações replicadas de maneira assíncrona conforme as alterações ocorrem no volume de armazenamento. Em vez disso, ela é um mecanismo que depende do fato de que os instantâneos desses volumes são criados com regularidade. O delta entre um instantâneo já replicado e um novo que ainda não foi replicado é, então, transferido para o site de recuperação de desastre em volumes de disco de destino.  Esses instantâneos são armazenados nos volumes e, no caso de um failover de recuperação de desastre, precisam ser restaurados nesses volumes.  

A primeira transferência dos dados completos do volume deve ser feita antes que a quantidade de dados se torne menor do que os deltas entre instantâneos. Como resultado, os volumes no local de recuperação de desastre contêm todos os instantâneos de volume executados no local de produção. Eventualmente, é possível usar esse sistema de DR para obter um status anterior para recuperar dados perdidos, sem reverter o sistema de produção.

No caso de implantações de MCOD com várias instâncias do SAP HANA independentes em uma unidade do SAP HANA nas Instâncias Grandes, espera-se que todas as instâncias do SAP HANA tenham o armazenamento replicado para o lado da DR.

Nos casos em que você usa a Replicação de sistema do HANA como funcionalidade de alta disponibilidade no seu site de produção e usa a replicação baseada em armazenamento para o site de DR, os volumes de ambos os nós do site primário para a instância de DR são replicados. Você deve adquirir armazenamento adicional (mesmo tamanho, a partir do nó primário) no site de DR para acomodar a replicação do primário e secundário para DR. 



>[!NOTE]
>A funcionalidade de replicação de armazenamento do SAP HANA nas Instâncias Grandes está espelhando e replicando os instantâneos de armazenamento. Se você não executar instantâneos de armazenamento como introduzidos na seção [Backup e restauração](#backup-and-restore) deste artigo, não poderá haver nenhuma replicação para o site de recuperação de desastre. A execução do instantâneo de armazenamento é um pré-requisito para a replicação de armazenamento para o local recuperação de desastres.



## <a name="preparation-of-the-disaster-recovery-scenario"></a>Preparação do cenário de recuperação de desastre
Nesse cenário, você tem um sistema de produção sendo executado do SAP HANA nas Instâncias Grandes na região do Azure de produção. Para as etapas a seguir, suponhamos que o SID desse sistema HANA seja "PRD" e que você tenha um sistema de não produção em execução no SAP HANA nas Instâncias Grandes na região de DR do Azure. Finalmente, vamos supor que o SID seja "TST." A imagem a seguir mostra esta configuração:

![Início da configuração da recuperação de desastre](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start1.PNG)

Se a instância do servidor ainda não tiver sido solicitada com o conjunto de volumes de armazenamento adicional, o Gerenciamento de Serviços do SAP HANA do Azure anexa o conjunto adicional de volumes como um destino na réplica de produção para a unidade do SAP HANA nas Instâncias Grandes na qual você está executando a instância do TST HANA. Para essa finalidade, é necessário fornecer o SID da sua instância do HANA de produção. Depois que o Gerenciamento de Serviços do SAP HANA no Azure confirmar a anexação desses volumes, será necessário montar esses volumes na unidade da Instância Grande do HANA.

![Próxima etapa de configuração da recuperação de desastre](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start2.PNG)

A próxima etapa é instalar a segunda instância do SAP HANA na unidade do SAP HANA nas Instâncias Grandes na região do Azure de recuperação de desastre, em que você executa a instância do TST HANA. A instância do SAP HANA recém-instalada precisa ter o mesmo SID. Os usuários criados precisam ter os mesmos UID e ID de grupo que a instância de produção. Se a instalação for bem-sucedida, será necessário:

- Execute a etapa 2 da preparação do instantâneo de armazenamento discutida anteriormente neste artigo.
- Crie uma chave pública para a unidade de DR da unidade do SAP HANA nas Instâncias Grandes, caso ainda não tenha feito isso. Consulte a etapa 3 da preparação do instantâneo de armazenamento discutida anteriormente neste artigo.
- Mantenha *HANABackupCustomerDetails.txt* com a nova instância do HANA e teste se a conectividade com o armazenamento funciona corretamente.  
- Pare a instância do SAP HANA instalada recentemente na unidade do SAP HANA nas Instâncias Grandes na região do Azure de DR.
- Desmonte esses volumes PRD e entre em contato com o Gerenciamento de Serviços do SAP HANA no Azure. Os volumes não podem permanecer montados na unidade, pois eles não podem ser acessados enquanto estão em funcionamento como o destino de replicação de armazenamento.  

![Etapa de configuração da recuperação de desastre antes de estabelecer a replicação](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start3.PNG)

A equipe de operações estabelece o relacionamento de replicação entre os volumes PRD na região do Azure de produção e os volumes PRD na região do Azure de recuperação de desastre.

>[!IMPORTANT]
>O volume /hana/log não é replicado porque não é necessário restaurar o banco de dados do SAP HANA replicado para um estado consistente no local de recuperação de desastre.

Em seguida, configure ou ajuste a programação de backup do instantâneo de armazenamento para chegar ao RTO e RPO no caso de desastre. Para minimizar o objetivo de ponto de recuperação, defina os intervalos de replicação a seguir no serviço de Instância Grande do HANA:
- Para os volumes cobertos pelo instantâneo combinado (tipo de instantâneo **hana**), defina para replicar a cada 15 minutos para os destinos de volume de armazenamento equivalentes no site de recuperação de desastre.
- Para o volume de backup de log de transações (tipo de instantâneo **logs**), defina para replicar a cada 3 minutos para os destinos de volume de armazenamento equivalentes no site de recuperação de desastre.

Para minimizar o objetivo de ponto de recuperação, configure o seguinte:
- Execute um instantâneo de armazenamento do tipo **hana** (consulte a “Etapa 7 – Executar instantâneos”) a cada 30 minutos a 1 hora.
- Execute backups de log de transações do SAP HANA a cada 5 minutos.
- Execute um instantâneo de armazenamento do tipo **logs** a cada 5-15 minutos. Com esse período de intervalo, você obtém um RPO de cerca de 15 a 25 minutos.

Com essa configuração, a sequência de backups de log de transações, os instantâneos de armazenamento e a replicação do volume de backup de log de transações do HANA e /hana/data e /hana/shared (inclui /usr/sap) podem ter a aparência dos dados mostrados neste gráfico:

 ![Relação entre um instantâneo de backup de log de transações e um espelho de ajuste em um eixo de tempo](./media/hana-overview-high-availability-disaster-recovery/snapmirror.PNG)

Para alcançar um RPO ainda melhor, no caso de recuperação de desastre, você pode copiar os backups de log de transações do HANA do SAP HANA (Instâncias Grandes) do Azure e para outra região do Azure. Para obter essa redução adicional de RPO, execute as etapas a seguir:

1. Faça backup do log de transações do HANA com a maior frequência possível para /hana/logbackups.
1. Use rsync para copiar os backups de log de transações para as máquinas virtuais do Azure hospedadas pelo compartilhamento NFS. As VMs estão nas redes virtuais do Azure na região de produção do Azure e nas regiões da recuperação de desastre. É necessário conectar as redes virtuais do Azure ao circuito que se conecta as Instâncias Grandes HANA de produção ao Azure. Consulte o gráfico na seção [Considerações de rede para recuperação de desastre com Instâncias Grandes HANA](#Network-considerations-for-disaster recovery-with-HANA-Large-Instances). 
1. Mantenha os backups de log de transações na região da VM anexada ao armazenamento exportado de NFS.
1. No caso de um failover de desastre, complemente os backups de log de transações que encontrar no volume /hana/logbackups com backups de log de transações mais recentes no compartilhamento de NFS no site de recuperação de desastre. 
1. Inicie um backup de log de transações para restaurar para o backup mais recente que foi salvo na região de recuperação de desastre.

Quando as operações do SAP HANA nas Instâncias Grandes confirmam a configuração do relacionamento de replicação e você inicia os backups de instantâneos de armazenamento de execução, a replicação de dados é iniciada.

![Etapa de configuração da recuperação de desastre antes de estabelecer a replicação](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start4.PNG)

Conforme a replicação avança, os instantâneos nos volumes PRD nas regiões do Azure de recuperação de desastre não são restaurados. Eles só são armazenados. Se os volumes forem montados nesse estado, eles representarão o estado em que você desmontou esses volumes após a instância do PRD SAP HANA ter sido instalada na unidade de servidor na região do Azure de recuperação de desastre. Eles também representam backups de armazenamento que ainda não foram restaurados.

No caso de um failover, também é possível optar por restaurar para um instantâneo de armazenamento mais antigo em vez do instantâneo de armazenamento mais recente.

## <a name="disaster-recovery-failover-procedure"></a>Procedimento de failover de recuperação de desastre
Há dois casos a serem considerados durante o failover para o site de recuperação de desastre:

- É necessário que o banco de dados do SAP HANA retorne para o status mais recente dos dados. Nesse caso, há um script de autoatendimento com o qual você pode fazer failover sem precisar entrar em contato com a Microsoft. No entanto, para o failback, você precisa trabalhar com a Microsoft.
- Se quiser restaurar para um instantâneo de armazenamento que não seja o último instantâneo replicado. Nesse caso, você precisa trabalhar com a Microsoft. 

>[!NOTE]
>As etapas a seguir precisam ser executadas na unidade do SAP HANA nas Instâncias Grandes, que representa a unidade de DR. 
 
Para restaurar os instantâneos de armazenamento replicados mais recentes, execute as etapas a seguir: 

1. Desligue a instância de não produção do HANA na unidade de recuperação de desastre do SAP HANA nas Instâncias Grandes que você está executando. Isso ocorre porque há uma instância de produção do HANA inativa pré-instalada.
1. Certifique-se de que não há nenhum processo do SAP HANA em execução. Use o comando a seguir para essa verificação: `/usr/sap/hostctrl/exe/sapcontrol –nr <HANA instance number> - function GetProcessList`. A saída agora deve mostrar o processo **hdbdaemon** em um estado interrompido e nenhum outro processo do HANA em estado de execução ou iniciado.
1. Na unidade do SAP HANA em Instâncias Grandes no site de recuperação de desastre, execute o script *azure_hana_dr_failover.pl*. O script está solicitando que um SID do SAP HANA seja restaurado. Quando solicitado, digite o SID do SAP HANA que foi replicado e que é mantido no arquivo *HANABackupCustomerDetails.txt* na unidade do SAP HANA em Instâncias Grandes no site de DR. 

      Se você quiser que várias instâncias do SAP HANA em Instâncias Grandes sejam substituídas, será necessário executar o script várias vezes. Quando solicitado, digite o SID do SAP HANA que você deseja fazer failover e restaurar. Na conclusão, o script mostrará uma lista de pontos de montagem dos volumes que são adicionados à unidade do SAP HANA em Instâncias Grandes. Essa lista inclui os volumes de recuperação de desastre restaurados.

1. Monte os volumes de recuperação de desastre restaurados usando comandos do sistema operacional Linux na unidade do SAP HANA em Instâncias Grandes no site de recuperação de desastre. 
1. Inicie a instância de produção do SAP HANA inativa.
1. Se optar por copiar logs de backup de log de transações para reduzir o tempo de RPO, será necessário mesclar esses backups de log de transações no diretório /hana/logbackups da recuperação de desastre montado recentemente. Não substitua backups existentes. Copie os backups mais recentes que não foram replicados com a replicação mais recente de um instantâneo de armazenamento.
1. Também é possível restaurar arquivos únicos dos instantâneos que foram replicados para o volume /hana/shared/PRD na região do Azure de recuperação de desastre. 

Você pode testar o failover de recuperação de desastres sem afetar a relação de replicação real. Para fazer um failover de teste, siga as etapas anteriores 1 e 2 e continue com a seguinte etapa 3.

>[!IMPORTANT]
>*Não* execute nenhuma transação de produção na instância que você criou no site de DR por meio do processo de **teste de um failover** com o script introduzido na etapa 3. Esse comando cria um conjunto de volumes sem relação com o site primário. Como resultado, a sincronização de volta para o site primário *não* é possível. 

Etapa 3 para o teste de failover:

Na unidade do SAP HANA em Instâncias Grandes no site de recuperação de desastre, execute o script **azure_hana_test_dr_failover.pl**. Esse script *não* está interrompendo a relação de replicação entre o site primário e site de recuperação de desastre. Em vez disso, o script está clonando os volumes de armazenamento da recuperação de desastres. Quando o processo de clonagem for bem-sucedido, os volumes clonados serão restaurados para o estado do instantâneo mais recente e, em seguida, serão montados na unidade de recuperação de desastres. O script está solicitando que um SID do SAP HANA seja restaurado. Digite um ou o único SID do SAP HANA que foi replicado e é mantido no *HANABackupCustomerDetails.txt* na unidade do SAP HANA em Instâncias Grandes no site de recuperação de desastre. 

Se você quiser ter várias instâncias do SAP HANA para testar, será necessário executar o script várias vezes. Quando solicitado, digite o SID do SAP HANA da instância que você deseja testar para fazer failover. Após a conclusão, o script mostrará uma lista de pontos de montagem dos volumes que são adicionados à unidade do SAP HANA em Instâncias Grandes. Essa lista inclui os volumes de recuperação de desastres clonados.

Prossiga para a etapa 4.

   >[!NOTE]
   >Se você precisar fazer failover no site de recuperação de desastre para resgatar alguns dados que foram excluídos há algumas horas e precisar que os volumes de DR sejam definidos para um instantâneo anterior, este procedimento se aplicará. 

1. Desligue a instância de não produção do HANA na unidade de recuperação de desastre do SAP HANA nas Instâncias Grandes que você está executando. Isso ocorre porque há uma instância de produção do HANA inativa pré-instalada.
1. Certifique-se de que não há nenhum processo do SAP HANA em execução. Use o comando a seguir para essa verificação: `/usr/sap/hostctrl/exe/sapcontrol –nr <HANA instance number> - function GetProcessList`. A saída agora deve mostrar o processo **hdbdaemon** em um estado interrompido e nenhum outro processo do HANA em estado de execução ou iniciado.
1. Determine para qual nome de instantâneo ou ID de backup do SAP HANA você deseja ter a recuperação de desastre restaurada. Em casos de recuperação de desastres reais, esse instantâneo é geralmente o instantâneo mais recente. Se você precisar recuperar dados perdidos, escolha um instantâneo mais antigo.
1. Contate o Suporte do Azure por meio de uma solicitação de suporte de alta prioridade. Solicite a restauração desse instantâneo (com o nome e a data do instantâneo) ou a ID de backup do HANA no site de DR. O padrão é que o lado das operações restaura apenas o volume /hana/data. Se você quiser ter os volumes /hana/logbackups também, será necessário declarar isso especificamente. *Não restaure o volume/hana/shared.* Em vez disso, você deve escolher arquivos específicos, como global.ini do diretório **.snapshot** e de seus subdiretórios após montar novamente o volume /hana/shared para PRD. 

   No lado das operações, ocorrem as seguintes etapas:

   a. A replicação de instantâneos do volume de produção para volumes de recuperação de desastre é interrompida. Isso pode já ter acontecido se uma interrupção no local de produção for o motivo de você precisar realizar o procedimento de recuperação de desastre.
   
   b. O nome do instantâneo de armazenamento ou o instantâneo com a ID de backup escolhido é restaurado nos volumes de recuperação de desastre.
   
   c. Após a restauração, os volumes de recuperação de desastre estão disponíveis para serem montados para as unidades do SAP HANA em Instâncias Grandes na região de recuperação de desastre.
      
1. Monte os volumes de recuperação de desastre para a unidade do SAP HANA em Instâncias Grandes no site de recuperação de desastre. 
1. Inicie a instância de produção do SAP HANA inativa.
1. Se optar por copiar logs de backup de log de transações para reduzir o tempo de RPO, será necessário mesclar esses backups de log de transações no diretório /hana/logbackups da recuperação de desastre montado recentemente. Não substitua backups existentes. Copie os backups mais recentes que não foram replicados com a replicação mais recente de um instantâneo de armazenamento.
1. Também é possível restaurar arquivos únicos dos instantâneos que foram replicados para o volume /hana/shared/PRD na região do Azure de recuperação de desastre.

A próxima sequência de etapas envolve a recuperação da instância de produção SAP HANA com base no instantâneo de armazenamento restaurado e nos backups de log de transações disponíveis:

1. Altere o local de backup para **hana/logbackups** usando o SAP HANA Studio.
   ![Alterar o local de backup para recuperação de desastre](./media/hana-overview-high-availability-disaster-recovery/change_backup_location_dr1.png)

1. O SAP HANA verifica os locais do arquivo de backup e sugere o backup de log de transações mais recente para o qual ser restaurado. A verificação pode levar alguns minutos até que uma tela como a seguinte seja exibida: ![Lista de backups de log de transações para a recuperação de desastre](./media/hana-overview-high-availability-disaster-recovery/backup_list_dr2.PNG)

1. Ajuste algumas destas configurações padrão:

      - Limpe **Usar backups delta**.
      - Selecione **Inicializar área de log**.

   ![Defina o Inicializar a área de log](./media/hana-overview-high-availability-disaster-recovery/initialize_log_dr3.PNG)

1. Selecione **Concluir**.

   ![Concluir a restauração da recuperação de desastre](./media/hana-overview-high-availability-disaster-recovery/finish_dr4.PNG)

Uma janela de progresso, como mostrado aqui, deve aparecer. Tenha em mente que o exemplo é de uma restauração de recuperação de desastre de uma configuração de expansão do SAP HANA de três nós.

![Progresso da restauração](./media/hana-overview-high-availability-disaster-recovery/restore_progress_dr5.PNG)

Se a restauração parece parar de responder na tela **Concluir** e não for mostrada na tela de andamento, confirme se todas as instâncias do SAP HANA nos nós de trabalho estão em execução. Se necessário, inicie manualmente as instâncias SAP HANA.


### <a name="failback-from-a-dr-to-a-production-site"></a>Failback de recuperação de desastre para um site de produção
É possível executar failback de uma recuperação de desastre para um local de produção. Vejamos um cenário em que o failover no site de recuperação de desastre foi causado por problemas na região de produção do Azure e não pela necessidade de recuperar dados perdidos. Você estava executando sua carga de trabalho de produção SAP por um tempo no site de recuperação de desastre. Conforme são resolvidos os problemas no local de produção, convém executar failback no local de produção. Como não é possível perder dados, a etapa no local de produção envolve várias etapas e cooperação com a equipe de operações do SAP HANA no Azure. Cabe a você acionar a equipe de operações para iniciar a sincronização de volta ao site de produção, depois que os problemas forem resolvidos.

Esta é a sequência de etapas a serem seguidas:

1. A equipe de operações do SAP HANA do Azure obtém o gatilho para sincronizar os volumes de armazenamento de produção dos volumes de armazenamento de recuperação de desastre, que agora representam o estado de produção. Nesse estado, a unidade de Instância Grande do HANA no local de produção é desligada.
1. A equipe de operações do SAP HANA do Azure monitora a replicação e garante que seja obtida antes de informá-lo.
1. Desligue os aplicativos que usam a instância do HANA de produção no site de recuperação de desastre. Em seguida, faça um backup de log de transações do HANA. Depois, você interrompe a instância do HANA em execução nas unidades do SAP HANA em Instâncias Grandes no site de recuperação de desastre.
1. Após a instância do HANA em execução na unidade do SAP HANA em Instâncias Grandes no site de recuperação de desastre ser desligada, a equipe de operações sincronizará manualmente os volumes de disco novamente.
1. A equipe de operações do SAP HANA no Azure inicia a unidade da Instância Grande do HANA no local de produção novamente e a passa para você. Garanta que a instância do SAP HANA está em um estado de desligamento no tempo de inicialização da unidade do SAP HANA em Instâncias Grandes.
1. Execute as mesmas etapas de restauração de banco de dados como fez no failover para o site de recuperação de desastre anteriormente.

### <a name="monitor-disaster-recovery-replication"></a>Monitorar a replicação de recuperação de desastre

É possível monitorar o status do andamento da sua replicação de armazenamento executando o script `azure_hana_replication_status.pl`. Esse script deve ser executado a partir de uma unidade em execução no site de recuperação de desastre para funcionar conforme o esperado. O script funciona independentemente da replicação estar ativa. O script pode ser executado para cada unidade de Instância Grande HANA do seu locatário no local de recuperação de desastres. Ele não pode ser usado para obter detalhes sobre o volume de inicialização.

Chame o script com este comando:
```
./replication_status.pl <HANA SID>
```

A saída é dividida, por volume, nas seguintes seções:  

- Status do link
- Atividade de replicação atual
- Instantâneo mais recente replicado 
- Tamanho do instantâneo mais recente
- Tempo de retardo atual entre instantâneos (entre a última replicação de instantâneo concluída e agora)

O status do link será mostrado como **Ativo**, a menos se o link entre os locais estiver inativo ou houver um evento de failover em andamento no momento. A atividade de replicação resolve se dados estão sendo replicados no momento ou se estão ociosos ou se outras atividades estão ocorrendo no momento para o link. O último instantâneo replicado deve ser exibido apenas como `snapmirror…`. O tamanho do último instantâneo é exibido. Por fim, o tempo de retardo é mostrado. O tempo de retardo representa o tempo desde a replicação agendada até o momento em que replicação é concluída. Um tempo de retardo pode ser maior do que uma hora para a replicação de dados, principalmente na replicação inicial, mesmo que a replicação tenha sido iniciada. O tempo de retardo continua aumentando até que a replicação em andamento seja concluída.

A seguir, é apresentando um exemplo da saída:

```
hana_data_hm3_mnt00002_t020_dp
-------------------------------------------------
Link Status: Broken-Off
Current Replication Activity: Idle
Latest Snapshot Replicated: snapmirror.c169b434-75c0-11e6-9903-00a098a13ceb_2154095454.2017-04-21_051515
Size of Latest Snapshot Replicated: 244KB
Current Lag Time between snapshots: -   ***Less than 90 minutes is acceptable***
```












