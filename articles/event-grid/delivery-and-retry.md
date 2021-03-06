---
title: Entrega e repetição da Grade de Eventos do Azure
description: Descreve como a Grade de Eventos do Azure entrega eventos e como ela trata mensagens não entregues.
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: tomfitz
ms.openlocfilehash: 2a9ff23e5182c8cb7c91ad93e368f61f258c84f8
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/06/2018
ms.locfileid: "43841585"
---
# <a name="event-grid-message-delivery-and-retry"></a>Entrega e repetição de mensagens da Grade de Eventos 

Este artigo descreve como a Grade de Eventos do Azure manipula eventos quando a entrega não é confirmada.

A entrega proporcionada pela Grade de Eventos tem um tempo de duração. Cada mensagem é entregue pelo menos uma vez para cada assinatura. Os eventos são imediatamente enviados ao webhook registrado de cada assinatura. Se um webhook não confirmar o recebimento de um evento em até 60 segundos a contar da primeira tentativa de entrega, a Grade de Eventos repetirá a entrega do evento. 

Atualmente, a Grade de Eventos envia cada evento individualmente aos assinantes. O assinante recebe uma matriz com um único evento.

## <a name="message-delivery-status"></a>Status de entrega da mensagem

A Grade de Eventos usa códigos de resposta HTTP para confirmar o recebimento de eventos. 

### <a name="success-codes"></a>Códigos de êxito

Os códigos de resposta HTTP a seguir indicam que um evento foi entregue com êxito ao webhook. A Grade de Eventos considera entrega concluída.

- 200 OK
- 202 Aceito

### <a name="failure-codes"></a>Códigos de falha

Os códigos de resposta HTTP a seguir indicam que houve falha na tentativa de entrega do evento.

- 400 Solicitação Inválida
- 401 Não Autorizado
- 404 Não Encontrado
- 408 Tempo limite da solicitação
- Solicitação 413 entidade muito grande
- 414 URI muito longo
- 429 Número excessivo de solicitações
- 500 Erro Interno do Servidor
- 503 Serviço Indisponível
- 504 Tempo Limite do Gateway

Se você tiver [configurado um ponto final de letra morta](manage-event-delivery.md) e a Rede de Eventos receber um código de resposta 400 ou 413, a Grade de Eventos enviará imediatamente o evento ao ponto final de letra morta. Caso contrário, a grade de eventos repete todos os erros.

## <a name="retry-intervals-and-duration"></a>Intervalos de repetição e duração

A Grade de Eventos usa uma política de repetição de retirada exponencial para a entrega de eventos. Se o webhook não responder ou retornar um código de falha, a Grade de Eventos tentará novamente fazer a entrega no seguinte agendamento:

1. 10 segundos
2. 30 segundos
3. 1 minuto
4. 5 minutos
5. 10 minutos
6. 30 minutos
7. 1 hora

A Grade de Eventos adiciona uma pequena aleatoriedade a todos os intervalos de repetição. Após uma hora, a entrega de eventos é repetida a cada uma hora.

Por padrão, a Grade de Eventos expira todos os eventos que não são entregues em 24 horas. Você pode [personalizar a política de repetição](manage-event-delivery.md) ao criar uma assinatura de evento. Forneça o número máximo de tentativas de entrega (o padrão é 30) e a vida útil do evento (o padrão é 1440 minutos).

## <a name="dead-letter-events"></a>Eventos de mensagens mortas

Quando a Grade de Eventos não pode fornecer um evento, ela pode enviar o evento não entregue para uma conta de armazenamento. Este processo é conhecido como armazenamento de mensagens mortas. Para ver os eventos não entregues, você pode recebê-los do local de mensagens mortas. Para mais informações, consulte [Mensagens mortas e políticas de repetição](manage-event-delivery.md).

## <a name="next-steps"></a>Próximas etapas

* Para exibir o status de entregas de evento, consulte [Entrega de mensagens da Grade de Eventos do Monitor](monitor-event-delivery.md).
* Para personalizar as opções de entrega de eventos, consulte [Configurações de entrega de Gerenciamento da Grade de Eventos](manage-event-delivery.md).
* Para ver uma introdução à Grade de Eventos, confira [About Event Grid](overview.md) (Sobre a Grade de Eventos).
* Para começar a usar rapidamente a Grade de Eventos, confira [Create and route custom events with Azure Event Grid](custom-event-quickstart.md) (Criar e rotear eventos personalizados com a Grade de Eventos do Azure).