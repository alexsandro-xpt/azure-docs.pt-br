---
title: Conectar ao OneDrive – Aplicativo Lógico do Azure | Microsoft Docs
description: Carregar e gerenciar arquivos com as APIs REST do OneDrive e o Aplicativo Lógico do Azure
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 10/18/2016
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 50bd9ecdd665cf72c146c63ae25efa6773934a3e
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35295373"
---
# <a name="get-started-with-the-onedrive-connector"></a>Introdução ao conector do OneDrive
Conecte-se ao OneDrive para gerenciar seus arquivos, incluindo carregar, obter, excluir arquivos e muito mais. 

Com o OneDrive, você: 

* Compile seu fluxo de trabalho armazenando os arquivos no OneDrive ou atualize os arquivos existentes no OneDrive. 
* Use gatilhos para iniciar o fluxo de trabalho quando um arquivo é criado ou atualizado em seu OneDrive.
* Usar ações para criar um arquivo, excluir um arquivo e muito mais. Por exemplo, quando um novo email do Office 365 for recebido com um anexo (um gatilho), crie um novo arquivo no OneDrive (uma ação).

Este artigo mostra como usar o conector do OneDrive em um aplicativo lógico, além de listar os gatilhos e as ações.

Para saber mais sobre os Aplicativos Lógicos, consulte [O que são aplicativos lógicos](../logic-apps/logic-apps-overview.md) e [Criar um aplicativo lógico](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="connect-to-onedrive"></a>Conectar o OneDrive
Antes do aplicativo lógico poder acessar qualquer serviço, primeiro crie uma *conexão* com o serviço. Uma conexão fornece uma conectividade entre um aplicativo lógico e outro serviço. Por exemplo, para se conectar ao OneDrive, primeiramente é necessária uma *conexão* do OneDrive. Para criar uma conexão, insira as credenciais que você normalmente usa para acessar o serviço ao qual deseja conectar-se. Assim, com o OneDrive, insira as credenciais de sua conta OneDrive para criar a conexão.

### <a name="create-the-connection"></a>Criar a conexão
> [!INCLUDE [Steps to create a connection to OneDrive](../../includes/connectors-create-api-onedrive.md)]
> 
> 

## <a name="use-a-trigger"></a>Usar um gatilho
Um gatilho é um evento que pode ser usado para iniciar o fluxo de trabalho definido em um aplicativo lógico. Gatilhos "sondam" o serviço no intervalo e na frequência desejados. [Saiba mais sobre gatilhos](../logic-apps/logic-apps-overview.md#logic-app-concepts).

1. No aplicativo lógico, digite "onedrive" para obter uma lista de gatilhos:  
   
    ![](./media/connectors-create-api-onedrive/onedrive-1.png)
2. Escolha **Quando um arquivo é modificado**. Se já existe uma conexão, escolha o botão Mostrar Seletor para selecionar uma pasta.
   
    ![](./media/connectors-create-api-onedrive/sample-folder.png)
   
    Se você for solicitado a entrar, insira os detalhes do logon para criar a conexão. [Criar a conexão](connectors-create-api-onedrive.md#create-the-connection) neste artigo lista as etapas. 
   
   > [!NOTE]
   > Neste exemplo, o aplicativo lógico é executado quando um arquivo na pasta escolhida é atualizado. Para ver os resultados deste gatilho, adicione outra ação que envia um email. Por exemplo, adicione a ação *Enviar um email* do Outlook do Office 365 que envia um email para você quando um arquivo é atualizado. 

3. Selecione o botão **Editar** e defina os valores **Frequência** e **Intervalo**. Por exemplo, se você quiser que o gatilho faça uma sondagem a cada 15 minutos, defina a **Frequência** como **Minuto** e **Intervalo** como **15**. 
   
    ![](./media/connectors-create-api-onedrive/trigger-properties.png)
4. **Salve** as alterações (canto superior esquerdo da barra de ferramentas). Seu aplicativo lógico é salvo e pode ser habilitado automaticamente.

## <a name="use-an-action"></a>Usar uma ação
Uma ação é uma operação executada pelo fluxo de trabalho definido em um aplicativo lógico. [Saiba mais sobre ações](../logic-apps/logic-apps-overview.md#logic-app-concepts).

1. Selecione o sinal de mais. Você tem várias opções: **adicionar uma ação**, **adicionar uma condição** ou uma das opções **Mais**.
   
    ![](./media/connectors-create-api-onedrive/add-action.png)
2. Escolha **Adicionar uma ação**.
3. Na caixa de texto, digite "onedrive" para obter uma lista de todas as ações disponíveis.
   
    ![](./media/connectors-create-api-onedrive/onedrive-actions.png) 
4. Em nosso exemplo, escolha **OneDrive – criar arquivo**. Se já existir uma conexão, escolha o **Caminho da Pasta** para colocar o arquivo, insira o **Nome do Arquivo** e escolha o **Conteúdo do Arquivo** desejado:  
   
    ![](./media/connectors-create-api-onedrive/sample-action.png)
   
    Se as informações de conexão forem solicitadas, insira os detalhes para criar a conexão. [Criar a conexão](connectors-create-api-onedrive.md#create-the-connection) neste artigo descreve essas propriedades. 
   
   > [!NOTE]
   > Neste exemplo, podemos criar um novo arquivo em uma pasta do OneDrive. Você pode usar a saída de outro gatilho para criar o arquivo do OneDrive. Por exemplo, adicione o gatilho *Quando um novo email chegar* do Outlook do Office 365. Em seguida, adicione a ação *Criar arquivo* do OneDrive, que usa os campos Anexos e Tipo de Conteúdo em um ForEach para criar o novo arquivo no OneDrive. 
   > 
   > ![](./media/connectors-create-api-onedrive/foreach-action.png)

5. **Salve** as alterações (canto superior esquerdo da barra de ferramentas). Seu aplicativo lógico é salvo e pode ser habilitado automaticamente.


## <a name="connector-specific-details"></a>Detalhes específicos do conector

Exiba os gatilhos e ações definidos no swagger e também os limites nos [detalhes do conector](/connectors/onedriveconnector/).

## <a name="more-connectors"></a>Mais conectores
Volte para a [Lista de APIs](apis-list.md).