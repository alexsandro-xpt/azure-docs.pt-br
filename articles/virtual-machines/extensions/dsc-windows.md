---
title: Manipulador de extensão da Configuração de Estado Desejado do Azure | Microsoft Docs
description: Atualizar e aplicar uma configuração de DSC do PowerShell em uma VM do Azure usando a extensão DSC
services: virtual-machines-windows
documentationcenter: ''
author: bobbytreed
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: windows
ms.workload: ''
ms.date: 03/26/2018
ms.author: robreed
ms.openlocfilehash: b9e96473a6f66dcbc675da1553deaed4ad61b249
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/15/2018
ms.locfileid: "45630914"
---
# <a name="powershell-dsc-extension"></a>Extensão de DSC do PowerShell

## <a name="overview"></a>Visão geral

A extensão DSC PowerShell para Windows é publicada e recebe suporte da Microsoft. A extensão atualiza e aplica uma configuração de DSC do PowerShell em uma VM do Azure. A extensão de DSC chama a DSC do PowerShell para aplicar a configuração DSC recebida na VM. Este documento detalha as plataformas com opções de plataformas, configurações e implantação com suporte para a extensão da máquina virtual do DSC para Windows.

## <a name="prerequisites"></a>Pré-requisitos

### <a name="operating-system"></a>Sistema operacional

A Extensão DSC suporta os seguintes OS

Windows Server 2016, Windows Server 2012R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1

### <a name="internet-connectivity"></a>Conectividade com a Internet

A extensão do agente DSC para Windows requer que a máquina virtual de destino esteja conectada à Internet. 

## <a name="extension-schema"></a>Esquema de extensão

O JSON a seguir mostra o esquema que serve para a parte das configurações da extensão DSC em um modelo do Azure Resource Manager. 

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "Microsoft.Powershell.DSC",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "properties": {
    "publisher": "Microsoft.Powershell",
    "type": "DSC",
    "typeHandlerVersion": "2.73",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "wmfVersion": "latest",
        "configuration": {
            "url": "http://validURLToConfigLocation",
            "script": "ConfigurationScript.ps1",
            "function": "ConfigurationFunction"
        },
        "configurationArguments": {
            "argument1": "Value1",
            "argument2": "Value2"
        },
        "configurationData": {
            "url": "https://foo.psd1"
        },
        "privacy": {
            "dataCollection": "enable"
        },
        "advancedOptions": {
            "forcePullAndApply": false
            "downloadMappings": {
                "specificDependencyKey": "https://myCustomDependencyLocation"
            }
        } 
    },
    "protectedSettings": {
        "configurationArguments": {
            "parameterOfTypePSCredential1": {
                "userName": "UsernameValue1",
                "password": "PasswordValue1"
            },
            "parameterOfTypePSCredential2": {
                "userName": "UsernameValue2",
                "password": "PasswordValue2"
            }
        },
        "configurationUrlSasToken": "?g!bber1sht0k3n",
        "configurationDataUrlSasToken": "?dataAcC355T0k3N"
    }
  }
}
```

### <a name="property-values"></a>Valores de propriedade

| NOME | Valor/Exemplo | Tipo de Dados |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | data |
| publicador | Microsoft.Powershell.DSC | string |
| Tipo | DSC | string |
| typeHandlerVersion | 2.73 | int |

### <a name="settings-property-values"></a>Valores da Propriedade de Configurações

| NOME | Tipo de Dados | DESCRIÇÃO
| ---- | ---- | ---- |
| settings.wmfVersion | string | Especifica a versão do Windows Management Framework que deve ser instalada em sua VM. Configurar essa propriedade como 'latest' instalará a versão mais atualizada do WMF. Os únicos valores possíveis atualmente para essa propriedade são ‘4.0’, ‘5.0’, e a mais recente. Esses valores possíveis estão sujeitos a atualizações. O valor padrão é ‘latest’. |
| settings.configuration.url | string | Especifica o local da URL de onde baixar o arquivo zip configuração DSC. Se a URL fornecida exigir um token SAS para acesso, será necessário definir a propriedade protectedSettings.configurationUrlSasToken como o valor do token de SAS. Esta propriedade será necessária se settings.configuration.script e/ou settings.configuration.function estiverem definidas.
| settings.configuration.script | string | Especifica o nome do arquivo do script que contém a definição de sua configuração DSC. Esse script deve estar na pasta raiz do arquivo zip baixado da URL especificada pela propriedade configuration.url. Esta propriedade é necessária se settings.configuration.url e/ou settings.configuration.script estiverem definidas.
| settings.configuration.function | string | Especifica o nome da configuração DSC. A configuração denominada deve estar contida no script definido por configuration.script. Esta propriedade será necessária se settings.configuration.script.url e/ou settings.configuration.function estiverem definidas.
| settings.configurationArguments | Coleção | Define os parâmetros que você deseja passar para a configuração de DSC. Esta propriedade não será criptografada.
| settings.configurationData.url | string | Especifica a URL de onde baixar o arquivo de dados de configuração (.pds1) para usar como entrada para a sua configuração de DSC. Se a URL fornecida exigir um token SAS para acesso, será necessário definir a propriedade protectedSettings.configurationDataUrlSasToken como o valor do token de SAS.
| settings.privacy.dataEnabled | string | Habilita ou desabilita a coleta de telemetria. Os únicos valores possíveis para essa propriedade são ‘Enable’, ‘Disable’, ”, ou $null. Deixar esta propriedade em branco ou nulo permitirá telemetria
| settings.advancedOptions.forcePullAndApply | Bool | Permite à extensão de DSC atualizar e aplicar configurações DSC quando o modo de atualização for Pull.
| settings.advancedOptions.downloadMappings | Coleção | Define locais alternativos para fazer o download de dependências como WMF e .NET

### <a name="protected-settings-property-values"></a>Valores da Propriedade de Configurações Protegidos

| NOME | Tipo de Dados | DESCRIÇÃO
| ---- | ---- | ---- |
| protectedSettings.configurationArguments | string | Define os parâmetros que você deseja passar para a configuração de DSC. Esta propriedade será criptografada. |
| protectedSettings.configurationUrlSasToken | string | Especifica o token SAS para acessar a URL definida por configuration.url. Esta propriedade será criptografada. |
| protectedSettings.configurationDataUrlSasToken | string | Especifica o token SAS para acessar a URL definida por configurationData.url. Esta propriedade será criptografada. |


## <a name="template-deployment"></a>Implantação de modelo

Extensões de VM do Azure podem ser implantadas com modelos do Azure Resource Manager. Modelos são ideais ao implantar uma ou mais máquinas virtuais que exigem configuração pós-implantação. Um modelo do Resource Manager de exemplo que inclui a extensão de VM do agente do OMS pode ser encontrado na [Galeria de Início Rápido do Azure](https://github.com/Azure/azure-quickstart-templates/tree/052db5feeba11f85d57f170d8202123511f72044/dsc-extension-iis-server-windows-vm). 

A configuração do JSON para uma extensão da máquina virtual pode ser aninhado dentro do recurso de máquina virtual ou localizado no nível de raiz ou superior de um modelo JSON do Resource Manager. O posicionamento da configuração do JSON afeta o valor do tipo e nome do recurso. 

Ao aninhar o recurso de extensão, o JSON é colocado no objeto `"resources": []` da máquina virtual. Ao inserir o JSON da extensão na raiz do modelo, o nome do recurso inclui uma referência na máquina virtual pai e o tipo reflete a configuração aninhada.  


## <a name="azure-cli-deployment"></a>Implantação da CLI do Azure

A CLI do Azure pode ser usado para implantar a extensão da VM do Agente do OMS para uma máquina virtual existente. Substitua a chave e a ID do OMS por aqueles de seu espaço de trabalho do OMS. 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name Microsoft.Powershell.DSC \
  --publisher Microsoft.Powershell \
  --version 2.73 --protected-settings '{}' \
  --settings '{}'
```

