---
title: Atualizar um registro de contêiner do Azure Clássico
description: Aproveite o conjunto expandido de recursos dos registros de contêiner gerenciados Básico, Standard e Premium atualizando seu registro de contêiner Clássico não gerenciado.
services: container-registry
author: mmacy
manager: jeconnoc
ms.service: container-registry
ms.topic: article
ms.date: 08/28/2018
ms.author: marsma
ms.openlocfilehash: 7a377c607639f5c044e689b11380e9778f9c72c8
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189014"
---
# <a name="upgrade-a-classic-container-registry"></a>Atualizar um registro de contêiner Clássico

O ACR (Registro de Contêiner do Azure) está disponível em várias camadas de serviço, [conhecidas como SKUs](container-registry-skus.md). A versão inicial do ACR oferecia um único SKU, o Clássico, que não tinha vários recursos inerentes aos SKUs Básico, Standard e Premium (coletivamente conhecidos como registros *gerenciados*).

O SKU clássico está sendo preterido e não estará disponível depois de março de 2019. Este artigo fornece detalhes sobre como migrar seu registro Clássico não gerenciado para um dos SKUs gerenciados para que você possa tirar proveito do seu conjunto de recursos aprimorado.

## <a name="why-upgrade"></a>Por que atualizar?

O SKU de registro clássico está sendo **preterido** e não estará disponível a partir de **março de 2019**. Todos os registros clássicos existentes devem ser atualizados antes de março de 2019.

Devido às funcionalidades limitadas e ao planejamento de se preterir os registros não gerenciados Clássicos, todos os registros Clássicos devem ser atualizados para registros gerenciados Básico, Standard ou Premium. Essas SKUs de nível superior integram o registro aos recursos do Azure mais profundamente.

Os registros gerenciados fornecem:

