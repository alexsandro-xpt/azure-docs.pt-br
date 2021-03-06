---
title: Mudança de host de um aplicativo de central de serviços da Contoso em Linux para o Azure e para o MySQL do Azure | Microsoft Docs
description: Saiba como a Contoso muda o host de um aplicativo Linux local migrando-o para VMs do Azure e para o MySQL do Azure.
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: 393a41016eed119305df3ca75c2ad8451216e249
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/06/2018
ms.locfileid: "43842712"
---
# <a name="contoso-migration-rehost-an-on-premises-linux-app-to-azure-vms-and-azure-mysql"></a>Migração da Contoso: mudança de host de um aplicativo Linux local para VMs do Azure e para o Azure MySQL

Este artigo mostra como a Contoso hospeda novamente o aplicativo de Central de Serviços no Linux local de duas camadas (osTicket), migrando-o para o Azure e para o MySQL no Azure.

Este documento é um de uma série de artigos que mostram como a empresa fictícia Contoso migra seus recursos locais para a nuvem do Microsoft Azure. A série inclui informações básicas e cenários que ilustram como configurar uma infraestrutura de migração e executar diferentes tipos de migrações. Cenários aumentam em complexidade. Adicionaremos mais artigos com o tempo.

**Artigo** | **Detalhes** | **Status**
--- | --- | ---
[Artigo 1: Visão geral](contoso-migration-overview.md) | Visão geral da série de artigos, da estratégia de migração da Contoso e dos aplicativos de exemplo usados na série. | Disponível
[Artigo 2: Implantar a infraestrutura do Azure](contoso-migration-infrastructure.md) | A Contoso prepara a infraestrutura local e a infraestrutura do Azure para migração. A mesma infraestrutura é usada para todos os artigos de migração da série. | Disponível
[Artigo 3: Avaliar recursos locais para migração para o Azure](contoso-migration-assessment.md)  | A Contoso executa uma avaliação de seu aplicativo do SmartHotel360 locais em execução no VMware. A Contoso avalia as VMs do aplicativo usando o serviço de Migrações para Azure e o banco de dados do SQL Server do aplicativo usando o Assistente de Migração de Dados. | Disponível
[Artigo 4: Hospedar novamente um aplicativo em uma Instância Gerenciada do Banco de Dados SQL e VM do Azure](contoso-migration-rehost-vm-sql-managed-instance.md) | A Contoso executa uma migração de elevação e troca para o Azure em seu aplicativo SmartHotel360 local. A Contoso migra a VM front-end do aplicativo usando o [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview). A Contoso migra o banco de dados do aplicativo para uma Instância Gerenciada do Banco de Dados SQL do Azure usando o [Serviço de Migração de Banco de Dados do Azure](https://docs.microsoft.com/azure/dms/dms-overview). | Disponível   
[Artigo 5: Hospedar novamente um aplicativo em VMs do Azure](contoso-migration-rehost-vm.md) | Contoso migra seus aplicativos do SmartHotel360 VMs para VMs do Azure usando o serviço Site Recovery. | Disponível
[Artigo 6: Hospedar novamente um aplicativo em VMs do Azure e em um grupo de disponibilidade AlwaysOn do SQL Server](contoso-migration-rehost-vm-sql-ag.md) | A Contoso migra o aplicativo do SmartHotel360. A Contoso usa o Site Recovery para migrar as VMs do aplicativo. Ele usa o Serviço de Migração de Banco de Dados para migrar o banco de dados do aplicativo para um cluster SQL Server protegido por um grupo de disponibilidade AlwaysOn. | Disponível 
[Artigo 7: Hospedar novamente um aplicativo do Linux nas VMs do Azure](contoso-migration-rehost-linux-vm.md) | A Contoso conclui uma migração lift-and-shift do aplicativo osTicket do Linux para as VMs do Azure, usando o Azure Site Recovery | Disponível
Artigo 8: Hospedar novamente um aplicativo Linux em VMs do Azure e no MySQL do Azure | A Contoso migra o aplicativo osTicket do Linux para as VMs do Azure usando o Azure Site Recovery e migra o banco de dados do aplicativo para uma instância do MySQL Server no Azure usando o MySQL Workbench. | Este artigo
[Artigo 9: Refatorar um aplicativo em aplicativos Web do Azure e no Banco de Dados SQL do Azure](contoso-migration-refactor-web-app-sql.md) | A Contoso migra o SmartHotel360 aplicativo para um aplicativo Web e migra o banco de dados do aplicativo para uma instância do SQL Server do Azure com o Assistente de migração de banco de dados | Disponível
[Artigo 10: Refatorar um aplicativo Linux em aplicativos Web do Azure e para o Azure MySQL](contoso-migration-refactor-linux-app-service-mysql.md) | A Contoso migra seu aplicativo osTicket do Linux para um aplicativo Web do Azure em várias regiões do Azure usando o Gerenciador de Tráfego do Azure, integrado ao GitHub para entrega contínua. A Contoso migra o banco de dados do aplicativo para uma instância do Banco de Dados do Azure para MySQL. | Disponível 
[Artigo 11: Refatorar o TFS no VSTS](contoso-migration-tfs-vsts.md) | A Contoso migra a implantação do Team Foundation Server local para o Visual Studio Team Services no Azure. | Disponível
[Artigo 12: Recriar a arquitetura de um aplicativo nos contêineres do Azure e no Banco de Dados SQL do Azure](contoso-migration-rearchitect-container-sql.md) | A Contoso migra seus aplicativos do SmartHotel360 para o Azure. Em seguida, ela cria novamente a camada da Web do aplicativo como um contêiner do Windows em execução no Azure Service Fabric e o banco de dados com o Banco de Dados SQL do Azure. | Disponível
[Artigo 13: Recompilar um aplicativo no Azure](contoso-migration-rebuild.md) | A Contoso recria seu aplicativo SmartHotel360 usando um intervalo de recursos e serviços do Azure, incluindo o Serviço de Aplicativo do Azure, o Serviço Azuis de Kubernetes (AKS), as Funções do Azure, os Serviços Cognitivos do Azure e o Banco de Dados do Azure Cosmos. | Disponível


Neste artigo, a Contoso migra um aplicativo de central de serviço (osTicket) de duas camadas Linux Apache MySQL PHP (LAMP) para o Azure. Se você quiser usar este aplicativo de software livre, poderá baixá-lo no [GitHub](https://github.com/osTicket/osTicket).



## <a name="business-drivers"></a>Geradores de negócios

A equipe de liderança de TI trabalhou em conjunto com seus parceiros comerciais para entender o que eles desejam obter:

- **Lidar com o crescimento comercial**: a Contoso está crescendo e, em função disso, há pressão sobre a infraestrutura e os sistemas locais.
- **Limitar o risco**: o aplicativo de Central de Serviços é essencial para os negócios. A Contoso quer movê-lo para o Azure com zero risco.
- **Estender**: a Contoso não quer alterar o aplicativo agora. Ela apenas quer manter o aplicativo estável.


## <a name="migration-goals"></a>Metas de migração

A equipe de nuvem Contoso fixou metas para essa migração a fi de determinar o melhor método de migração:

- Após a migração, o aplicativo no Azure deve ter os mesmos recursos de desempenho atuais em seu ambiente de VMWare local.  O aplicativo permanecerá essencial tanto na nuvem quanto localmente. 
- A Contoso não quer investir nesse aplicativo.  Ele é importante para os negócios, mas em sua forma atual, a Contoso apenas deseja passá-lo para a nuvem com segurança.
- Depois de concluir algumas migrações de aplicativo do Windows, a Contoso deseja aprender a usar uma infraestrutura baseada em Linux no Azure.
- A Contoso deseja minimizar as tarefas de administração do banco de dados depois que o aplicativo é movido para a nuvem.

## <a name="proposed-architecture"></a>Arquitetura proposta

Neste cenário:

- O aplicativo está em camadas em duas VMs (OSTICKETWEB e OSTICKETMYSQL).
- As VMs estão localizadas no host VMware ESXi **contosohost1.contoso.com** (versão 6.5).
- O ambiente VMware é gerenciado pelo vCenter Server 6.5 (**vcenter.contoso.com**) em execução em uma VM.
- A Contoso tem um datacenter local (contoso-datacenter), com um controlador de domínio local (**contosodc1**).
- O aplicativo de camada Web em OSTICKETWEB será migrado para uma VM IaaS do Azure.
- O banco de dados do aplicativo será migrado para o serviço PaaS Banco de Dados do Azure para MySQL.
- Como a Contoso está migrando uma carga de trabalho de produção, os recursos residirão no grupo de recursos de produção **ContosoRG**.
- Os recursos serão replicados na região principal (Leste dos EUA 2) e colocadas na rede de produção (VNET-PROD-EUS2):
    - A VM Web reside na sub-rede de front-end (PROD-FE-EUS2).
    - A instância do banco de dados reside na sub-rede do banco de dados (PROD-DB-EUS2).
- O banco de dados do aplicativo será migrado para o MySQL do Azure usando as ferramentas do MySQL.
- As VMs locais no datacenter da Contoso serão descomissionadas após a migração.


![Arquitetura de cenário](./media/contoso-migration-rehost-linux-vm-mysql/architecture.png) 


## <a name="migration-process"></a>Processo de migração

A Contoso concluirá o processo de migração da seguinte maneira:

Para migrar a máquina virtual Web:

1. Como uma primeira etapa, a Contoso configura o Azure e a infraestrutura local necessária para implantar o Site Recovery.
2. Depois de preparar os componentes locais e do Azure, a Contoso configura e habilita a replicação para a VM Web.
3. Quando a replicação estiver em execução, a Contoso migrará a VM fazendo failover para o Azure.

Para migrar o banco de dados:

1. A Contoso provisiona uma instância do MySQL no Azure.
2. A Contoso configura o MySQL Workbench e faz o backup do banco de dados localmente.
3. Em seguida, ela restaura o banco de dados do backup local para o Azure.

![Processo de migração](./media/contoso-migration-rehost-linux-vm-mysql/migration-process.png) 


### <a name="azure-services"></a>Serviços do Azure

**Serviço** | **Descrição** | **Custo**
--- | --- | ---
[Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/) | O serviço coordena e gerencia a migração e a recuperação de desastre para VMs do Azure e servidores físicos e de VMs locais.  | Durante a replicação para o Azure, são gerados encargos do Armazenamento do Azure.  As VMs do Azure são criadas e incorrem em encargos quando ocorre failover. [Saiba mais](https://azure.microsoft.com/pricing/details/site-recovery/) sobre encargos e preços.
[Banco de Dados do Azure para MySQL](https://docs.microsoft.com/azure/mysql/) | O banco de dados baseia-se no mecanismo do servidor MySQL de software livre. Ele fornece um banco de dados como serviço MySQL comunitário, pronto para empresas e totalmente gerenciado para o desenvolvimento e a implantação de aplicativos. 

 
## <a name="prerequisites"></a>Pré-requisitos

Veja o que a Contoso precisa para esse cenário.

**Requisitos** | **Detalhes**
--- | ---
**Assinatura do Azure** | A Contoso já criou assinaturas em um artigo anterior. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/pricing/free-trial/).<br/><br/> Se você criar uma conta gratuita, será o administrador da assinatura e poderá executar todas as ações.<br/><br/> Se você usar uma assinatura existente e não for o administrador, será necessário trabalhar com o administrador para receber permissões de Proprietário ou de Colaborador.<br/><br/> Se forem necessárias permissões mais granulares, leia [este artigo](../site-recovery/site-recovery-role-based-linked-access-control.md). 
**Infraestrutura do Azure** | A Contoso configura a infraestrutura do Azure conforme é descrito em [Infraestrutura do Azure para migração](contoso-migration-infrastructure.md).<br/><br/> Saiba mais sobre os requisitos de [rede](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#network) e [armazenamento](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#storage) específicos para o Site Recovery.
**Servidores locais** | O servidor vCenter local deve estar executando a versão 5.5, 6.0 ou 6.5<br/><br/> Um host ESXi que executa a versão 5.5, 6.0 ou 6.5<br/><br/> Uma ou mais VMs do VMware em execução no host ESXi.
**VMs locais** | [Verifique os requisitos da VM Linux](https://docs.microsoft.com//azure/site-recovery/vmware-physical-azure-support-matrix#replicated-machines) que têm suporte para migração pelo Site Recovery.<br/><br/> Verifique os [sistemas de arquivo e armazenamento Linux](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#linux-file-systemsguest-storage) que têm suporte.<br/><br/> As VMs devem atender aos [requisitos do Azure](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#azure-vm-requirements).


## <a name="scenario-steps"></a>Etapas do cenário

É assim que os administradores da Contoso concluirão a migração:

> [!div class="checklist"]
> * **Etapa 1: Preparar o Azure para recuperação de Site**: eles criar uma conta de armazenamento do Azure para armazenar os dados replicados e criará um cofre de serviços de recuperação.
> * **Etapa 2: Preparar VMware local para o Site Recovery**: eles preparam contas para a instalação de descoberta e agente VM e se preparam para se conectar a máquinas virtuais do Azure após o failover.
 * **Etapa 3: Provisionar o banco de dados]**: no Azure, eles provisionam uma instância do Banco de Dados MySQL no Azure.
> * **Etapa 4: Replicar VMs**: eles configuram o ambiente de origem e de destino do Site Recovery, definem uma política de replicação e começam a replicar as VMs para o armazenamento do Azure.
> * **Etapa 5: Migrar o banco de dados**: eles configuram a migração com as ferramentas do MySQL.
> * **Etapa 6: Migrar as VMs com o Site Recovery**: por fim, eles executam um failover de teste para verificar se tudo está funcionando e, em seguida, executam um failover completo para migrar as VMs para o Azure.




## <a name="step-1-prepare-azure-for-the-site-recovery-service"></a>Etapa 1: Preparar o Azure para o serviço de recuperação de Site

A Contoso precisa de alguns componentes do Azure para o Site Recovery:

- Uma VNet na qual os recursos com failover estão localizados. A Contoso já criou a VNET durante a [implantação da infraestrutura do Azure](contoso-migration-infrastructure.md)
- Uma nova conta de armazenamento do Azure para armazenar dados replicados. 
- Um cofre dos Serviços de Recuperação no Azure.

Os administradores da Contoso criam uma conta de armazenamento e um cofre da seguinte maneira:

1. Eles criam uma conta de armazenamento do Azure (**contosovmsacc20180528**) na região Leste dos EUA 2.

    - A conta de armazenamento deve estar na mesma região do cofre de Serviços de Recuperação.
    - Trata-se de uma conta de uso geral, com armazenamento padrão e replicação de LRS.

    ![Armazenamento do Site Recovery](./media/contoso-migration-rehost-linux-vm-mysql/asr-storage.png)

3. Com a rede e a conta de armazenamento prontas, eles criam um cofre (ContosoMigrationVault) e colocam-no grupo de recursos **ContosoFailoverRG**, na região Leste dos EUA 2 primária.

    ![Cofre dos Serviços de Recuperação](./media/contoso-migration-rehost-linux-vm-mysql/asr-vault.png)

**Precisa de mais ajuda?**

[Saiba mais sobre](https://docs.microsoft.com/azure/site-recovery/tutorial-prepare-azure) como configurar o Azure para o Site Recovery.


## <a name="step-2-prepare-on-premises-vmware-for-site-recovery"></a>Etapa 2: Preparar VMware local para o Site Recovery

A Contoso administra a infraestrutura de VMware local da seguinte maneira:

- Eles criaram uma conta no vCenter Server para automatizar a descoberta de VM.
- Eles criam uma conta que permite a instalação automática do serviço de mobilidade nas VMs do VMware que serão replicadas.
- Eles preparam as VMs locais, para que eles possam se conectar às VMs do Azure quando forem criadas após a migração.


### <a name="prepare-an-account-for-automatic-discovery"></a>Preparar uma conta para a descoberta automática

O Site Recovery precisa de acesso aos servidores VMware para:

- Descobrir VMs automaticamente. Pelo menos uma conta somente leitura é necessária.
- Orquestrar a replicação, o failover e o failback. É necessária uma conta que possa executar operações como criação e remoção de discos e ativação de VMs.

Os administradores da Contoso configuram a conta da seguinte maneira:

1. Eles criam uma função no nível do vCenter.
2. Em seguida, eles atribuem a essa função as permissões necessárias.


### <a name="prepare-an-account-for-mobility-service-installation"></a>Preparar uma conta para a instalação do serviço de Mobilidade

O serviço de mobilidade deve ser instalado em cada VM que a Contoso deseja migrar.

- O Site Recovery poderá fazer uma instalação automática por push desse componente quando você habilitar a replicação das VMs.
- Para instalação automática. O Site Recovery precisa de uma conta com permissões para acessar a VM. 
- Os detalhes da conta são inseridos durante a instalação da replicação. 
- A conta pode ser de domínio ou local, desde que tenha permissões de instalação.


### <a name="prepare-to-connect-to-azure-vms-after-failover"></a>Preparar para conectar VMs do Azure após o failover

Após o failover para o Azure, a Contoso deseja poder se conectar às VMs do Azure. Para isso, os administradores da Contoso precisam fazer o seguinte:

- Para ter acesso à Internet, ela habilita SSH na VM Linux local antes da migração.  No cado do Ubuntu, isso pode ser feito usando o seguinte comando: **Sudo apt-get ssh install -y**.
- Após o failover, ela deve verificar os **Diagnósticos de inicialização** para exibir uma captura de tela da VM.
- Se isso não funcionar, ela precisa verificar se a VM está em execução e examinar as [dicas de solução de problemas](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).

**Precisa de mais ajuda?**

- [Saiba mais sobre](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-automatic-discovery) como criar e atribuir uma função para a descoberta automática.
- [Saiba mais sobre](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-mobility-service-installation) como criar uma conta para a instalação do serviço de mobilidade por push.


## <a name="step-3-provision-azure-database-for-mysql"></a>Etapa 3: Provisionar o Banco de Dados do Azure para MySQL

Os administradores da Contoso provisionam uma instância do banco de dados MySQL na região Leste dos EUA 2 primária.

1. No portal do Azure, ela cria um recurso de Banco de Dados do Azure para MySQL. 

    ![MySQL](./media/contoso-migration-rehost-linux-vm-mysql/mysql-1.png)

2. Ela adiciona o nome **contosoosticket** ao banco de dados do Azure. Ela adiciona o banco de dados ao grupo de recursos de produção **ContosoRG** e especifica credenciais para ele.
3. O banco de dados MySQL local é a versão 5.7 e, portanto, ela escolhe essa versão por questões de compatibilidade. Ela usa os tamanhos padrão que correspondem aos requisitos de banco de dados.

     ![MySQL](./media/contoso-migration-rehost-linux-vm-mysql/mysql-2.png)

4. Nas **Opções de Redundância de Backup**, eles escolhem usar **Com Redundância Geográfica**. Essa opção permite restaurar o banco de dados na região secundária Centro dos EUA em caso de interrupção. Ela só pode configurar essa opção quando provisiona o banco de dados.

     ![Redundância](./media/contoso-migration-rehost-linux-vm-mysql/db-redundancy.png)

4. Na rede **VNET-PROD-EUS2** > **Pontos de extremidade de serviço**, ela adiciona um ponto de extremidade de serviço (uma sub-rede do banco de dados) ao serviço SQL.

    ![MySQL](./media/contoso-migration-rehost-linux-vm-mysql/mysql-3.png)

5. Depois de adicionar a sub-rede, ela cria uma regra de rede virtual que permite o acesso da sub-rede do banco de dados na rede de produção.

    ![MySQL](./media/contoso-migration-rehost-linux-vm-mysql/mysql-4.png)


## <a name="step-4-replicate-the-on-premises-vms"></a>Etapa 4: Replicar VMs locais

Antes de poder migrar a VM Web para o Azure, os administradores da Contoso configuram e habilitam a replicação.

### <a name="set-a-protection-goal"></a>Definir uma meta de proteção

1. No cofre, sob o nome do cofre (ContosoVMVault), ela define uma meta de replicação (**Introdução** > **Site Recovery** > **Preparar infraestrutura**.
2. Ela especifica que suas máquinas são locais, que são VMs VMware e que ela deseja replicar no Azure.

    ![Meta de replicação](./media/contoso-migration-rehost-linux-vm-mysql/replication-goal.png)

### <a name="confirm-deployment-planning"></a>Confirmar planejamento de implantação

Para continuar, ela confirma que concluiu o planejamento da implantação selecionando **Sim, fiz isso**. A Contoso só está migrando uma VM neste cenário e não precisa de planejamento de implantação.

### <a name="set-up-the-source-environment"></a>Configurar o ambiente de origem

Agora, os administradores da Contoso configuram o ambiente de origem. Para fazer isso, usando um modelo OVF, ela implanta um servidor de configuração do Site Recovery como uma VM VMware local altamente disponível. Depois que a configuração do servidor tiver sido finalizada, registre-o no cofre.

O servidor de configuração executa vários componentes:

- O componente de servidor de configuração que coordena a comunicação entre o ambiente local e o Azure e gerencia a replicação de dados.
- Servidor em processo que age como um gateway de replicação. Ele recebe dados de replicação, otimiza-os com caching, compactação e criptografia e os envia para o Armazenamento do Azure.
- O servidor de processo também instala o Serviço de Mobilidade nas VMs que você deseja replicar e executa a descoberta automática de VMs VMware locais.

Os administradores da Contoso fazem isso da seguinte maneira:


1. Ela baixam o modelo OVF de **Preparar infraestrutura** > **Fonte** > **Servidor de Configuração**.
    
    ![Baixe o OVF](./media/contoso-migration-rehost-linux-vm-mysql/add-cs.png)

2. Ela importa o modelo para o VMware a fim de criar e implantar a VM.

    ![Modelo de OVF](./media/contoso-migration-rehost-linux-vm-mysql/vcenter-wizard.png)

3. Ao ligar a máquina virtual pela primeira vez, ela é inicializada como uma experiência de instalação do Windows Server 2016. Eles aceitam o contrato de licença e inserem uma senha de administrador.
4. Após a conclusão da instalação, entram na VM como administrador. No primeiro logon, a Ferramenta de Configuração do Azure Site Recovery é iniciada por padrão.
5. Na ferramenta, ela especifica um nome a ser usado para registrar o servidor de configuração no cofre.
6. A ferramenta verifica se a VM pode se conectar ao Azure.
7. Depois que a conexão é estabelecida, entram na assinatura do Azure. As credenciais devem ter acesso ao cofre no qual você deseja registrar o servidor de configuração.

    ![Registrar servidor de configuração](./media/contoso-migration-rehost-linux-vm-mysql/config-server-register2.png)

8. A ferramenta executa algumas tarefas de configuração e, em seguida, é reinicializada.
9. Entram na máquina novamente e o Assistente de Gerenciamento do Servidor de Configuração é iniciado automaticamente.
10. No assistente, selecionam o NIC que vai receber o tráfego de replicação. Essa configuração não pode ser alterada depois de definida.
11. Ela seleciona a assinatura, o grupo de recursos e o cofre no qual registrar o servidor de configuração.

    ![cofre](./media/contoso-migration-rehost-linux-vm-mysql/cswiz1.png) 

12. Agora, ela baixa e instala o servidor MySQL e o VMWare PowerCLI. 
13. Após a validação, ela especifica o FQDN ou endereço IP do host do servidor vSphere ou vCenter. Ela deixa a porta padrão e especifica um nome amigável para o servidor vCenter.
14. Ela insere a conta criada para descoberta automática e as credenciais que o Site Recovery usará para instalar o serviço de mobilidade automaticamente. 

    ![vCenter](./media/contoso-migration-rehost-linux-vm-mysql/cswiz2.png)

14. Após a conclusão do registro, no portal do Azure, eles verificam se o servidor de configuração e o VMware Server estão listados na página **Origem** no cofre. A descoberta pode levar 15 minutos ou mais. 
15. Com tudo pronto, o Site Recovery se conecta a servidores VMware e descobre VMs.

### <a name="set-up-the-target"></a>Configurar o destino

Agora, os administradores da Contoso inserem as configurações de replicação do destino.

1. Em **Preparar infraestrutura** > **Destino**, selecionam as configurações de destino.
2. O Site Recovery verifica se há uma rede e uma conta de armazenamento do Azure no destino especificado.


### <a name="create-a-replication-policy"></a>Criar uma política de replicação

Com a origem e o destino configurados, os administradores da Contoso estão prontos para criar uma política de replicação.

1. Em **Preparar infraestrutura** > **Configurações de Replicação** > **Política de Replicação** >  **Criar e Associar**, criam uma política **ContosoMigrationPolicy**.
2. Usam as configurações padrão:
    - **Limite de RPO**: padrão de 60 minutos. Esse valor define a frequência em que são criados os pontos de recuperação. Um alerta será gerado se a replicação contínua exceder esse limite.
    - **Retenção do ponto de recuperação**. Padrão de 24 horas. Esse valor especifica qual é o tempo da janela de retenção para cada ponto de recuperação. VMs replicadas podem ser recuperadas para qualquer ponto em uma janela.
    - **Frequência de instantâneos consistentes com o aplicativo**. O padrão é de uma hora. Esse valor especifica a frequência com que são criados instantâneos consistentes com o aplicativo.
 
        ![Criar política de replicação](./media/contoso-migration-rehost-linux-vm-mysql/replication-policy.png)

5. A política é associada automaticamente ao servidor de configuração. 

    ![Associar política de replicação](./media/contoso-migration-rehost-linux-vm-mysql/replication-policy2.png)


**Precisa de mais ajuda?**

- Você pode ler uma explicação completa de todas essas etapas em [Configurar a recuperação de desastres para máquinas virtuais do VMware locais](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial).
- As instruções detalhadas estão disponíveis para ajudá-lo a [configurar o ambiente da fonte](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-source), [implantar o servidor de configuração](https://docs.microsoft.com/azure/site-recovery/vmware-azure-deploy-configuration-server), e [definir configurações de replicação](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-replication).
- [Saiba mais](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux) sobre o agente convidado do Azure para Linux.

### <a name="enable-replication-for-the-web-vm"></a>Habilitar replicação para a VM Web

Agora os administradores da Contoso podem iniciar a replicação da VM **OSTICKETWEB**.

1. Em **Replicar aplicativo** > **Fonte** > **+Replicar**, ela seleciona as configurações de fonte.
2. Ela indica que deseja habilitar as máquinas virtuais e seleciona as configurações de fonte, incluindo o servidor vCenter, e o servidor de configuração.

    ![Habilitar a replicação](./media/contoso-migration-rehost-linux-vm-mysql/enable-replication-source.png)

3. Agora, ela especifica as configurações de destino. Isso inclui o grupo de recursos e a rede em que a VM do Azure ficará após o failover e a conta de armazenamento na qual serão armazenados os dados replicados. 

     ![Habilitar a replicação](./media/contoso-migration-rehost-linux-vm-mysql/enable-replication2.png)

3. Eles selecionam **OSTICKETWEB** para replicação. 

    ![Habilitar a replicação](./media/contoso-migration-rehost-linux-vm-mysql/enable-replication3.png)

4. Nas propriedades da VM, ela seleciona a conta que deve ser usada para instalar o serviço de mobilidade na VM automaticamente.

     ![Serviço de mobilidade](./media/contoso-migration-rehost-linux-vm-mysql/linux-mobility.png)

5. Em **Configurações de Replicação** > **Definir Configurações de Replicação**, ela verifica se a política de replicação correta está aplicada e seleciona **Habilitar Replicação**. O serviço de mobilidade será instalado automaticamente.
6.  Eles rastreiam o progresso da replicação em **Trabalhos**. Após o trabalho de **Finalizar Proteção** ser executado, o computador estará pronto para failover.


**Precisa de mais ajuda?**

Você pode ler um passo a passo de todas essas etapas em [Habilitar replicação](https://docs.microsoft.com/azure/site-recovery/vmware-azure-enable-replication).


## <a name="step-5-migrate-the-database"></a>Etapa 5: Migrar o banco de dados

Os administradores da Contoso migram o banco de dados usando backup e restauração, com as ferramentas do MySQL. Ela instala o Workbench do MySQL, faz backup do banco de dados de OSTICKETMYSQL e, em seguida, o restaura no servidor do Banco de Dados do Azure para MySQL.

### <a name="install-mysql-workbench"></a>Instalar o MySQL Workbench

1. Eles verificam os [pré-requisitos e downloads do MySQL Workbench](https://dev.mysql.com/downloads/workbench/?utm_source=tuicool).
2. Ela instala o Workbench do MySQL para Windows de acordo com as [instruções de instalação](https://dev.mysql.com/doc/workbench/en/wb-installing.html).
3. No Workbench do MySQL, ela cria uma conexão MySQL para OSTICKETMYSQL. 

    ![Workbench do MySQL](./media/contoso-migration-rehost-linux-vm-mysql/workbench1.png)

4. Ela exporta o banco de dados como **osticket** para um arquivo autossuficiente local.

    ![Workbench do MySQL](./media/contoso-migration-rehost-linux-vm-mysql/workbench2.png)

5. Depois que o backup do banco de dados foi feito localmente, ela cria uma conexão com a instância do Banco de Dados do Azure para MySQL.

    ![Workbench do MySQL](./media/contoso-migration-rehost-linux-vm-mysql/workbench3.png)

6. Agora, eles podem importar (restaurar) o banco de dados na instância do MySQL no Azure usando o arquivo autossuficiente. Um novo esquema (osticket) é criado para a instância.

    ![Workbench do MySQL](./media/contoso-migration-rehost-linux-vm-mysql/workbench4.png)

## <a name="step-6-migrate-the-vms-with-site-recovery"></a>Etapa 6: Migrar VMs com o Site Recovery

Por fim, os administradores da Contoso executam um rápido failover de teste e, em seguida, migram a VM.

### <a name="run-a-test-failover"></a>Execute um teste de failover

A execução de um failover de teste ajuda a verificar se tudo está funcionando como esperado antes da migração. 

1. Eles executam um failover de teste, até o último momento disponível (**Último processamento**).
2. Selecionam **Desligar o computador antes do início do failover**, para que o Site Recovery tente desligar a VM da fonte antes de disparar o failover. O failover continuará mesmo o desligamento falhar. 
3. O failover de teste é executado: 

    - Uma verificação de pré-requisitos será executada para conferir se todas as condições exigidas para a migração foram cumpridas.
    - O failover processa os dados para que uma VM do Azure possa ser criada. Se o ponto de recuperação mais recente for selecionado, um ponto de recuperação será criado nos dados.
    - Uma VM do Azure é criada usando os dados processados na etapa anterior.

3. Após o failover ser concluído, a réplica da VM do Azure aparecerá no portal do Azure. Eles Verifique se a VM é o tamanho apropriado, o que ele está conectado à rede certa, e se ele está em execução. 
4. Depois da verificação, ela limpa o failover, registra e salva as observações.

### <a name="migrate-the-vm"></a>Migração da VM

Para migrar a VM, os administradores da Contoso criam um plano de recuperação que inclui a VM e fazem failover do plano para o Azure.

1. Eles criam um plano e adicionam **OSTICKETWEB** a ele.

    ![Plano de recuperação](./media/contoso-migration-rehost-linux-vm-mysql/recovery-plan.png)

2. Ela executa um failover no plano. Ela seleciona o último ponto de recuperação e especifica que o Site Recovery deve tentar desligar a VM local antes de disparar o failover. Podem acompanhar o progresso do failover na página **Trabalhos**.

    ![Failover](./media/contoso-migration-rehost-linux-vm-mysql/failover1.png)

3. Durante o failover, o vCenter Server emite comandos para parar as duas VMs em execução no host ESXi.

    ![Failover](./media/contoso-migration-rehost-linux-vm-mysql/vcenter-failover.png)

4. Após o failover, eles verificam se a VM do Azure aparece conforme o esperado no portal do Azure.

    ![Failover](./media/contoso-migration-rehost-linux-vm-mysql/failover2.png)  

5. Depois de verificar a VM, ela conclui a migração. Isso interrompe a replicação para a VM e interrompe a cobrança do Site Recovery para a VM.

    ![Failover](./media/contoso-migration-rehost-linux-vm-mysql/failover3.png)

**Precisa de mais ajuda?**

- [Saiba mais](https://docs.microsoft.com/azure/site-recovery/tutorial-dr-drill-azure) sobre como executar failovers de teste. 
- [Saiba](https://docs.microsoft.com/azure/site-recovery/site-recovery-create-recovery-plans) como criar um plano de recuperação.
- [Saiba mais sobre](https://docs.microsoft.com/azure/site-recovery/site-recovery-failover) como fazer failover para o Azure.


### <a name="connect-the-vm-to-the-database"></a>Conectar a VM ao banco de dados

Como etapa final no processo de migração, os administradores da Contoso atualizam a cadeia de conexão do aplicativo para apontar para o Banco de Dados do Azure para MySQL. 

1. Ela faz uma conexão SSH com a VM OSTICKETWEB usando Putty ou outro cliente SSH. A VM é particular e, portanto, ela se conecta usando o endereço IP privado.

    ![Conectar ao banco de dados](./media/contoso-migration-rehost-linux-vm-mysql/db-connect.png)  

    ![Conectar ao banco de dados](./media/contoso-migration-rehost-linux-vm-mysql/db-connect2.png)  

2. Ela atualiza as configurações para que a VM **OSTICKETWEB** possa se comunicar com o banco de dados **OSTICKETMYSQL**. Atualmente, a configuração está codificada com o endereço IP local 172.16.0.43.

    **Antes da atualização**
    
    ![Atualizar IP](./media/contoso-migration-rehost-linux-vm-mysql/update-ip1.png)  

    **Após a atualização**
    
    ![Atualizar IP](./media/contoso-migration-rehost-linux-vm-mysql/update-ip2.png) 
    
    ![Atualizar IP](./media/contoso-migration-rehost-linux-vm-mysql/update-ip3.png) 

3. Ela reinicia o serviço com **systemctl restart apache2**.

    ![Reiniciar](./media/contoso-migration-rehost-linux-vm-mysql/restart.png) 

4. Por fim, ela atualiza os registros DNS para **OSTICKETWEB** em um dos controladores de domínio da Contoso.

    ![Atualizar o DNS](./media/contoso-migration-rehost-linux-vm-mysql/update-dns.png) 


##  <a name="clean-up-after-migration"></a>Limpar após a migração

Com a migração concluída, as camadas do aplicativo osTicket estão funcionando em VMs do Azure.

Agora, a Contoso precisa fazer o seguinte: 
- Remover VMs VMware do inventário do vCenter
- Remover as VMs locais de trabalhos de backup locais.
- Atualizar a documentação interna para mostrar novos locais e endereços IP. 
- Examinar todos os recursos que interagem com as VMs locais e atualizar as configurações ou documentação relevantes para refletir a nova configuração.
- A Contoso usou o serviço Migrações para Azure com mapeamento de dependências a fim de avaliar a VM **OSTICKETWEB** para migração. Agora, ela deve remover da VM os agentes (Microsoft Monitoring Agent/agente de dependência) instalados para essa finalidade.

## <a name="review-the-deployment"></a>Revisar a implantação

Com o aplicativo em execução, a Contoso precisa operacionalizar e proteger totalmente sua infraestrutura de novo.

### <a name="security"></a>Segurança

A equipe de segurança da Contoso examina a VM e o banco de dados para determinar os problemas de segurança.

- Ela revisa os NSGs (grupos de segurança de rede) da VM para controlar o acesso. Os NSGs são usados para garantir que somente o tráfego permitido para o aplicativo pode passar.
- Ela pensa em proteger os dados nos discos de VM usando criptografia de disco e o Azure Key Vault.
- A comunicação entre a VM e a instância do banco de dados não está configurada para SSL. Ela precisa fazer isso para fazer com que o tráfego de banco de dados não possa ser atacado.

[Leia mais](https://docs.microsoft.com/azure/security/azure-security-best-practices-vms#vm-authentication-and-access-control) sobre as práticas recomendadas de segurança para máquinas virtuais.

### <a name="bcdr"></a>BCDR

Para continuidade dos negócios e recuperação de desastres, a Contoso executa as seguintes ações:

- **Manter os dados seguros**: a Contoso faz backup dos dados na VM do aplicativo usando o serviço de Backup do Azure. [Saiba mais](https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Eles não precisam configurar o backup do banco de dados. O Banco de Dados do Azure para MySQL cria e armazena backups do servidor. Ela escolheu usar a redundância geográfica para o banco de dados e, portanto, ele é resiliente e está pronto para a produção.
- **Mantém os aplicativos em funcionamento**: a Contoso replica as VMs do aplicativo no Azure para uma região secundária usando o Site Recovery. [Saiba mais](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-quickstart).


### <a name="licensing-and-cost-optimization"></a>Licenciamento e otimização de custo

- Após a implantação de recursos, a Contoso atribui marcas do Azure conforme as decisões tomadas durante a implantação da [infraestrutura do Azure](contoso-migration-infrastructure.md#set-up-tagging).
- Não há nenhum problema de licenciamento nos servidores Ubuntu da Contoso.
- A Contoso habilitará o Gerenciamento de Custos do Azure licenciado pela Cloudyn, uma subsidiária da Microsoft. É uma solução de gerenciamento de custo de multi-nuvem que ajuda você a usar e gerenciar o Azure e outros recursos de nuvem.  [Saiba mais](https://docs.microsoft.com/azure/cost-management/overview) sobre o Gerenciamento de Custos do Azure.


## <a name="next-steps"></a>Próximas etapas

Neste momento, mostramos o cenário final de nova hospedagem. A Contoso migrou a VM de front-end do aplicativo osTicket no Linux local para uma VM do Azure e migrou o banco de dados do aplicativo para uma instância do MySQL no Azure.

No próximo conjunto de tutoriais na série de migrações, vamos mostrar como a Contoso executou um conjunto mais complexo de migrações, envolvendo a refatoração de aplicativo, em vez de migrações lift-and-shift simples.
