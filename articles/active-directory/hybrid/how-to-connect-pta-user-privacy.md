---
title: Privacidade do usuário e autenticação de passagem do Azure Active Directory | Microsoft Docs
description: Este artigo trata da Autenticação de Passagem do Microsoft Azure AD (Azure Active Directory) e da conformidade com GDPR.
services: active-directory
keywords: A Autenticação de Passagem do Azure AD Connect, GDPR, requer componentes necessários para o Microsoft Azure AD, SSO, Logon Único
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/23/2018
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: bcb448296313eedebb8f8389b320c5da5f1a80dd
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/19/2018
ms.locfileid: "46305645"
---
# <a name="user-privacy-and-azure-active-directory-pass-through-authentication"></a>Privacidade do usuário e autenticação de passagem do Azure Active Directory


[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

## <a name="overview"></a>Visão geral

A Autenticação de Passagem do Azure AD cria o seguinte tipo de log, que pode conter dados pessoais:

- Arquivos de log de rastreamento do Azure AD Connect.
- Arquivos de log de rastreamento do Agente de Autenticação.
- Arquivos de log de Eventos do Windows.

É possível aumentar a privacidade do usuário na Autenticação de Passagem de duas maneiras:

1.  Mediante solicitação, extraia dados de uma pessoa e remova os dados dessa pessoa das instalações.
2.  Certifique-se de que nenhum dado é retido além de 48 horas.

É altamente recomendável a segunda opção, pois é mais fácil de implementar e manter. A seguir, estão as instruções para cada tipo de log:

### <a name="delete-azure-ad-connect-trace-log-files"></a>Arquivos de log de rastreamento do Azure AD Connect

Verifique o conteúdo da pasta **%ProgramData%\AADConnect** e exclua o conteúdo do log de rastreamento (arquivos **trace-\*.log**) dessa pasta dentro de 48 horas após instalar ou atualizar o Azure AD Connect ou modificar a configuração de Autenticação de Passagem, pois essa ação pode criar dados cobertos pelo GDPR.

>[!IMPORTANT]
>Não exclua o arquivo **PersistedState.xml** nessa pasta, pois esse arquivo será utilizado para manter o estado da instalação anterior do Azure AD Connect e quando uma instalação de upgrade for feita. Esse arquivo nunca conterá dados sobre uma pessoa e nunca deverá ser excluído.

É possível revisar e excluir esses arquivos de log de rastreamento utilizando o Windows Explorer ou usar o script a seguir do PowerShell para executar as ações necessárias:

```
$Files = ((Get-Item -Path "$env:programdata\aadconnect\trace-*.log").VersionInfo).FileName 
 
Foreach ($file in $Files) { 
    {Remove-Item -Path $File -Force} 
}
```

Salve o script em um arquivo com a extensão ".PS1". Execute esse script, conforme necessário.

Para saber mais sobre os requisitos do GDPR do Azure AD Connect relacionados, consulte [este artigo](reference-connect-user-privacy.md).

### <a name="delete-authentication-agent-event-logs"></a>Excluir logs de eventos do Agente de Autenticação

Este produto também pode criar **Logs de Eventos do Windows**. Para saber mais, leia [este artigo](https://msdn.microsoft.com/library/windows/desktop/aa385780(v=vs.85).aspx).

Para exibir logs relacionados ao Agente de Autenticação de Passagem, abra o aplicativo **Visualizador de Eventos** no servidor e verifique em **Application and Service Logs\Microsoft\AzureAdConnect\AuthenticationAgent\Admin**.

### <a name="delete-authentication-agent-trace-log-files"></a>Excluir arquivos de log de rastreamento do Agente de Autenticação

É necessário verificar regularmente o conteúdo de **%ProgramData%\Microsoft\Azure AD Connect Authentication Agent\Trace\** e excluir o conteúdo dessa pasta a cada 48 horas. 

>[!IMPORTANT]
>Se o Serviço de Autenticação do Agente estiver em execução, não será possível excluir o arquivo de log atual na pasta. Pare o serviço antes de tentar novamente. Para evitar falhas de entrada do usuário, é necessário ter configurado a Autenticação de Passagem para [alta disponibilidade](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability).

Você pode revisar e excluir esses arquivos, utilizando o Windows Explorer ou usar o script a seguir para executar as ações necessárias:

```
$Files = ((Get-childitem -Path "$env:programdata\microsoft\azure ad connect authentication agent\trace" -Recurse).VersionInfo).FileName 
 
Foreach ($file in $files) { 
    {Remove-Item -Path $File -Force} 
}
```

Para agendar esse script para ser executado a cada 48 horas, siga estas etapas:

1.  Salve o script em um arquivo com a extensão ".PS1".
2.  Abra o **Painel de Controle** e clique em **Sistema e Segurança**.
3.  No cabeçalho **Ferramentas Administrativas**, clique em “**Agendar Tarefas**”.
4.  Em **Agendador de Tarefas**, clique com o botão direito do mouse em “**Biblioteca de Agendamento de Tarefas**” e clique em “**Criar tarefa básica…**”.
5.  Insira o nome para a nova tarefa e clique em **Avançar**.
6.  Selecione “**Diariamente**” para o **Disparador de Tarefa** e clique em **Avançar**.
7.  Defina a recorrência para dois dias e clique em **Avançar**.
8.  Selecione “**Iniciar um programa**” como a ação e clique em **Avançar**.
9.  Digite “**PowerShell**” na caixa para o programa/script e, na caixa rotulada “**Adicionar argumentos (opcional)**”, insira o caminho completo para o script que você criou anteriormente e clique em **Avançar**.
10. A próxima tela mostra um resumo da tarefa que você está prestes a criar. Verifique os valores e clique em **Concluir** para criar a tarefa:
 
### <a name="note-about-domain-controller-logs"></a>Observação sobre os logs do Controlador de Domínio

Se o log de auditoria estiver habilitado, esse produto poderá gerar logs de segurança para os Controladores de Domínio. Para saber mais sobre como configurar políticas de auditoria, leia este [artigo](https://technet.microsoft.com/library/dd277403.aspx).

## <a name="next-steps"></a>Próximas etapas
* [Revise a Política de Privacidade da Microsoft na Central de Confiabilidade](https://www.microsoft.com/trustcenter)
- [**Solução de problemas**](tshoot-connect-pass-through-authentication.md) – Saiba como resolver problemas comuns do recurso.
