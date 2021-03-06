---
title: Inscrição de autoatendimento ou de avaliação no Azure Active Directory | Microsoft Docs
description: Usar a inscrição por autoatendimento em um locatário do Azure AD (Azure Active Directory)
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: users-groups-roles
ms.topic: article
ms.workload: identity
ms.date: 01/28/2018
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: it-pro
ms.openlocfilehash: 99c5e99fa3bd33ef42e8df6ceba5be4be2cd1249
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/06/2018
ms.locfileid: "37869923"
---
# <a name="what-is-self-service-signup-for-azure-active-directory"></a>O que é inscrição por autoatendimento no Azure Active Directory?
Este artigo explica a inscrição por autoatendimento e como dar suporte a ela no Azure AD (Azure Active Directory). Se você desejar assumir um nome de domínio de um locatário do Azure AD não gerenciado, consulte [Take over an unmanaged directory as administrator](domains-admin-takeover.md) (Assumir um diretório não gerenciado como administrador).

## <a name="why-use-self-service-signup"></a>Por que usar a inscrição por autoatendimento?
* Leve aos clientes os serviços que eles desejam mais rápido
* Crie ofertas baseadas em email para um serviço
* Crie fluxos de inscrição baseados em email que permitem rapidamente que os usuários criem identidades usando seus aliases de email de trabalho fáceis de lembrar
* Um autoatendimento criado no diretório do Azure AD pode ser convertido em um diretório gerenciado que pode ser usado para outros serviços

## <a name="terms-and-definitions"></a>Termos e definições
* **Inscrição por autoatendimento**: esse é o método pelo qual um usuário se inscreve em um serviço de nuvem e tem uma identidade criada automaticamente para ele no Azure AD, com base no seu domínio de email.
* **Diretório do Azure AD não gerenciado**: este é o diretório em que a identidade é criada. Um diretório não gerenciado é um diretório sem nenhum administrador global.
* **Usuário verificado por email**: Este é um tipo de conta de usuário no AD do Azure. Um usuário que tem uma identidade criada automaticamente após a inscrição para uma oferta de autoatendimento é conhecido como um usuário verificado por email. Um usuário verificadas por email é um membro comum de um diretório marcado com creationmethod=EmailVerified.

## <a name="how-do-i-control-self-service-settings"></a>Como controlar as configurações de autoatendimento?
Os administradores têm dois controles de autoatendimento atualmente. Eles podem controlar se:

* Os usuários podem ingressar no diretório por email.
* Os usuários podem licenciar eles próprios para aplicativos e serviços.

### <a name="how-can-i-control-these-capabilities"></a>Como posso controlar esses recursos?
Um administrador pode configurar esses recursos usando os parâmetros Set-MsolCompanySettings do cmdlet do Azure AD a seguir:

* **AllowEmailVerifiedUsers** controla se um usuário pode criar ou ingressar em um diretório não gerenciado. Se você definir esse parâmetro como $false, nenhum usuário verificado por email poderá ingressar no diretório.
* **AllowAdHocSubscriptions** controla a capacidade de os usuários realizarem a inscrição por autoatendimento. Se você definir esse parâmetro como $false, nenhum usuário poderá realizar inscrição por autoatendimento. 
  
  > [!NOTE]
  > Assinaturas de avaliação do Flow e do PowerApps não são controladas pela configuração **AllowAdHocSubscriptions**. Para obter mais informações, consulte os seguintes artigos:
  > * [Como posso impedir meus usuários existentes de começarem a usar o Power BI?](https://support.office.com/article/Power-BI-in-your-Organization-d7941332-8aec-4e5e-87e8-92073ce73dc5#bkmk_preventjoining)
  > * [Perguntas e respostas sobre o Flow na sua organização](https://docs.microsoft.com/flow/organization-q-and-a)

### <a name="how-do-the-controls-work-together"></a>Como os controles funcionam juntos?
Esses dois parâmetros podem ser usados em conjunto para definir um controle mais preciso sobre a inscrição por autoatendimento. Por exemplo, o comando a seguir permitirá que os usuários realizem a inscrição por autoatendimento, mas apenas se os usuários já tiverem uma conta no Azure AD (em outras palavras, os usuários que precisassem criar primeiro uma conta verificada por email não poderiam realizar inscrição por autoatendimento):

````
    Set-MsolCompanySettings -AllowEmailVerifiedUsers $false -AllowAdHocSubscriptions $true
````
O fluxograma a seguir explica as diferentes combinações desses parâmetros e as condições resultantes para o diretório e para a inscrição por autoatendimento.

![][1]

Para obter mais informações e exemplos de como usar esses parâmetros, consulte [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings?view=azureadps-1.0).

## <a name="next-steps"></a>Próximas etapas
* [Adicionar um nome de domínio personalizado ao Azure AD](../fundamentals/add-custom-domain.md)
* [Como instalar e configurar o PowerShell do Azure](/powershell/azure/overview)
* [PowerShell do Azure](/powershell/azure/overview)
* [Referência de Cmdlets do Azure](/powershell/azure/get-started-azureps)
* [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings?view=azureadps-1.0)

<!--Image references-->
[1]: ./media/directory-self-service-signup/SelfServiceSignUpControls.png
