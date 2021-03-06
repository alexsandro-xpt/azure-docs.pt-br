---
title: Início Rápido - Criar um conjunto de dimensionamento de máquinas virtuais com a CLI do Azure | Microsoft Docs
description: Saiba como criar um rapidamente uma escala de máquina virtual com o Azure PowerShell
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: quickstart
ms.custom: mvc
ms.date: 03/27/18
ms.author: cynthn
ms.openlocfilehash: 348074592da8465762b178b49af4713aadcc8c8d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970645"
---
# <a name="quickstart-create-a-virtual-machine-scale-set-with-the-azure-cli"></a>Início Rápido: criar um conjunto de dimensionamento de máquinas virtuais com a CLI do Azure
Um conjunto de dimensionamento de máquinas virtuais permite implantar e gerenciar um conjunto de máquinas virtuais idênticas de dimensionamento automático. É possível dimensionar o número de VMs manualmente no conjunto de dimensionamento ou definir as regras para o dimensionamento automático com base no uso de recursos, como CPU, demanda de memória ou tráfego de rede. Um balanceador de carga do Azure então distribui o tráfego para as instâncias de VM no conjunto de dimensionamento. Neste guia de início rápido, você cria um conjunto de dimensionamento de máquinas virtuais e implanta um aplicativo de exemplo com a CLI do Azure.

Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Se optar por instalar e usar a CLI localmente, este tutorial exigirá que você esteja executando a CLI do Azure versão 2.0.29 ou posterior. Execute `az --version` para encontrar a versão. Se você precisa instalar ou atualizar, consulte [Instalar a CLI do Azure]( /cli/azure/install-azure-cli). 


## <a name="create-a-scale-set"></a>Criar um conjunto de escala
Antes de criar uma máquina virtual, crie um grupo de recursos com o [az group create](/cli/azure/group#az_group_create). O exemplo a seguir cria um grupo de recursos chamado *myResourceGroup* na localização *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Crie um conjunto de dimensionamento de máquinas virtuais com [az vmss create](/cli/azure/vmss#az_vmss_create). O exemplo a seguir cria um conjunto de dimensionamento chamado *myScaleSet* que está definido para atualizar automaticamente à medida que alterações são aplicadas e gera chaves SSH caso elas não existam em *~/.ssh/id_rsa*. Essas chaves SSH são usadas se você precisar fazer logon nas instâncias de VM. Para usar um conjunto existente de chaves SSH, em vez disso, use o parâmetro `--ssh-key-value` e especifique o local de suas chaves.

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys
```

Leva alguns minutos para criar e configurar todos os recursos e as VMs do conjunto de dimensionamento.


## <a name="deploy-sample-application"></a>Implantar um aplicativo de exemplo
Para testar seu conjunto de dimensionamento, instale um aplicativo Web básico. A Extensão de Script Personalizado do Azure para baixar e executar um script que instala um aplicativo nas instâncias de VM. Essa extensão é útil para a configuração de implantação de postagem, instalação de software ou qualquer outra configuração/tarefa de gerenciamento. Para obter mais informações, consulte a [Visão geral da Extensão de Script Personalizado](../virtual-machines/linux/extensions-customscript.md).

Use a Extensão de Script Personalizado para instalar um servidor Web NGINX básico. Aplique a Extensão do Script Personalizado que instala o NGINX com [az vmss extension set](/cli/azure/vmss/extension#az_vmss_extension_set) da seguinte maneira:

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],"commandToExecute":"./automate_nginx.sh"}'
```


## <a name="allow-traffic-to-application"></a>Permitir o tráfego para o aplicativo
Quando o conjunto de dimensionamento foi criado, um balanceador de carga do Azure foi automaticamente implantado. Um balanceador de carga distribui o tráfego para as instâncias de VM no conjunto de dimensionamento. Para permitir que o tráfego alcance o aplicativo Web de exemplo, crie uma regra de balanceador de carga com [az network lb rule create](/cli/azure/network/lb/rule#az_network_lb_rule_create). O exemplo a seguir cria uma regra chamada *myLoadBalancerRuleWeb*:

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroup \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```


## <a name="test-your-scale-set"></a>Testar seu conjunto de dimensionamento
Para ver seu conjunto de dimensionamento em ação, acesse o aplicativo Web de exemplo em um navegador da Web. Obtenha o endereço IP público de seu balanceador de carga com [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show). O seguinte exemplo obtém o endereço IP de *myScaleSetLBPublicIP*, criado como parte do conjunto de dimensionamento:

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query '[ipAddress]' \
  --output tsv
```

Insira o endereço IP público do balanceador de carga em um navegador da Web. O balanceador de carga distribui o tráfego para uma de suas instâncias de VM, conforme mostrado no exemplo a seguir:

![Página da Web padrão em NGINX](media/virtual-machine-scale-sets-create-cli/running-nginx-site.png)


## <a name="clean-up-resources"></a>Limpar recursos
Quando não for mais necessário, use [az group delete](/cli/azure/group#az_group_delete) para remover o grupo de recursos, o conjunto de dimensionamento e todos os recursos relacionados, como demonstrado a seguir. O parâmetro `--no-wait` retorna o controle ao prompt sem aguardar a conclusão da operação. O parâmetro `--yes` confirma que você deseja excluir os recursos sem um prompt adicional para fazer isso.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```


## <a name="next-steps"></a>Próximas etapas
Neste início rápido, você criou um conjunto de dimensionamento básico e usou uma Extensão de Script Personalizado para instalar um servidor Web NGINX básico nas instâncias de VM. Para saber mais, siga para o tutorial sobre como criar e gerenciar um conjunto de dimensionamento de máquinas virtuais do Azure.

> [!div class="nextstepaction"]
> [Criar e gerenciar um conjunto de dimensionamento de máquinas virtuais do Azure](tutorial-create-and-manage-cli.md)

