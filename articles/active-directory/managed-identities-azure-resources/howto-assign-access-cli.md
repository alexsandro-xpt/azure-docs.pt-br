---
title: Como atribuir um acesso MSI a um recurso do Azure usando a CLI do Azure
description: Instruções passo a passo sobre como atribuir um MSI em um recurso, acesso a outro recurso, usando a CLI do Azure.
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/25/2017
ms.author: daveba
ms.openlocfilehash: 2e3b85251b9dabd6efd23e5b41372703a237d227
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949070"
---
# <a name="assign-a-managed-service-identity-msi-access-to-a-resource-using-azure-cli"></a>Atribuir um acesso de MSI (Identidade de Serviço Gerenciado) a um recurso usando a CLI do Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Depois de configurar um recurso do Azure com um MSI, você pode conceder o acesso de MSI a outro recurso, assim como qualquer entidade de segurança. Este exemplo mostra como conceder o acesso de uma máquina virtual do Azure ou MSI do conjunto de dimensionamento de máquinas virtuais a uma conta de Armazenamento do Azure usando a CLI do Azure.

## <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

Para executar os exemplos de script da CLI, você tem três opções:

- Usar o [Azure Cloud Shell](../../cloud-shell/overview.md) no Portal do Azure (confira a próxima seção).
- Usar o Azure Cloud Shell inserido por meio do botão "Experimentar", localizado no canto superior direito de cada bloco de código.
- [Instale a última versão da CLI do Azure](https://docs.microsoft.com/cli/azure/install-azure-cli), se preferir usar um console de CLI local. 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="use-rbac-to-assign-the-msi-access-to-another-resource"></a>Usar o RBAC para atribuir o acesso de MSI a outro recurso

Após habilitar a MSI em um recurso do Azure, como uma [máquina virtual do Azure](qs-configure-cli-windows-vm.md) ou [conjunto de dimensionamento de máquinas virtuais do Azure](qs-configure-cli-windows-vmss.md): 

1. Se você estiver usando a CLI do Azure em um console local, primeiro entre no Azure usando o [logon az](/cli/azure/reference-index#az-login). Use uma conta associada à assinatura do Azure sob a qual deseja implantar a VM ou o conjunto de dimensionamento de máquinas virtuais:

   ```azurecli-interactive
   az login
   ```

2. Neste exemplo, fornecemos acesso de uma máquina virtual do Azure para uma conta de armazenamento. Primeiro, usamos a [lista de recursos az](/cli/azure/resource/#az-resource-list) para obter a entidade de serviço para a máquina virtual nomeada "myVM":

   ```azurecli-interactive
   spID=$(az resource list -n myVM --query [*].identity.principalId --out tsv)
   ```
   Para um conjunto de dimensionamento de máquinas virtuais do Azure, o comando é o mesmo, exceto aqui, você obtém a entidade de serviço para o conjunto de dimensionamento de máquinas virtuais nomeado "DevTestVMSS":
   
   ```azurecli-interactive
   spID=$(az resource list -n DevTestVMSS --query [*].identity.principalId --out tsv)
   ```

3. Após obter a ID da entidade e serviço, use [criar atribuição de função az](/cli/azure/role/assignment#az-role-assignment-create) para que a máquina virtual ou o conjunto de dimensionamento de máquinas virtuais configurem o acesso do "Leitor" a uma conta de armazenamento nomeada "myStorageAcct":

   ```azurecli-interactive
   az role assignment create --assignee $spID --role 'Reader' --scope /subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.Storage/storageAccounts/myStorageAcct
   ```

## <a name="troubleshooting"></a>solução de problemas

Se o MSI para o recurso não aparecer na lista de identidades disponíveis, verifique se ele foi habilitado corretamente. No nosso caso, podemos retornar para a máquina virtual do Azure ou o conjunto de dimensionamento de máquinas virtuais definido no [Portal do Azure](https://portal.azure.com) e:

- Examine a página "Configuração" e certifique-se de que o MSI esteja habilitado = "Sim".
- Examine a página "Extensões" e certifique-se de que a extensão da MSI implantada com êxito (página **Extensões** não está disponível para um conjunto de dimensionamento de máquinas virtuais do Azure).

Se um deles estiver incorreto, reimplante o MSI em seu recurso ou solucione a falha de implantação.

## <a name="related-content"></a>Conteúdo relacionado

- Para obter uma visão geral do MSI, confira [Visão geral da Identidade de Serviço Gerenciado](overview.md).
- Para habilitar a MSI em uma máquina virtual do Azure, consulte [Configurar uma MSI (Identidade de Serviço Gerenciada) da VM do Azure usando a CLI do Azure](qs-configure-cli-windows-vm.md).
- Para habilitar a MSI em um conjunto de dimensionamento de máquinas virtuais do Azure, consulte [Configurar uma MSI (Identidade de Serviço Gerenciada) do conjunto de dimensionamento de máquinas virtuais do Azure usando o Portal do Azure](qs-configure-portal-windows-vmss.md)

Use a seção de comentários a seguir para fornecer seus comentários e nos ajudar a aprimorar e adaptar nosso conteúdo.

