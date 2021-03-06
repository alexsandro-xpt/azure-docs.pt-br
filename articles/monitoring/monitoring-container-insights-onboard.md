---
title: Como carregar o Azure Monitor para contêineres | Microsoft Docs
description: Este artigo descreve como carregar e configurar o Azure Monitor para contêineres para que você possa entender o desempenho do seu contêiner e quais problemas relacionados ao desempenho foram identificados.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/18/2018
ms.author: magoedte
ms.openlocfilehash: 6b1de03814017ca4eb811362b2aaf5ac24f5ea86
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/24/2018
ms.locfileid: "46999793"
---
# <a name="how-to-onboard-azure-monitor-for-containers"></a>Como carregar o Azure Monitor para contêineres
Este artigo descreve como configurar o Azure Monitor para contêineres para monitorar o desempenho das cargas de trabalho implantadas em ambientes do Kubernetes e hospedadas no [Serviço de Kubernetes do Azure](https://docs.microsoft.com/azure/aks/).

## <a name="prerequisites"></a>Pré-requisitos 
Antes de começar, verifique se você tem o seguinte:

- Um cluster AKS novo ou existente.
- Um agente do Log Analytics em contêineres para a versão do Linux microsoft / oms:ciprod04202018 ou posterior. O número de versão é representado por uma data no formato a seguir: *mmddaaaa*. Esse agente é instalado automaticamente durante a integração desse recurso. 
- Um espaço de trabalho do Log Analytics. É possível criá-lo ao habilitar o monitoramento do novo cluster do AKS ou permitir que a experiência de integração crie um espaço de trabalho padrão no grupo de recursos padrão da assinatura de cluster do AKS. Se optar por criá-lo por conta própria, será possível criá-lo por meio do [Azure Resource Manager](../log-analytics/log-analytics-template-workspace-configuration.md), do [PowerShell](https://docs.microsoft.com/azure/log-analytics/scripts/log-analytics-powershell-sample-create-workspace?toc=%2fpowershell%2fmodule%2ftoc.json) ou no [portal do Azure](../log-analytics/log-analytics-quick-create-workspace.md).
- A função de colaborador do Log Analytics, para habilitar o monitoramento de contêiner. Para obter mais informações sobre como controlar o acesso a um espaço de trabalho do Log Analytics, veja [Gerenciar espaços de trabalho](../log-analytics/log-analytics-manage-access.md).

[!INCLUDE [log-analytics-agent-note](../../includes/log-analytics-agent-note.md)]

## <a name="components"></a>Componentes 

Sua capacidade de monitorar o desempenho depende de um agente de Log Analytics contido no container para Linux, que coleta dados de desempenho e de eventos de todos os nós no cluster. O agente é automaticamente implantado e registrado no espaço de trabalho do Log Analytics especificado depois que o monitoramento de contêiner é habilitado. 

>[!NOTE] 
>Se você já implantou um cluster do AKS, habilite o monitoramento usando a CLI do Azure ou um modelo do Azure Resource Manager fornecido, conforme demonstrado posteriormente neste artigo. Não é possível usar `kubectl` para atualizar, excluir, implantar ou reimplantar o agente. 
>

## <a name="sign-in-to-the-azure-portal"></a>Entre no Portal do Azure
Entre no [Portal do Azure](https://portal.azure.com). 

## <a name="enable-monitoring-for-a-new-cluster"></a>Habilitar o monitoramento para um novo cluster
Durante a implantação, você poderá habilitar o monitoramento de um novo cluster do AKS no portal do Azure ou com a CLI do Azure. Siga as etapas no artigo de início rápido [Implantar um cluster do AKS (Serviço de Kubernetes do Azure)](../aks/kubernetes-walkthrough-portal.md), se quiser habilitar a partir do portal. Na página **Monitoramento**, para a opção **Habilitar Monitoramento**, selecione **Sim** e, em seguida, selecione um espaço de trabalho do Log Analytics existente ou crie um novo. 

Para habilitar o monitoramento de um novo cluster do AKS criado com a CLI do Azure, siga a etapa no artigo de início rápido na seção [Criar cluster do AKS](../aks/kubernetes-walkthrough.md#create-aks-cluster).  

>[!NOTE]
>Se você optar por usar a CLI do Azure, primeiro precisará instalar e usar a CLI localmente. Você deve estar executando a CLI do Azure versão 2.0.43 ou posterior. Para identificar sua versão, execute `az --version`. Caso precise instalar ou atualizar a CLI do Azure, veja [Instalar a CLI do Azure](https://docs.microsoft.com/cli/azure/install-azure-cli). 
>

Depois de habilitar o monitoramento e todas as tarefas de configuração terem sido concluídas com sucesso, você poderá monitorar o desempenho do cluster de uma destas duas maneiras:

* Diretamente no cluster do AKS selecionando **Integridade** no painel esquerdo.
* Ao selecionar o bloco **Monitorar insights do contêiner** na página de cluster do AKS do cluster selecionado. No Azure Monitor, no painel esquerdo, selecione **Integridade**. 

  ![Opções para selecionar o Azure Monitor para contêineres no AKS](./media/monitoring-container-insights-onboard/kubernetes-select-monitoring-01.png)

Depois de habilitar o monitoramento, poderão ser necessários cerca de 15 minutos antes de exibir as métricas de integridade para o cluster. 

## <a name="enable-monitoring-for-existing-managed-clusters"></a>Habilitar monitoramento para clusters gerenciados existentes
É possível habilitar o monitoramento de um cluster do AKS que já tenha sido implantado usando a CLI do Azure, a partir do portal ou com o modelo do Azure Resource Manager fornecido, usando o cmdlet `New-AzureRmResourceGroupDeployment` do PowerShell. 

### <a name="enable-monitoring-using-azure-cli"></a>Habilitar o monitoramento usando a CLI do Azure
A etapa a seguir permite o monitoramento do cluster do AKS usando a CLI do Azure. Neste exemplo, não é necessário criar ou especificar um espaço de trabalho existente. Esse comando simplificará o processo, criando um espaço de trabalho padrão no grupo de recursos padrão da assinatura do cluster do AKS, se ainda não existir um na região.  O espaço de trabalho padrão criado é semelhante ao formato do *DefaultWorkspace-<GUID>-<Region>*.  

```azurecli
az aks enable-addons -a monitoring -n MyExistingManagedCluster -g MyExistingManagedClusterRG  
```

A saída será semelhante à seguinte:

```azurecli
provisioningState       : Succeeded
```

### <a name="enable-monitoring-from-azure-monitor"></a>Habilitar o monitoramento do Azure Monitor
Para habilitar o monitoramento do seu contêiner AKS no portal do Azure por meio do Azure Monitor, faça o seguinte:

1. No portal do Azure, selecione **Monitor**. 
2. Selecione **Contêineres (versão prévia)** na lista.
3. Na página **Monitor – contêineres (versão prévia)**, selecione **Clusters não monitorados**.
4. Na lista de clusters não monitorados, localize o contêiner na lista e clique em **Habilitar**.   
5. Na página **Integração para logs e integridade do contêiner**, se você tiver um espaço de trabalho do Log Analytics existente na mesma assinatura do cluster, selecione-o na lista suspensa.  
    A lista seleciona previamente o espaço de trabalho e o local padrão no qual o contêiner do AKS está implantado na assinatura. 

    ![Habilitar o monitoramento de insights do contêiner do AKS](./media/monitoring-container-insights-onboard/kubernetes-onboard-brownfield-01.png)

    >[!NOTE]
    >Se você quiser criar um novo espaço de trabalho do Log Analytics para armazenar os dados de monitoramento do cluster, siga as instruções em [Criar um espaço de trabalho do Log Analytics](../log-analytics/log-analytics-quick-create-workspace.md). Crie o espaço de trabalho na mesma assinatura em que o contêiner AKS é implantado. 
 
Depois de habilitar o monitoramento, poderão ser necessários cerca de 15 minutos antes de exibir as métricas de integridade para o cluster. 

### <a name="enable-monitoring-from-aks-cluster-in-the-portal"></a>Habilitar o monitoramento do cluster do AKS no portal
Para habilitar o monitoramento do seu contêiner AKS no portal do Azure, faça o seguinte:

1. No portal do Azure, clique em **Todos os serviços**. 
2. Na lista de recursos, comece digitando **Contêineres**.  
    A lista filtra com base na sua entrada. 
3. Selecione **Serviços de Kubernetes**.  

    ![O link de serviços do Kubernetes](./media/monitoring-container-insights-onboard/portal-search-containers-01.png)

4. Na lista de contêineres, selecione um contêiner.
5. Na página de visão geral do contêiner, selecione **Monitorar integridade do contêiner**.  
6. Na página **Integração para Logs e Integridade do Contêiner**, se você tiver um espaço de trabalho do Log Analytics existente na mesma assinatura que o cluster, selecione-o na lista suspensa.  
    A lista seleciona previamente o espaço de trabalho e o local padrão no qual o contêiner do AKS está implantado na assinatura. 

    ![Habilitar o monitoramento de integridade do contêiner do AKS](./media/monitoring-container-insights-onboard/kubernetes-onboard-brownfield-02.png)

    >[!NOTE]
    >Se você quiser criar um novo espaço de trabalho do Log Analytics para armazenar os dados de monitoramento do cluster, siga as instruções em [Criar um espaço de trabalho do Log Analytics](../log-analytics/log-analytics-quick-create-workspace.md). Crie o espaço de trabalho na mesma assinatura em que o contêiner AKS é implantado. 
 
Depois de habilitar o monitoramento, poderão ser necessários cerca de 15 minutos antes de exibir dados operacionais para o cluster. 

### <a name="enable-monitoring-by-using-an-azure-resource-manager-template"></a>Habilitar o monitoramento usando um modelo do Azure Resource Manager
Esse método inclui dois modelos JSON. Um modelo especifica a configuração para habilitar o monitoramento e o outro contém valores de parâmetro que você configura para especificar o seguinte:

* A ID de recurso do contêiner do AKS. 
* O grupo de recursos no qual o cluster está implantado.
* O espaço de trabalho do Log Analytics e a região na qual criar o espaço de trabalho. 

O espaço de trabalho do Log Analytics deve ser criado manualmente. Para criar o espaço de trabalho, você pode configurá-lo por meio do [Azure Resource Manager](../log-analytics/log-analytics-template-workspace-configuration.md), por meio do [PowerShell](https://docs.microsoft.com/azure/log-analytics/scripts/log-analytics-powershell-sample-create-workspace?toc=%2fpowershell%2fmodule%2ftoc.json) ou no [portal do Azure](../log-analytics/log-analytics-quick-create-workspace.md).

Se você não estiver familiarizado com o conceito de implantação de recursos usando um modelo, veja:
* [Implantar recursos com modelos do Resource Manager e o Azure PowerShell](../azure-resource-manager/resource-group-template-deploy.md)
* [Implantar recursos com modelos do Resource Manager e a CLI do Azure](../azure-resource-manager/resource-group-template-deploy-cli.md)

Se você optar por usar a CLI do Azure, primeiro precisará instalar e usar a CLI localmente. Você deve estar executando a CLI do Azure versão 2.0.27 ou posterior. Para identificar sua versão, execute `az --version`. Caso precise instalar ou atualizar a CLI do Azure, veja [Instalar a CLI do Azure](https://docs.microsoft.com/cli/azure/install-azure-cli). 

#### <a name="create-and-execute-a-template"></a>Criar e executar um modelo

1. Copie e cole a seguinte sintaxe JSON em seu arquivo:

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "aksResourceId": {
        "type": "string",
        "metadata": {
           "description": "AKS Cluster Resource ID"
           }
    },
    "aksResourceLocation": {
    "type": "string",
     "metadata": {
        "description": "Location of the AKS resource e.g. \"East US\""
       }
    },
    "workspaceResourceId": {
      "type": "string",
      "metadata": {
         "description": "Azure Monitor Log Analytics Resource ID"
       }
    },
    "workspaceRegion": {
    "type": "string",
    "metadata": {
       "description": "Azure Monitor Log Analytics workspace region"
      }
     }
    },
    "resources": [
      {
    "name": "[split(parameters('aksResourceId'),'/')[8]]",
    "type": "Microsoft.ContainerService/managedClusters",
    "location": "[parameters('aksResourceLocation')]",
    "apiVersion": "2018-03-31",
    "properties": {
      "mode": "Incremental",
      "id": "[parameters('aksResourceId')]",
      "addonProfiles": {
        "omsagent": {
          "enabled": true,
          "config": {
            "logAnalyticsWorkspaceResourceID": "[parameters('workspaceResourceId')]"
          }
         }
       }
      }
     },
    {
        "type": "Microsoft.Resources/deployments",
        "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]", 
        "apiVersion": "2017-05-10",
        "subscriptionId": "[split(parameters('workspaceResourceId'),'/')[2]]",
        "resourceGroup": "[split(parameters('workspaceResourceId'),'/')[4]]",
        "properties": {
            "mode": "Incremental",
            "template": {
                "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [
                    {
                        "apiVersion": "2015-11-01-preview",
                        "type": "Microsoft.OperationsManagement/solutions",
                        "location": "[parameters('workspaceRegion')]",
                        "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]",
                        "properties": {
                            "workspaceResourceId": "[parameters('workspaceResourceId')]"
                        },
                        "plan": {
                            "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                            "product": "[Concat('OMSGallery/', 'ContainerInsights')]",
                            "promotionCode": "",
                            "publisher": "Microsoft"
                        }
                    }
                ]
            },
            "parameters": {}
        }
       }
     ]
    }
    ```

2. Salve esse arquivo como **existingClusterOnboarding.json** em uma pasta local.
3. Cole a seguinte sintaxe JSON em seu arquivo:

    ```json
    {
       "$schema": "https://schema.management.azure.com/  schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
         "aksResourceId": {
           "value": "/subscriptions/<SubscriptiopnId>/resourcegroups/<ResourceGroup>/providers/Microsoft.ContainerService/managedClusters/<ResourceName>"
       },
       "aksResourceLocation": {
         "value": "<aksClusterLocation>"
       },
       "workspaceResourceId": {
         "value": "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroup>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>"
       },
       "workspaceRegion": {
         "value": "<workspaceLocation>"
       }
     }
    }
    ```

4. Edite os valores para **aksResourceId** e **aksResourceLocation** usando os valores na página **Visão Geral do AKS** para o cluster do AKS. O valor para **workspaceResourceId** é a ID do recurso completa do espaço de trabalho do Log Analytics, que inclui o nome do espaço de trabalho. Especifique também o local do espaço de trabalho para **workspaceRegion**. 
5. Salve esse arquivo como **existingClusterParam.json** em uma pasta local.
6. Você está pronto para implantar o modelo. 

    * Use os seguintes comandos do PowerShell na pasta que contém o modelo:

        ```powershell
        New-AzureRmResourceGroupDeployment -Name OnboardCluster -ClusterResourceGroupName ClusterResourceGroupName -TemplateFile .\existingClusterOnboarding.json -TemplateParameterFile .\existingClusterParam.json
        ```
        A alteração da configuração pode levar alguns minutos para ser concluída. Quando ela for concluída, será exibida uma mensagem semelhante à seguinte e incluindo o resultado:

        ```powershell
        provisioningState       : Succeeded
        ```

    * Para executar o comando a seguir usando a CLI do Azure:
    
        ```azurecli
        az login
        az account set --subscription "Subscription Name"
        az group deployment create --resource-group <ResourceGroupName> --template-file ./existingClusterOnboarding.json --parameters @./existingClusterParam.json
        ```

        A alteração da configuração pode levar alguns minutos para ser concluída. Quando ela for concluída, será exibida uma mensagem semelhante à seguinte e incluindo o resultado:

        ```azurecli
        provisioningState       : Succeeded
        ```
Depois de habilitar o monitoramento, poderão ser necessários cerca de 15 minutos antes de exibir as métricas de integridade para o cluster. 

## <a name="verify-agent-and-solution-deployment"></a>Verificar a implantação do agente e solução
Com a versão do agente *06072018* ou posterior, você pode verificar se tanto o agente quanto a solução foram implantados com êxito. Com versões anteriores do agente, você pode verificar somente a implantação do agente.

### <a name="agent-version-06072018-or-later"></a>Agente versão 06072018 ou posterior
Execute o seguinte comando para verificar se o agente foi implantado com êxito. 

```
kubectl get ds omsagent --namespace=kube-system
```

A saída deve ser semelhante à seguinte, que indica que ela foi implantada corretamente:

```
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```  

Para verificar a implantação da solução, execute o seguinte comando:

```
kubectl get deployment omsagent-rs -n=kube-system
```

A saída deve ser semelhante à seguinte, que indica que ela foi implantada corretamente:

```
User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system 
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
omsagent   1         1         1            1            3h
```

### <a name="agent-version-earlier-than-06072018"></a>Versão do agente anterior a 06072018

Para verificar se a versão do agente do Log Analytics liberada antes de *06072018* está implementada corretamente, execute o seguinte comando:  

```
kubectl get ds omsagent --namespace=kube-system
```

A saída deve ser semelhante à seguinte, que indica que ela foi implantada corretamente:  

```
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```  

## <a name="view-configuration-with-cli"></a>Exibir configuração com CLI
Use o comando `aks show` para obter detalhes, como se a solução estivesse habilitada ou não, o que é o resourceID do espaço de trabalho do Log Analytics e detalhes de resumo sobre o cluster.  

```azurecli
az aks show -g <resoourceGroupofAKSCluster> -n <nameofAksCluster>
```

Após alguns minutos, o comando concluirá e retornará informações no formato JSON sobre a solução.  Os resultados do comando deverão mostrar o perfil do complemento de monitoramento e serem semelhantes à seguinte saída de exemplo:

```
"addonProfiles": {
    "omsagent": {
      "config": {
        "logAnalyticsWorkspaceResourceID": "/subscriptions/<WorkspaceSubscription>/resourceGroups/<DefaultWorkspaceRG>/providers/Microsoft.OperationalInsights/workspaces/<defaultWorkspaceName>"
      },
      "enabled": true
    }
  }
```
## <a name="next-steps"></a>Próximas etapas

* Se enfrentar problemas ao tentar carregar a solução, examine o [guia de solução de problemas](monitoring-container-insights-troubleshoot.md)

* Com o monitoramento habilitado para capturar métricas de integridade para os nós de cluster do AKS e pods, essas métricas de integridade estão disponíveis no portal do Azure. Para saber como usar o Azure Monitor para contêineres, veja [Exibir integridade do Serviço de Kubernetes do Azure](monitoring-container-insights-analyze.md).