---
title: Crie e gerencie as VMs do Windows no Azure que usam várias NICs | Microsoft Docs
description: Saiba como criar e gerenciar uma VM do Windows que tem várias NICs anexadas usando o Azure PowerShell ou modelos do Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: 9bff5b6d-79ac-476b-a68f-6f8754768413
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 09/26/2017
ms.author: cynthn
ms.openlocfilehash: 47f02c008a0498492af3503d90fda8ff6e2eefa8
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/17/2018
ms.locfileid: "42142226"
---
# <a name="create-and-manage-a-windows-virtual-machine-that-has-multiple-nics"></a>Cria e gerencia uma máquina virtual do Windows que tem várias NICs
As máquinas virtuais (VMs) no Azure podem ter várias placas de interface de rede virtual (NICs) anexadas a elas. Um cenário comum é ter sub-redes diferentes para conectividade de front-end e back-end. Você pode associar várias NICs em uma VM para várias sub-redes, mas essas sub-redes devem residir na mesma rede virtual (vNet). Este artigo fornece detalhes sobre como criar uma VM que tem várias NICs anexadas. Você também aprenderá a adicionar ou remover as NICs de uma VM existente. Diferentes [tamanhos de VM](sizes.md) dão suporte a um número variável de NICs, sendo assim, dimensione sua VM adequadamente.

## <a name="prerequisites"></a>Pré-requisitos
Verifique se você tem [ a versão mais recente do Azure PowerShell instalada e configurada](/powershell/azure/overview).

Nos exemplos a seguir, substitua os nomes de parâmetro de exemplo com seus próprios valores. Os nomes de parâmetro de exemplo incluem *myResourceGroup*, *myVnet* e *myVM*.


## <a name="create-a-vm-with-multiple-nics"></a>Criar uma VM com diversos NICs
Em primeiro lugar, crie um grupo de recursos. O exemplo a seguir cria um grupo de recursos chamado *myResourceGroup* no local *EastUs*:

```powershell
New-AzureRmResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

### <a name="create-virtual-network-and-subnets"></a>Crie a rede virtual e as sub-redes
Um cenário comum é para uma rede virtual ter duas ou mais sub-redes. Uma sub-rede pode ser usada para o tráfego de front-end, a outra para o tráfego de back-end. Para se conectar a ambas as sub-redes, você usa várias NICs em sua VM.

1. Defina duas sub-redes de rede virtual com [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig). O exemplo a seguir define as sub-redes, para *mySubnetFrontEnd* e *mySubnetBackEnd*:

    ```powershell
    $mySubnetFrontEnd = New-AzureRmVirtualNetworkSubnetConfig -Name "mySubnetFrontEnd" `
        -AddressPrefix "192.168.1.0/24"
    $mySubnetBackEnd = New-AzureRmVirtualNetworkSubnetConfig -Name "mySubnetBackEnd" `
        -AddressPrefix "192.168.2.0/24"
    ```

2. Crie uma rede virtual e sub-redes com [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork). O seguinte exemplo cria uma rede virtual chamada *myVnet*:

    ```powershell
    $myVnet = New-AzureRmVirtualNetwork -ResourceGroupName "myResourceGroup" `
        -Location "EastUs" `
        -Name "myVnet" `
        -AddressPrefix "192.168.0.0/16" `
        -Subnet $mySubnetFrontEnd,$mySubnetBackEnd
    ```


### <a name="create-multiple-nics"></a>Crie múltiplas NICs
Crie duas NICs com [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface). Anexe uma NIC à sub-rede de front-end e uma NIC à sub-rede de back-end. O exemplo a seguir cria NICs, chamadas *myNic1* e *myNic2*:

```powershell
$frontEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetFrontEnd'}
$myNic1 = New-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic1" `
    -Location "EastUs" `
    -SubnetId $frontEnd.Id

$backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}
$myNic2 = New-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic2" `
    -Location "EastUs" `
    -SubnetId $backEnd.Id
```

