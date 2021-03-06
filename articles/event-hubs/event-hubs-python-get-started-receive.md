---
title: Receber eventos dos Hubs de Eventos do Azure usando Python | Microsoft Docs
description: Comece a receber eventos de Hubs de Eventos usando Python
services: event-hubs
author: sethmanheim
manager: femila
ms.service: event-hubs
ms.workload: core
ms.topic: article
ms.date: 07/26/2018
ms.author: sethm
ms.openlocfilehash: f5388f2de599d94f68a1d24a7d701a2cb4795915
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/05/2018
ms.locfileid: "43703210"
---
# <a name="receive-events-from-event-hubs-using-python"></a>Receber eventos de Hubs de Eventos usando Python

Os Hubs de Eventos do Azure são um sistema gerenciamento de inclusão altamente dimensionável que pode receber milhões de eventos por segundo, permitindo que um aplicativo processe e analise grandes quantidades de dados produzidos por dispositivos conectados e outros sistemas. Depois de coletados em um hub de eventos, você pode receber e manipular eventos usando manipuladores em processo ou por encaminhamento para outros sistemas de análise.

Para saber mais sobre os Hubs de Eventos, consulte a [Visão geral dos Hubs de Eventos][Event Hubs overview].

Este tutorial descreve como receber eventos de um hub de eventos de um aplicativo escrito em Python. Para enviar eventos, consulte [o artigo de Envio correspondente](event-hubs-python-get-started-send.md).

O código deste tutorial é retirado [destes exemplos do GitHub](https://github.com/Azure/azure-event-hubs-python/tree/master/examples), que você pode examinar para ver todo o aplicativo de trabalho, incluindo instruções de importação e declarações de variável. Outros exemplos estão disponíveis na mesma pasta do GitHub.

## <a name="prerequisites"></a>Pré-requisitos

Para concluir este tutorial, você precisará dos seguintes pré-requisitos:

- Python 3.4 ou posterior.
- Um namespace dos Hubs de Eventos existente e Hub de Eventos do Azure. Você pode criar essas entidades seguindo as instruções deste [artigo](event-hubs-create.md). 

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]


## <a name="install-python-package"></a>Instalar o pacote do Python

Para instalar o pacote do Python para Hubs de Eventos, abra um prompt de comando que tenha Python no caminho e, em seguida, execute este comando: 

```bash
pip install azure-eventhub
```

## <a name="create-a-python-script-to-receive-events"></a>Criar um script do Python para receber eventos

Em seguida, crie um aplicativo do Python que receba eventos de um hub de eventos:

1. Abra seu editor de Python favorito, como o [Visual Studio Code][Visual Studio Code].
2. Crie um script chamado **recv.py**.
3. Cole o seguinte código em recv.py, substituindo os valores ADDRESS, USER e KEY pelos valores obtidos no portal do Azure na seção anterior: 

```python
import os
import sys
import logging
import time
from azure.eventhub import EventHubClient, Receiver, Offset

logger = logging.getLogger("azure")

# Address can be in either of these formats:
# "amqps://<URL-encoded-SAS-policy>:<URL-encoded-SAS-key>@<mynamespace>.servicebus.windows.net/myeventhub"
# "amqps://<mynamespace>.servicebus.windows.net/myeventhub"
# For example:
ADDRESS = "amqps://mynamespace.servicebus.windows.net/myeventhub"

# SAS policy and key are not required if they are encoded in the URL
USER = "RootManageSharedAccessKey"
KEY = "namespaceSASKey"
CONSUMER_GROUP = "$default"
OFFSET = Offset("-1")
PARTITION = "0"

total = 0
last_sn = -1
last_offset = "-1"
client = EventHubClient(ADDRESS, debug=False, username=USER, password=KEY)
try:
    receiver = client.add_receiver(CONSUMER_GROUP, PARTITION, prefetch=5000, offset=OFFSET)
    client.run()
    start_time = time.time()
    for event_data in receiver.receive(timeout=100):
        last_offset = event_data.offset
        last_sn = event_data.sequence_number
        print("Received: {}, {}".format(last_offset, last_sn))
        total += 1

    end_time = time.time()
    client.stop()
    run_time = end_time - start_time
    print("Received {} messages in {} seconds".format(total, run_time))

except KeyboardInterrupt:
    pass
finally:
    client.stop()
```

## <a name="receive-events"></a>Receber eventos

Para executar o script, abra um prompt de comando que tenha Python no caminho e, em seguida, execute este comando:

```bash
start python recv.py
```
 
## <a name="next-steps"></a>Próximas etapas

Para enviar eventos, consulte [o artigo de Envio correspondente](event-hubs-python-get-started-send.md).

Para saber mais sobre os Hubs de Eventos,visite as páginas a seguir:

* [Visão Geral dos Hubs de Eventos][Event Hubs overview]
* [Criar um hub de eventos](event-hubs-create.md)
* [Perguntas frequentes sobre os Hubs de Eventos](event-hubs-faq.md)

<!-- Links -->
[Event Hubs overview]: event-hubs-about.md
[Visual Studio Code]: https://code.visualstudio.com/
[free account]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
