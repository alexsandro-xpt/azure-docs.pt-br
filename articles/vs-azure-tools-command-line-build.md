---
title: Build de linha de comando do Azure| Microsoft Docs
description: Compilação de linha de comando do Azure
services: visual-studio-online
author: ghogen
manager: douge
assetId: 94b35d0d-0d35-48b6-b48b-3641377867fd
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/05/2017
ms.author: ghogen
ms.openlocfilehash: c47da12feeef2a1536cdbfbe0187fe8991b3257d
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141619"
---
# <a name="building-azure-projects-from-the-command-line"></a>Criação de projetos do Azure na linha de comando
Ao usar o Microsoft Build Engine (MSBuild), você pode criar produtos nos ambientes de laboratório de criação em que o Visual Studio não está instalado. O MSBuild usa um formato XML para arquivos de projeto extensíveis e com suporte total da Microsoft. Usando o formato de arquivo MSBuild, é possível descrever quais itens devem ser criados para uma ou mais plataformas e configurações.

Você também pode executar o MSBuild na linha de comando, e este tópico descreve essa abordagem. Ao configurar propriedades na linha de comando, você pode criar configurações específicas de um projeto. Da mesma forma, também é possível definir os destinos que o comando MSBuild cria. Para saber mais sobre parâmetros de linha de comando e o MSBuild, confira [Referência de linha de comando MSBuild](https://msdn.microsoft.com/library/ms164311.aspx).

## <a name="msbuild-parameters"></a>Parâmetros do MSBuild
A maneira mais simples de criar um pacote é executar o MSBuild com a opção `/t:Publish` . Por padrão, este comando cria um diretório em relação à pasta raiz do projeto, como `<ProjectDirectory>\bin\Configuration\app.publish\`. Quando você cria um projeto do Azure, dois arquivos são gerados: o arquivo de pacote em si e o arquivo de configuração que o acompanha:

* Arquivo de pacote (`project.cspkg`)
* Arquivo de configuração (`ServiceConfiguration.TargetProfile.cscfg`)

Por padrão, cada projeto do Azure inclui um arquivo de configuração de serviço para criações locais (depuração) e outro para criações na nuvem (preparo ou produção). No entanto, você pode adicionar ou remover arquivos de configuração de serviço conforme necessário. Ao compilar um pacote no Visual Studio, você define o arquivo de configuração de serviço a ser incluído com o pacote. Quando você compila um pacote usando o MSBuild, o arquivo de configuração de serviço local é incluído por padrão. Para incluir um arquivo de configuração de serviço diferente, defina a propriedade `TargetProfile` do comando MSBuild (`MSBuild /t:Publish /p:TargetProfile=ProfileName`).

Se você quiser usar um diretório alternativo para o pacote armazenado e arquivos de configuração, defina o caminho usando a opção `/p:PublishDir=Directory\`, incluindo o separador de barra invertida à direita.

## <a name="next-steps"></a>Próximas etapas
Depois que o pacote é compilado, você pode implantá-lo no Azure.