Normalmente, você também criaria um [grupo de segurança de rede](../../virtual-network/security-overview.md) para filtrar o tráfego de rede para a VM e um [balanceador de carga](../../load-balancer/load-balancer-overview.md) para distribuir o tráfego entre diversas VMs.

### <a name="create-the-virtual-machine"></a>Criar a máquina virtual
Agora, comece a criar sua configuração de VM. Cada tamanho de VM tem um limite para o número total de NICs que podem ser adicionada a uma VM. Para obter mais informações, consulte [Tamanhos da VM no Windows](sizes.md) .

1. Defina suas credenciais de VM para a variável `$cred` da seguinte maneira:

    ```powershell
    $cred = Get-Credential
    ```

2. Defina sua VM com [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig). O exemplo a seguir define uma VM chamada *myVM* e usa um tamanho de VM que dá suporte a até duas NICs (*Standard_DS3_v2*):

    ```powershell
    $vmConfig = New-AzureRmVMConfig -VMName "myVM" -VMSize "Standard_DS3_v2"
    ```

3. Cria o restante da sua configuração de VM com [Set-AzureRmVMOperatingSystem](/powershell/module/azurerm.compute/set-azurermvmoperatingsystem) e [Set-AzureRmVMSourceImage](/powershell/module/azurerm.compute/set-azurermvmsourceimage). O exemplo a seguir cria uma VM do Windows Server 2016:

    ```powershell
    $vmConfig = Set-AzureRmVMOperatingSystem -VM $vmConfig `
        -Windows `
        -ComputerName "myVM" `
        -Credential $cred `
        -ProvisionVMAgent `
        -EnableAutoUpdate
    $vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig `
        -PublisherName "MicrosoftWindowsServer" `
        -Offer "WindowsServer" `
        -Skus "2016-Datacenter" `
        -Version "latest"
   ```

4. Anexe as duas NICs que você criou anteriormente com [Add-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/add-azurermvmnetworkinterface):

    ```powershell
    $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $myNic1.Id -Primary
    $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $myNic2.Id
    ```

5. Crie sua VM com [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm):

    ```powershell
    New-AzureRmVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "EastUs"
    ```

6. Adicione rotas para NICs secundárias ao SO, concluindo as etapas em [Configurar o sistema operacional para várias NICs](#configure-guest-os-for-multiple-nics).

## <a name="add-a-nic-to-an-existing-vm"></a>Adicionar uma NIC a uma VM existente
Para adicionar uma NIC virtual a uma VM existente, você desalocar a VM, adiciona a NIC virtual e inicia a VM. Diferentes [tamanhos de VM](sizes.md) dão suporte a um número variável de NICs, sendo assim, dimensione sua VM adequadamente. Se necessário, é possível [redimensionar uma VM](resize-vm.md).

1. Desaloque a VM com o [Stop-AzureRmVM](/powershell/module/azurerm.compute/stop-azurermvm). O seguinte exemplo desaloca a VM chamada *myVM* no *myResourceGroup*:

    ```powershell
    Stop-AzureRmVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. Obtenha a configuração existente da VM com [Get-AzureRmVm](/powershell/module/azurerm.compute/get-azurermvm). O exemplo a seguir obtém informações sobre a VM chamada *myVM* no *myResourceGroup*:

    ```powershell
    $vm = Get-AzureRmVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. O exemplo a seguir cria uma NIC virtual com [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) chamada *myNic3* que está anexada ao *mySubnetBackEnd*. A NIC virtual é anexada a VM chamada *myVM* no *myResourceGroup* com [Add-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/add-azurermvmnetworkinterface):

    ```powershell
    # Get info for the back end subnet
    $myVnet = Get-AzureRmVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup"
    $backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}

    # Create a virtual NIC
    $myNic3 = New-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" `
        -Name "myNic3" `
        -Location "EastUs" `
        -SubnetId $backEnd.Id

    # Get the ID of the new virtual NIC and add to VM
    $nicId = (Get-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" -Name "MyNic3").Id
    Add-AzureRmVMNetworkInterface -VM $vm -Id $nicId | Update-AzureRmVm -ResourceGroupName "myResourceGroup"
    ```

    ### <a name="primary-virtual-nics"></a>NICs virtuais primárias
    Uma das NICs em uma VM com várias NICs precisa ser primária. Se uma das NICs virtuais existentes na VM já estiver definida como primária, você pode ignorar esta etapa. O exemplo a seguir presume que duas NICs virtuais agora estão presentes em uma VM e você deseja adicionar a primeira NIC (`[0]`) como a primária:
        
    ```powershell
    # List existing NICs on the VM and find which one is primary
    $vm.NetworkProfile.NetworkInterfaces
    
    # Set NIC 0 to be primary
    $vm.NetworkProfile.NetworkInterfaces[0].Primary = $true
    $vm.NetworkProfile.NetworkInterfaces[1].Primary = $false
    
    # Update the VM state in Azure
    Update-AzureRmVM -VM $vm -ResourceGroupName "myResourceGroup"
    ```

4. Inicie a VM com [Start-AzureRmVm](/powershell/module/azurerm.compute/start-azurermvm):

    ```powershell
    Start-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVM"
    ```

5. Adicione rotas para NICs secundárias ao SO, concluindo as etapas em [Configurar o sistema operacional para várias NICs](#configure-guest-os-for-multiple-nics).

## <a name="remove-a-nic-from-an-existing-vm"></a>Remover uma NIC de uma VM existente
Para remover uma NIC virtual de uma VM existente, você desaloca a VM, remove a NIC virtual e inicia a VM.

1. Desaloque a VM com o [Stop-AzureRmVM](/powershell/module/azurerm.compute/stop-azurermvm). O seguinte exemplo desaloca a VM chamada *myVM* no *myResourceGroup*:

    ```powershell
    Stop-AzureRmVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. Obtenha a configuração existente da VM com [Get-AzureRmVm](/powershell/module/azurerm.compute/get-azurermvm). O exemplo a seguir obtém informações sobre a VM chamada *myVM* no *myResourceGroup*:

    ```powershell
    $vm = Get-AzureRmVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. Obter informações sobre a remoção de NIC com [Get-AzureRmNetworkInterface](/powershell/module/azurerm.network/get-azurermnetworkinterface). O seguinte exemplo obtém informações sobre *myNic3*:

    ```powershell
    # List existing NICs on the VM if you need to determine NIC name
    $vm.NetworkProfile.NetworkInterfaces

    $nicId = (Get-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" -Name "myNic3").Id   
    ```

4. Remova a NIC com [Remove-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/remove-azurermvmnetworkinterface) e, em seguida, atualize a VM com [Update-AzureRmVm](/powershell/module/azurerm.compute/update-azurermvm). O exemplo a seguir remove *myNic3* conforme obtidas pelo `$nicId` na etapa anterior:

    ```powershell
    Remove-AzureRmVMNetworkInterface -VM $vm -NetworkInterfaceIDs $nicId | `
        Update-AzureRmVm -ResourceGroupName "myResourceGroup"
    ```   

5. Inicie a VM com [Start-AzureRmVm](/powershell/module/azurerm.compute/start-azurermvm):

    ```powershell
    Start-AzureRmVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```   

## <a name="create-multiple-nics-with-templates"></a>Criar várias NICs com modelos
Os modelos do Azure Resource Manager oferecem uma maneira de criar várias instâncias de um recurso durante a implantação, como a criação de várias NICs. Os modelos do Resource Manager usam arquivos JSON declarativos para definir o seu ambiente. Para saber mais, consulte [Visão geral do Azure Resource Manager](../../azure-resource-manager/resource-group-overview.md). Você usa *copiar* para especificar o número de instâncias a serem criadas:

```json
"copy": {
    "name": "multiplenics",
    "count": "[parameters('count')]"
}
```

Para obter mais informações, consulte [criando várias instâncias usando *cópia*](../../resource-group-create-multiple.md). 

Você também pode usar `copyIndex()` para acrescentar um número a um nome de recurso. Você pode criar *myNic1*, *MyNic2* e assim por diante. O código a seguir mostra um exemplo de como acrescentar o valor de índice:

```json
"name": "[concat('myNic', copyIndex())]", 
```

Você pode ler um exemplo completo em [criando várias NICs usando modelos do Resource Manager](../../virtual-network/template-samples.md).

Adicione rotas para NICs secundárias ao SO, concluindo as etapas em [Configurar o sistema operacional para várias NICs](#configure-guest-os-for-multiple-nics).

## <a name="configure-guest-os-for-multiple-nics"></a>Configurar o SO convidado para várias NICs

O Azure atribui um gateway padrão ao primeiro adaptador de rede (primário) anexado à máquina virtual. O Azure não atribui um gateway padrão aos adaptadores de rede adicionais (secundários) anexados à máquina virtual. Portanto, por padrão, não é possível se comunicar com os recursos fora da sub-rede na qual um adaptador de rede secundária se encontra. Entretanto, adaptadores de rede secundários podem se comunicar com recursos fora da sub-rede deles, embora as etapas para habilitar a comunicação sejam diferentes para os diversos sistemas operacionais.

1. Em um prompt de comando do Windows, execute o comando `route print`, que retorna uma saída semelhante à seguinte saída para uma máquina virtual com dois adaptadores de rede anexados:

    ```
    ===========================================================================
    Interface List
    3...00 0d 3a 10 92 ce ......Microsoft Hyper-V Network Adapter #3
    7...00 0d 3a 10 9b 2a ......Microsoft Hyper-V Network Adapter #4
    ===========================================================================
    ```
 
    Neste exemplo, o **Adaptador de Rede nº 4 do Microsoft Hyper-V** (interface 7) é o adaptador de rede secundário que não tem um gateway padrão atribuído a ele.

2. Em um prompt de comando, execute o comando `ipconfig` para ver qual endereço IP é atribuído ao adaptador de rede secundário. Neste exemplo, 192.168.2.4 está atribuído à interface 7. Nenhum endereço de gateway padrão é retornado para o adaptador de rede secundário.

3. Para rotear todo o tráfego destinado a endereços fora da sub-rede do adaptador de rede secundário para o gateway para a sub-rede, execute o seguinte comando:

    ```
    route add -p 0.0.0.0 MASK 0.0.0.0 192.168.2.1 METRIC 5015 IF 7
    ```

    O endereço de gateway para a sub-rede é o primeiro endereço IP (terminando em .1) no intervalo de endereços definido para a sub-rede. Se você não quiser para rotear todo o tráfego para fora da sub-rede, adicione rotas individuais para destinos específicos. Por exemplo, se você quiser rotear o tráfego do adaptador de rede secundário para a rede 192.168.3.0, digite o comando:

      ```
      route add -p 192.168.3.0 MASK 255.255.255.0 192.168.2.1 METRIC 5015 IF 7
      ```
  
4. Para confirmar a comunicação com êxito com um recurso na rede 192.168.3.0, por exemplo, digite o seguinte comando para executar o ping 192.168.3.4 usando a interface 7 (192.168.2.4):

    ```
    ping 192.168.3.4 -S 192.168.2.4
    ```

    Talvez seja necessário abrir o ICMP através do firewall do Windows do dispositivo no qual você está fazendo o ping com o seguinte comando:
  
      ```
      netsh advfirewall firewall add rule name=Allow-ping protocol=icmpv4 dir=in action=allow
      ```
  
5. Para confirmar se a rota adicionada está na tabela de rotas, insira o comando `route print`, que retorna uma saída semelhante ao seguinte texto:

    ```
    ===========================================================================
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
              0.0.0.0          0.0.0.0      192.168.1.1      192.168.1.4     15
              0.0.0.0          0.0.0.0      192.168.2.1      192.168.2.4   5015
    ```

    A rota listada com *192.168.1.1* em **Gateway** é aquela que existe por padrão para o adaptador de rede primário. A rota *192.168.2.1* em **Gateway** é aquela que você adicionou.

## <a name="next-steps"></a>Próximas etapas
Analise os [tamanhos de VM do Windows](sizes.md) quando estiver tentando criar uma VM que tem várias NICs. Preste atenção ao número máximo de NICs a que cada VM dá suporte. 