* Integração com o Azure Active Directory para [logon individual](container-registry-authentication.md#individual-login-with-azure-ad)
* Suporte a exclusão de marca e imagem
* [Replicação geográfica](container-registry-geo-replication.md)
* [Webhooks](container-registry-webhook.md)

O registro Clássico depende da conta de armazenamento que o Azure provisionou automaticamente na sua assinatura do Azure quando você criou o registro. Por outro lado, SKUs Básico, Standard e Premium aproveitam os [recursos de armazenamento avançados](container-registry-storage.md) do Azure, manipulando de modo transparente o armazenamento das imagens para você. Uma conta de armazenamento separada não é criada em sua própria assinatura.

Um armazenamento de registro gerenciado oferece os seguintes benefícios:

* As imagens de contêiner são [criptografadas em repouso](container-registry-storage.md#encryption-at-rest).
* As imagens são armazenadas usando [armazenamento com redundância geográfica](container-registry-storage.md#geo-redundant-storage), garantindo o backup das imagens com a replicação em várias regiões.
* Capacidade de [movimentação entre SKUs](container-registry-skus.md#changing-skus) de modo livre, permitindo uma maior taxa de transferência ao escolher um SKU de nível superior. Com cada SKU, o ACR pode atender a seus requisitos de taxa de transferência conforme a necessidade.
* O modelo de segurança unificada para o registro e seu armazenamento fornece gerenciamento de direitos simplificado. Você pode gerenciar permissões apenas para o registro de contêiner, sem a necessidade de gerenciar também permissões para uma conta de armazenamento separada.

Para obter detalhes adicionais sobre o armazenamento de imagens no ACR, consulte [Armazenamento de imagens de contêiner no Registro de Contêiner do Azure](container-registry-storage.md).

## <a name="migration-considerations"></a>Considerações sobre a migração

Ao alterar um registro Clássico para um registro gerenciado, o Azure deve copiar todas as imagens de contêiner existentes da conta de armazenamento criada por ACR na sua assinatura para uma conta de armazenamento gerenciada pelo Azure. Dependendo do tamanho do registro, esse processo pode demorar de alguns minutos a várias horas.

Durante o processo de conversão, todas as operações `docker push` são bloqueadas, enquanto o `docker pull` continua a funcionar.

Não exclua nem modifique o conteúdo da conta de armazenamento que dá suporte ao seu registro Clássico durante o processo de conversão. Isso pode resultar na corrupção de suas imagens de contêiner.

Quando a migração é concluída, a conta de armazenamento em sua assinatura que originalmente dava suporte ao registro Clássico é mais usada pelo ACR. Depois de verificar que a migração foi bem-sucedida, considere a exclusão da conta de armazenamento para ajudar a minimizar o custo.

>[!IMPORTANT]
> A atualização do Clássico para um dos SKUs gerenciados é um **processo unidirecional**. Depois de converter um registro Clássico para Básico, Standard ou Premium, não será possível reverter para o Clássico. É possível, no entanto, mover livremente entre SKUs gerenciados com uma capacidade suficiente para o registro.

## <a name="how-to-upgrade"></a>Como atualizar

É possível atualizar um registro Clássico não gerenciado para um dos SKUs gerenciados de várias maneiras. Nas seções a seguir, descrevemos o processo para usar a [CLI do Azure][azure-cli] e o [Portal do Azure][azure-portal].

## <a name="upgrade-in-azure-cli"></a>Atualização na CLI do Azure

Para atualizar um registro Clássico na CLI do Azure, execute o comando [az acr update][az-acr-update] e especifique o novo SKU para o registro. No exemplo a seguir, um registro Clássico denominado *myclassicregistry* é atualizado para o SKU Premium:

```azurecli-interactive
az acr update --name myclassicregistry --sku Premium
```

Quando a migração for concluída, você verá uma saída semelhante à seguinte. Observe que o `sku` é "Premium" e o `storageAccount` é "null", o que indica que o Azure agora gerencia o armazenamento de imagens para esse registro.

```JSON
{
  "adminUserEnabled": false,
  "creationDate": "2017-12-12T21:23:29.300547+00:00",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry",
  "location": "eastus",
  "loginServer": "myregistry.azurecr.io",
  "name": "myregistry",
  "provisioningState": "Succeeded",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "name": "Premium",
    "tier": "Premium"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

Se especificar um SKU de registro gerenciado cuja capacidade máxima é menor que o tamanho do registro Clássico, você receberá uma mensagem de erro semelhante à seguinte.

`Cannot update the registry SKU due to reason: Registry size 12936251113 bytes exceeds the quota value 10737418240 bytes for SKU Basic. The suggested SKU is Standard.`

Se receber um erro semelhante, execute o comando [az acr update][az-acr-update] novamente e especifique o SKU sugerido, que é o próximo SKU de nível mais alto que pode acomodar suas imagens.

## <a name="upgrade-in-azure-portal"></a>Atualizar no Portal do Azure

Ao atualizar um registro Clássico usando o Portal do Azure, o Azure seleciona automaticamente o SKU de nível mais baixo que pode acomodar suas imagens. Por exemplo, se o registro contiver 12 GiB em imagens, o Azure selecionará e converterá automaticamente o registro Clássico para o Standard (máximo de 100 GiB).

Para atualizar o registro Clássico usando o Portal do Azure, navegue até o registro de contêiner **Visão geral** e selecione **Atualizar para o registro gerenciado**.

![Botão de atualização do registro Clássico na interface do usuário do Portal do Azure][update-classic-01-upgrade]

Selecione **OK** para confirmar que você deseja atualizar um registro gerenciado.

![Confirmação da atualização do registro Clássico na interface do usuário do Portal do Azure][update-classic-02-confirm]

Durante a migração, o portal indica que o **estado de provisionamento** do registro é *Atualizando*. Como mencionado anteriormente, as operações `docker push` são desabilitadas durante a migração e você não deve excluir nem atualizar a conta de armazenamento usada pelo registro Clássico enquanto a migração estiver em andamento. Isso poderá resultar na corrupção da imagem.

![Progresso da atualização do registro Clássico na interface do usuário do Portal do Azure][update-classic-03-updating]

Quando a migração for concluída, o **Estado de provisionamento** indicará *Êxito*, e você poderá `docker push` novamente no registro.

![Estado de conclusão da atualização do registro Clássico na interface do usuário do Portal do Azure][update-classic-04-updated]

## <a name="next-steps"></a>Próximas etapas

Depois que você tiver atualizado um registro Clássico para Básico, Standard ou Premium, o Azure não usará a conta de armazenamento que originalmente deu suporte ao registro Clássico. Para reduzir o custo, considere a possibilidade de excluir a conta de armazenamento ou o contêiner de Blob dentro da conta que contém as imagens do contêiner antigo.

<!-- IMAGES -->
[update-classic-01-upgrade]: ./media/container-registry-upgrade\update-classic-01-upgrade.png
[update-classic-02-confirm]: ./media/container-registry-upgrade\update-classic-02-confirm.png
[update-classic-03-updating]: ./media/container-registry-upgrade\update-classic-03-updating.png
[update-classic-04-updated]: ./media/container-registry-upgrade\update-classic-04-updated.png

<!-- LINKS - internal -->
[az-acr-update]: /cli/azure/acr#az-acr-update
[azure-cli]: /cli/azure/install-azure-cli
[azure-portal]: https://portal.azure.com