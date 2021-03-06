---
title: Expandir a unidade do sistema operacional de uma VM do Windows no Azure | Microsoft Docs
description: Expanda o tamanho da unidade do sistema operacional de uma máquina virtual usando o Azure PowerShell no modelo de implantação do Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: kirpasingh
manager: roshar
editor: ''
tags: azure-resource-manager
ms.assetid: d9edfd9f-482f-4c0b-956c-0d2c2c30026c
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 07/05/2018
ms.author: kirpas
ms.openlocfilehash: 3ea57a834bfbb1583c53bbb1be80daffe1f05de6
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/11/2018
ms.locfileid: "44380260"
---
# <a name="how-to-expand-the-os-drive-of-a-virtual-machine"></a>Como expandir a unidade do sistema operacional de uma máquina virtual

Quando você cria uma nova VM (máquina virtual) em um Grupo de Recursos implantando uma imagem do [Azure Marketplace](https://azure.microsoft.com/marketplace/), a unidade do sistema operacional padrão normalmente é 127 GB (algumas imagens têm tamanhos de disco de sistema operacional menores por padrão). Embora seja possível adicionar discos de dados à VM (a quantidade depende do SKU escolhido) e além de ser recomendável instalar aplicativos e cargas de trabalho que consomem bastante CPU nesses discos adicionados, muitas vezes, os clientes precisam expandir a unidade do sistema operacional para oferecer suporte a determinados cenários como estes:

- Suporte a aplicativos herdados que instalam componentes na unidade do sistema operacional.
- Migração de um PC físico ou máquina virtual local com uma unidade do sistema operacional maior.


> [!IMPORTANT]
> O redimensionamento do Disco de SO de uma Máquina Virtual do Azure causará a reinicialização do sistema.
>
> Depois de expandir os discos, você precisará [expandir o volume no sistema operacional](#expand-the-volume-within-the-os) para aproveitar o disco maior.
> 

## <a name="resize-a-managed-disk"></a>Redimensionar um disco gerenciado

Abra o ISE do PowerShell ou a janela do PowerShell no modo administrativo e siga as etapas abaixo:

1. Entre em sua conta do Microsoft Azure no modo de gerenciamento de recursos e escolha a sua assinatura como se segue:
   
   ```powershell
   Connect-AzureRmAccount
   Select-AzureRmSubscription –SubscriptionName 'my-subscription-name'
   ```
2. Defina o nome do grupo de recursos e o nome da VM como se segue:
   
   ```powershell
   $rgName = 'my-resource-group-name'
   $vmName = 'my-vm-name'
   ```
3. Obtenha uma referência para sua VM da seguinte maneira:
   
   ```powershell
   $vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
   ```
4. Interrompa a VM antes de redimensionar o disco da seguinte maneira:
   
    ```Powershell
    Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
    ```
5. Obtenha uma referência ao disco do sistema operacional gerenciado. Defina o tamanho do disco do sistema operacional gerenciado para o valor desejado e atualize o disco da seguinte maneira:
   
   ```Powershell
   $disk= Get-AzureRmDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
   $disk.DiskSizeGB = 1023
   Update-AzureRmDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
   ```   
   > [!WARNING]
   > O novo tamanho deve ser maior que o tamanho do disco existente. O máximo permitido é de 2048 GB para discos do sistema operacional. (É possível expandir o blob VHD além desse tamanho, mas o sistema operacional só poderá trabalhar com os primeiros 2048 GB de espaço.)
   > 
   > 
6. A atualização da VM pode demorar um pouco. Depois que o comando concluir a execução, reinicie a VM como se segue:
   
   ```Powershell
   Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
   ```

É isso. Agora, com o RDP na VM, abra Gerenciamento do Computador (ou Gerenciamento de Disco) e expanda a unidade usando o espaço recentemente alocado.

## <a name="resize-an-unmanaged-disk"></a>Redimensionar um disco não gerenciado

Abra o ISE do PowerShell ou a janela do PowerShell no modo administrativo e siga as etapas abaixo:

1. Entre em sua conta do Microsoft Azure no modo de gerenciamento de recursos e escolha a sua assinatura como se segue:
   
   ```Powershell
   Connect-AzureRmAccount
   Select-AzureRmSubscription –SubscriptionName 'my-subscription-name'
   ```
2. Defina o nome do grupo de recursos e o nome da VM como se segue:
   
   ```Powershell
   $rgName = 'my-resource-group-name'
   $vmName = 'my-vm-name'
   ```
3. Obtenha uma referência para sua VM da seguinte maneira:
   
   ```Powershell
   $vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
   ```
4. Interrompa a VM antes de redimensionar o disco da seguinte maneira:
   
    ```Powershell
    Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
    ```
5. Defina o tamanho do disco do sistema operacional não gerenciado para o valor desejado e atualize a VM da seguinte maneira:
   
   ```Powershell
   $vm.StorageProfile.OSDisk.DiskSizeGB = 1023
   Update-AzureRmVM -ResourceGroupName $rgName -VM $vm
   ```
   
   > [!WARNING]
   > O novo tamanho deve ser maior que o tamanho do disco existente. O máximo permitido é de 2048 GB para discos do sistema operacional. (É possível expandir o blob VHD além desse tamanho, mas o sistema operacional só poderá trabalhar com os primeiros 2048 GB de espaço.)
   > 
   > 
   
6. A atualização da VM pode demorar um pouco. Depois que o comando concluir a execução, reinicie a VM como se segue:
   
   ```Powershell
   Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
   ```


## <a name="scripts-for-os-disk"></a>Scripts para o disco do sistema operacional

Abaixo está o script completo para sua referência para discos gerenciados e não gerenciados:


**Discos gerenciados**

```Powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
$disk= Get-AzureRmDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
$disk.DiskSizeGB = 1023
Update-AzureRmDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
```

**Discos não gerenciados**

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
$vm.StorageProfile.OSDisk.DiskSizeGB = 1023
Update-AzureRmVM -ResourceGroupName $rgName -VM $vm
Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName
```

## <a name="resizing-data-disks"></a>Redimensionando discos de dados

Este artigo se concentra principalmente em expandir o disco do SO da VM, mas o script também pode ser usado para expandir os discos de dados conectados à VM. Por exemplo, para expandir o primeiro disco de dados conectado à VM, substitua o objeto `OSDisk` de `StorageProfile` pela matriz `DataDisks` e use um índice numérico para obter uma referência para o primeiro disco de dados conectado, como mostrado abaixo:

**Disco gerenciado**

```powershell
$disk= Get-AzureRmDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.DataDisks[0].Name
$disk.DiskSizeGB = 1023
```


**Disco não gerenciado**

```powershell
$vm.StorageProfile.DataDisks[0].DiskSizeGB = 1023
```



Da mesma forma, você pode referenciar outros discos de dados conectados à VM usando um índice, conforme é mostrado acima, ou a propriedade **Nome** do disco:


**Disco gerenciado**

```powershell
(Get-AzureRmDisk -ResourceGroupName $rgName -DiskName ($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'})).Name).DiskSizeGB = 1023
```

**Disco não gerenciado**

```powershell
($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'}).DiskSizeGB = 1023
```

## <a name="expand-the-volume-within-the-os"></a>Expandir o volume no sistema operacional

Após expandir o disco da VM, você precisará acessar o sistema operacional e expandir o volume para abranger o novo espaço. Há vários métodos para expandir uma partição. Esta seção aborda como conectar a VM usando uma conexão RDP para expandir a partição usando **DiskPart**.

1. Abra uma conexão RDP com a VM.

2.  Abra um prompt de comando e digite **diskpart**.

2.  No prompt **DISKPART**, digite `list volume`. Anote o volume que você deseja estender.

3.  No prompt **DISKPART**, digite `select volume <volumenumber>`. Isso seleciona o volume *volumenumber* que você deseja estender no espaço vazio contíguo no mesmo disco.

4.  No prompt **DISKPART**, digite `extend [size=<size>]`. Isso estende o volume selecionado pelo *tamanho* em megabytes (MB).


##<a name="next-steps"></a>Próximas etapas

Você também pode conectar discos usando o [portal do Azure](attach-managed-disk-portal.md).