## <a name="troubleshoot-and-support"></a>Solução de problemas e suporte

### <a name="troubleshoot"></a>Solucionar problemas

Dados sobre o estado das implantações de extensão podem ser recuperados do Portal do Azure usando a CLI do Azure. Para ver o estado da implantação das extensões de uma determinada VM, execute o comando a seguir usando a CLI do Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Pacote de extensão é baixado e implantado para esse local na VM do Azure
```
C:\Packages\Plugins\{Extension_Name}\{Extension_Version}
```

Arquivo de extensão de status contém o status de sub e códigos de status de acertos/erros junto com o erro detalhado e a descrição para cada extensão de execução.
```
C:\Packages\Plugins\{Extension_Name}\{Extension_Version}\Status\{0}.Status  -> {0} being the sequence number
```

Logs de saída de extensão são registrados no seguinte diretório:

```
C:\WindowsAzure\Logs\Plugins\{Extension_Name}\{Extension_Version}
```

### <a name="error-codes-and-their-meanings"></a>Códigos de erro e seus significados

| Código do Erro | Significado | Ação possível |
| :---: | --- | --- |
| 1000 | Erro genérico | A mensagem de erro é fornecida pela exceção específica em logs de extensão |
| 52 | Erro de Instalação da Extensão | A mensagem de erro é fornecida pela exceção específica |
| 1002 | Erro de instalação Wmf | Erro ao instalar WMF. |
| 1004 | Pacote Zip Inválido | Zip inválido ; Erro ao desempacotar o zip |
| 1100 | Erro de Argumento | Indica um problema na entrada fornecida pelo usuário. A mensagem de erro é fornecida pela exceção específica|


### <a name="support"></a>Suporte

Caso precise de mais ajuda em qualquer ponto deste artigo, entre em contato com os especialistas do Azure nos [fóruns do Azure e do Stack Overflow no MSDN](https://azure.microsoft.com/support/forums/). Como alternativa, você pode registrar um incidente de suporte do Azure. Vá para o [site de suporte do Azure](https://azure.microsoft.com/support/options/) e selecione Obter suporte. Para saber mais sobre como usar o suporte do Azure, leia as [Perguntas frequentes sobre o suporte do Microsoft Azure](https://azure.microsoft.com/support/faq/).