---
title: Início Rápido do Azure - Criar um blob no armazenamento de objeto usando Azure PowerShell | Microsoft Docs
description: Neste início rápido, você usa o Azure PowerShell no armazenamento de objeto (Blob). Em seguida, use o PowerShell para carregar um blob no Armazenamento do Azure, baixar um blob e listar os blobs em um contêiner.
services: storage
author: roygara
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 04/09/2018
ms.author: rogarana
ms.openlocfilehash: b482379c05133dcf58e54bd01f38f0c3cee95e8d
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/01/2018
ms.locfileid: "39398586"
---
# <a name="quickstart-upload-download-and-list-blobs-using-azure-powershell"></a>Início rápido: Carregar, baixar e listar blobs usando o Azure PowerShell

O módulo do Azure PowerShell é usado para criar e gerenciar recursos do Azure da linha de comando do PowerShell ou em scripts. Este guia detalha o uso do PowerShell para transferir arquivos entre o disco local e o Armazenamento de Blobs do Azure.

Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

Este início rápido requer o módulo Azure PowerShell versão 3.6 ou posterior. Execute `Get-Module -ListAvailable AzureRM` para encontrar a versão. Se você precisar instalá-lo ou atualizá-lo, confira [Instalar o módulo do Azure PowerShell](/powershell/azure/install-azurerm-ps).

[!INCLUDE [storage-quickstart-tutorial-intro-include-powershell](../../../includes/storage-quickstart-tutorial-intro-include-powershell.md)]

## <a name="create-a-container"></a>Criar um contêiner

Os blobs são sempre carregados em um contêiner. É possível organizar grupos de blobs de forma similar àquela em que você organiza os arquivos em pastas no seu computador.

Defina o nome do contêiner e crie o contêiner usando [New-AzureStorageContainer](/powershell/module/azure.storage/new-azurestoragecontainer), definindo as permissões para o “blob” para permitir o acesso público dos arquivos. O nome do contêiner neste exemplo é *quickstartblobs*.

```powershell
$containerName = "quickstartblobs"
New-AzureStorageContainer -Name $containerName -Context $ctx -Permission blob
```

## <a name="upload-blobs-to-the-container"></a>Carregar blobs para o contêiner

O Armazenamento de Blobs dá suporte a blobs de blocos, blobs de acréscimo e blobs de páginas. Os arquivos VHD usados para auxiliar VMs IaaS são blobs de páginas. Os blobs de acréscimo são usados para registro em log, como quando você quer gravar em um arquivo e depois adicionar mais informações. A maioria dos arquivos armazenados no Armazenamento de Blobs são blobs de blocos. 

Para carregar um arquivo em um blob de blocos, obtenha uma referência de contêiner e uma referência para o blob de blocos nesse contêiner. Após obter a referência de blob, você pode carregar dados nele usando [Set-AzureStorageBlobContent](/powershell/module/azure.storage/set-azurestorageblobcontent). Essa operação cria o blob, se ele ainda não existir, ou o substitui, se ele já existir.

Os exemplos a seguir carregam a Image001.jpg e Image002.png da pasta D:\\_TestImages no disco local para o contêiner que você criou.

```powershell
# upload a file
Set-AzureStorageBlobContent -File "D:\_TestImages\Image001.jpg" `
  -Container $containerName `
  -Blob "Image001.jpg" `
  -Context $ctx 

# upload another file
Set-AzureStorageBlobContent -File "D:\_TestImages\Image002.png" `
  -Container $containerName `
  -Blob "Image002.png" `
  -Context $ctx
```

Carregue quantos arquivos desejar antes de continuar.

## <a name="list-the-blobs-in-a-container"></a>Listar os blobs em um contêiner

Obtenha uma lista de blobs no contêiner usando [Get-AzureStorageBlob](/powershell/module/azure.storage/get-azurestorageblob). Este exemplo mostra apenas os nomes dos blobs carregados.

```powershell
Get-AzureStorageBlob -Container $ContainerName -Context $ctx | select Name 
```

## <a name="download-blobs"></a>Baixar blobs

Baixe os blobs em seu disco local. Para cada blob a ser baixado, defina o nome e a chame [Get-AzureStorageBlobContent](/powershell/module/azure.storage/get-azurestorageblobcontent) para baixar o blob.

Este exemplo baixa os blobs para D:\\_TestImages\Downloads no disco local. 

```powershell
# download first blob
Get-AzureStorageBlobContent -Blob "Image001.jpg" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx 

# download another blob
Get-AzureStorageBlobContent -Blob "Image002.png" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx 
```

## <a name="data-transfer-with-azcopy"></a>Transferência de dados com AzCopy

O utilitário [AzCopy](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) é outra opção para a transferência de dados programável por script de alto desempenho para o Armazenamento do Azure. Você pode usar o AzCopy para transferir dados para e do armazenamento de Blobs, Arquivo e Tabela.

Como um exemplo rápido, aqui está o comando do AzCopy para carregar um arquivo chamado *myfile.txt* para o contêiner *mystoragecontainer* de dentro de uma janela do PowerShell.

```PowerShell
./AzCopy `
    /Source:C:\myfolder `
    /Dest:https://mystorageaccount.blob.core.windows.net/mystoragecontainer `
    /DestKey:<storage-account-access-key> `
    /Pattern:"myfile.txt"
```

## <a name="clean-up-resources"></a>Limpar recursos

Remova todos os ativos que você criou. A maneira mais fácil de fazer isso é excluir o grupo de recursos. Isso também exclui todos os recursos contidos no grupo. Nesse caso, ele remove a conta de armazenamento e o próprio grupo de recursos.

```powershell
Remove-AzureRmResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Próximas etapas

Nesse guia de início rápido, você aprendeu a transferir arquivos entre o disco local e Armazenamento de Blobs do Azure. Para saber mais sobre como trabalhar com o armazenamento de Blobs usando o PowerShell, continue a usar o Guia de instruções do Azure PowerShell com o Armazenamento do Azure.

> [!div class="nextstepaction"]
> [Usando o PowerShell do Azure com o Armazenamento do Azure](../common/storage-powershell-guide-full.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

### <a name="microsoft-azure-powershell-storage-cmdlets-reference"></a>Referência de cmdlets de Armazenamento do Microsoft Azure PowerShell

* [Cmdlets do PowerShell do Armazenamento](/powershell/module/azurerm.storage#storage)

### <a name="microsoft-azure-storage-explorer"></a>Gerenciador do Armazenamento do Microsoft Azure

* [O Gerenciador de Armazenamento do Microsoft Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) é um aplicativo autônomo e gratuito da Microsoft que possibilita o trabalho visual com os dados do Armazenamento do Azure no Windows, MacOS e Linux.