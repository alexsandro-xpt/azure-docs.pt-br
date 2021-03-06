---
title: Entenda o desconto de Instâncias de VM Reservadas do Azure | Microsoft Docs
description: Saiba como o desconto de Instância de VM Reservada do Azure é aplicado às máquinas virtuais em execução.
services: billing
documentationcenter: ''
author: yashesvi
manager: yashar
editor: ''
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/08/2018
ms.author: yashar
ms.openlocfilehash: de4a771672b45f930385c2c923b3935c65b6c045
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301639"
---
# <a name="understand-how-the-azure-reservation-discount-is-applied-to-virtual-machines"></a>Entenda como é aplicado o desconto em reserva do Azure a máquinas virtuais

Após uma Instância Reservada de Máquina Virtual do Azure, o desconto de reserva é aplicado automaticamente às máquinas virtuais correspondentes aos atributos e à quantidade da reserva. Uma reserva abrange os custos de infraestrutura das suas máquinas virtuais.

Para a capacidade reservada do Banco de Dados SQL do Microsoft Azure, consulte [Entenda o desconto de Instâncias Reservadas do Azure](billing-understand-reservation-charges.md).

A tabela a seguir ilustra os custos da máquina virtual após a compra de uma instância reservada de máquina virtual. Em todos os casos, você é cobrado pelo uso de armazenamento e rede com as taxas normais.

| Tipo de Máquina Virtual  | Encargos com a Instância Reservada de VM |
|-----------------------|--------------------------------------------|
|VMs Linux sem software adicional | A reserva abrange os custos de infraestrutura da sua VM.|
|VMs Linux com encargos de software (por exemplo, Red Hat) | A reserva abrange os custos de infraestrutura. Você é cobrado por software adicional.|
|VMs Windows sem software adicional |A reserva abrange os custos de infraestrutura. Você é cobrado por software do Windows.|
|VMs Windows com software adicional (por exemplo, SQL Server) | A reserva abrange os custos de infraestrutura. Você é cobrado para software Windows e por software adicional.|
|VMs Windows com [Benefício Híbrido do Azure](https://docs.microsoft.com/azure/virtual-machines/windows/hybrid-use-benefit-licensing) | A reserva abrange os custos de infraestrutura. Os custos de software Windows são cobertos pelo Benefício Híbrido do Azure. Qualquer software adicional é cobrado separadamente.| 

## <a name="application-of-reservation-discount-to-non-windows-vms"></a>Aplicação do desconto de reserva a VMs não - Windows

 O desconto de reserva do Azure é aplicado às instâncias de VM execução, por hora. Para a aplicação do desconto de reserva, as reservas compradas são comparadas ao uso emitido pelas VMs em execução. Para VMs que não podem ser executadas por uma hora completa, a reserva será preenchida com outras VMs que não estejam usando uma reserva, incluindo VMs em execução simultânea. Ao final da hora, o aplicativo de reserva para VMs na hora é bloqueado. No caso de uma máquina virtual não ser executada por uma hora ou VMs simultâneas em uma hora não preencherem a hora de reserva, a reserva é subutilizada nessa hora. O gráfico a seguir ilustra a aplicação de uma reserva ao uso faturável de VM. A ilustração baseia-se na compra de uma reserva e duas instâncias de VM correspondentes.

![Captura de tela de uma reserva aplicada e duas instâncias de VM correspondentes](media/billing-reserved-vm-instance-application/billing-reserved-vm-instance-application.png)

1. Qualquer uso acima da linha de reserva é cobrado com base nas taxas regulares pagas conforme o uso. Não há cobrança por qualquer uso abaixo das reservas, pois ele já foi pago como parte da compra de reserva.
2. Na hora 1, a instância 1 é executada por 0,75 hora e a instância 2, por 0,5 hora. O uso total relativo à hora 1 é de 1,25 hora. São cobradas as taxas de pagamento pelo uso da 0,25 hora restante.
3. Nas horas 2 e 3, ambas as instâncias foram executadas por 1 hora. Uma instância está coberta pela reserva e a outro é cobrada com as taxas de pagamento pelo uso.
4. Na hora 4, a instância 1 é executada por 0,5 hora e a instância 2, por 1 hora. A instância 1 está totalmente coberta pela reserva, e também 0,5 hora da instância 2. São cobradas as taxas de pagamento pelo uso da 0,5 hora restante.

Para entender e exibir o aplicativo das Reservas do Azure nos relatórios de uso de cobrança, consulte [Entender o uso de reserva](https://go.microsoft.com/fwlink/?linkid=862757).

## <a name="application-of-reservation-discount-to-windows-vms"></a>Aplicação do desconto de reserva a VMs Windows

Quando você executa instâncias de VM Windows, a reserva é aplicada para cobrir os custos de infraestrutura. A aplicação da reserva aos custos de infraestrutura de VMs é igual para máquinas virtuais Windows e não - Windows. Você é cobrado separadamente pelo software Windows por vCPU. Consulte [Custos do software Windows com Reservas](https://go.microsoft.com/fwlink/?linkid=862756). Você pode cobrir os custos de licenciamento do Windows com [Benefício Híbrido do Azure para Windows Server] (https://docs.microsoft.com/azure/virtual-machines/windows/hybrid-use-benefit-licensing).

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre as Reservas do Azure, consulte os seguintes artigos:

- [Quais são as Reservas do Azure?](billing-save-compute-costs-reservations.md)
- [Pré-pagamento para máquinas virtuais com instâncias de VMs reservadas do Azure](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Pagar antecipadamente por recursos de computação de banco de dados SQL com capacidade reservada do Azure SQL Database](../sql-database/sql-database-reserved-capacity.md)
- [Gerenciar Reservas do Azure](billing-manage-reserved-vm-instance.md)
- [Entender o uso de reserva para a sua assinatura paga conforme o uso](billing-understand-reserved-instance-usage.md)
- [Entenda o uso de reservas para o seu registro Enterprise](billing-understand-reserved-instance-usage-ea.md)
- [Entender o uso de reserva para assinaturas de CSP](https://docs.microsoft.com/partner-center/azure-reservations)
- [Custos de software do Windows não estão incluídos nas Reservas](billing-reserved-instance-windows-software-costs.md)

## <a name="need-help-contact-support"></a>Precisa de ajuda? Contate o suporte

Se ainda tiver dúvidas, [entre em contato com o suporte](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver seu problema rapidamente.
