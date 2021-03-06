---
title: O que é o Azure Active Directory B2C? | Microsoft Docs
description: Saiba mais sobre como você pode criar e gerenciar sua conexão de aplicativo usando o Azure Active Directory B2C.
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 04/05/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 6949ab89cf806818783c86199e6df334e263b046
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/03/2018
ms.locfileid: "37440874"
---
# <a name="what-is-azure-active-directory-b2c"></a>O que é o Azure Active Directory B2C?

O Azure Active Directory (Azure AD) B2C é um serviço de gerenciamento de identidade que permite personalizar e controlar como os clientes se inscrevem, entram e gerenciam seus perfis ao usar seus aplicativos. Isso inclui aplicativos desenvolvidos para iOS, Android e .NET, entre outros. O Azure AD B2C habilita essas ações enquanto protege as identidades dos seus clientes ao mesmo tempo.

Você pode configurar um aplicativo registrado com o Azure AD B2C para executar uma variedade de ações de gerenciamento de identidade. Alguns exemplos incluem:

- Habilitar um cliente para entrar para usar o aplicativo registrado
- Habilitar um cliente inscrito entrar e começar a usar seu aplicativo
- Habilitar um cliente inscrito para editar seu perfil
- Habilitar a autenticação multifator em seu aplicativo
- Habilitar o cliente para inscrever-se e entrar com provedores de identidade específico
- Conceder acesso do aplicativo para APIs que você criar
- Personalizar a aparência da experiência de inscrição e entrar
- Gerenciar sessões de logon único para o seu aplicativo

## <a name="what-do-i-need-to-think-about-before-using-azure-ad-b2c"></a>O que é necessário pensar antes de usar o Azure AD B2C?

- Como fazer com que o cliente interaja com o meu aplicativo?
- O que é a experiência de interface do usuário que eu quero fornecer para os clientes?
- Quais provedores de identidade eu quero permitir que os clientes escolham a partir do meu aplicativo?
- O processo de entrada exige a execução de APIs adicionais?

### <a name="customer-interaction"></a>Interação do cliente

O Azure AD B2C oferece suporte a [OpenID Connect](https://openid.net/connect/) para todas as experiências do cliente. Na implementação do Azure AD B2C do OpenID Connect, seu aplicativo inicia esse percurso emitindo solicitações de autenticação para o Azure AD B2C. O resultado da solicitação é um `id_token`. Esse token de segurança representa a identidade do cliente.

Cada aplicativo que usa o Azure AD B2C deve ser registrado em um locatário Azure AD B2C usando o Portal do Azure. O processo de registro do aplicativo coleta e atribui valores ao seu aplicativo. Esses valores incluem uma ID de aplicativo que identifica exclusivamente o aplicativo e um URI que pode ser usado para direcionar as respostas para ele de redirecionamento.

A interação de cada aplicativo segue um padrão semelhante de alto nível:

1. O aplicativo direciona o cliente para executar uma política.
2. O cliente conclui a política de acordo com a definição de política.
3. O aplicativo recebe um token de segurança.
4. O aplicativo usa o token de segurança para tentar acessar ou um recurso protegido.
5. O servidor de recurso valida o token de segurança para verificar se o acesso pode ser concedido.
6. O aplicativo atualiza periodicamente o token de segurança.

Essas etapas podem variar um pouco com base no tipo de aplicativo que você está compilando.

O Azure AD B2C interage com provedores de identidade, clientes e outros sistemas e o diretório local em sequência para concluir uma tarefa de identidade. Por exemplo, entrar em um cliente, registrar um novo cliente ou redefinir uma senha. A plataforma subjacente que estabelece a relação de confiança de várias partes e conclui essas etapas é chamada de Estrutura de Experiência de Identidade. Essa estrutura e uma política (também chamada de percurso do usuário ou política Estrutura Confiável) define explicitamente os atores, as ações, os protocolos e a sequência de etapas para concluir.

O Azure AD B2C protege de ataque de negação de serviço e senha contra seus aplicativos de várias formas. O Azure AD B2C usa técnicas de detecção e de mitigação como cookies SYN e limites de taxa e de conexão para proteger os recursos contra esses ataques de negação de serviço. A mitigação também inclui ataques de senhas de força bruta e ataques de senhas de dicionário.

#### <a name="built-in-policies"></a>Políticas internas

Cada solicitação enviada para o Azure AD B2C especifica uma política. Uma política controla o comportamento de como seu aplicativo interage com o Azure AD B2C. Políticas internas são predefinidas para tarefas mais comuns de identidade, como se inscrever, entrar e editar perfil.  Por exemplo, uma política de inscrição permite controlar comportamentos definindo as seguintes configurações:

- Contas sociais que o usuário pode usar para entrar no aplicativo
- Dados coletados do cliente, como nome ou código postal
- Autenticação multifator
- Aparência e funcionalidade de todas as páginas de inscrição
- Informações retornadas para o aplicativo

#### <a name="custom-policies"></a>Políticas personalizadas 

As [politicas personalizadas](active-directory-b2c-overview-custom.md) são arquivos de configuração que definem o comportamento da Estrutura de Experiência de Identidade em seu locatário do Azure AD B2C. Políticas personalizadas podem ser editadas totalmente para realizar muitas tarefas. Uma política personalizada é representada como um ou vários arquivos formatados em XML que se referenciam entre si em uma cadeia hierárquica. 

Várias políticas personalizadas de tipos diferentes podem ser usadas em seu locatário do Azure AD B2C conforme necessário e podem ser reutilizadas em aplicativos. Essa flexibilidade permite definir e modificar experiências de identidade do consumidor com pouca ou nenhuma alteração no seu código. As políticas podem ser executadas usando um parâmetro de consulta especial nas solicitações de autenticação HTTP.

Políticas personalizadas podem ser usadas para controlar os percursos de usuário das seguintes maneiras:

- Definindo a interação com as APIs para capturar informações adicionais, verifique as declarações fornecidas pelo cliente ou dispare processos externos.
- Alterando o comportamento baseado em declarações de APIs ou de declarações no diretório como *migrationStatus*.
- Qualquer fluxo de trabalho não abrangido por políticas internas. Por exemplo, a coleta de informações de mais de um cliente durante uma experiência de logon e executar uma verificação de autorização para acessar um recurso.

### <a name="identity-providers"></a>Provedores de identidade

Um provedor de identidade é um serviço que autentica as identidades dos usuários e emite tokens de segurança. No Azure AD B2C, você pode configurar um número de provedores de identidade em seu locatário, como uma conta da Microsoft, Facebook ou Amazon entre outros. 

Para configurar um provedor de identidade em seu locatário do Azure AD B2C, registre o identificador de aplicativo ou um identificador de cliente e o segredo do cliente ou a senha do aplicativo de provedor de identidade que você criar. O identificador e a senha são então usados para configurar seu aplicativo.

### <a name="user-interface-experience"></a>Experiência de Interface do Usuário

A maioria do conteúdo HTML e CSS que é apresentado aos clientes pode ser controlada. Usando o recurso de personalização da interface do usuário da página, você pode personalizar a aparência de qualquer política. Também pode manter a consistência visual e da marca entre seu aplicativo e o Azure AD B2C.

O Azure AD B2C executa o código no navegador do cliente e usa uma abordagem moderna chamada Compartilhamento de Recursos entre Origens (CORS). Primeiro, especifique uma URL na política com um conteúdo personalizado em HTML. O Azure AD B2C mescla os elementos de interface do usuário com o conteúdo HTML carregado da URL e exibe a página para o cliente.

Você pode enviar parâmetros para o Azure AD B2C em uma cadeia de caracteres de consulta. Passando o parâmetro para seu ponto de extremidade HTML, é possível alterar dinamicamente o conteúdo da página. Por exemplo, é possível alterar a imagem de tela de fundo na página de inscrição ou de entrada do Azure AD B2C, com base em um parâmetro passado do seu aplicativo Web ou móvel.

## <a name="how-do-i-get-started-with-azure-ad-b2c"></a>Como fazer para começar a utilizar o Azure AD B2C?

No Azure AD B2C, um locatário representa a sua organização e é um diretório de usuários. Cada locatário do Azure AD B2C é distinto e separado de outros locatários do Azure AD B2C. Um locatário contém informações sobre os clientes que se inscreveram para usar o aplicativo. Por exemplo, senhas, dados de perfil e permissões.

Você precisa vincular o seu locatário do Azure AD B2C à sua assinatura do Azure para habilitar todas as funcionalidades e pagar pelos encargos de uso. Para permitir que os clientes do Azure AD B2C entrem no aplicativo, você deve registrá-lo em um locatário do Azure AD B2C.

Antes de configurar seu aplicativo para usar o Azure AD B2C, primeiro crie um locatário Azure AD B2C e registre seu aplicativo. Para registrar seu aplicativo, conclua as etapas em [Tutorial: Registrar um aplicativo para habilitar a inscrição e entrar usando o Azure AD B2C](tutorial-register-applications.md).
  
Se você for um desenvolvedor de aplicativos web ASP.NET, configure seu aplicativo para autenticar contas usando as etapas no [Tutorial: habilitar um aplicativo web para autenticar com contas usando o Azure AD B2C](active-directory-b2c-tutorials-web-app.md).

Se você for um desenvolvedor de aplicativo da área de trabalho, configure seu aplicativo para autenticar contas usando as etapas no [Tutorial: habilitar um aplicativo da área de trabalho para autenticar com contas usando o Azure AD B2C](active-directory-b2c-tutorials-desktop-app.md).

Se você for um desenvolvedor de aplicativo de página única usando Node.js, configure seu aplicativo para autenticar contas usando as etapas no [Tutorial: habilitar um aplicativo de página única para autenticar com contas usando o Azure AD B2C](active-directory-b2c-tutorials-spa.md).

## <a name="next-steps"></a>Próximas etapas

Começar a configurar seu aplicativo para a experiência de inscrição e entrada ao continuar o tutorial.

> [!div class="nextstepaction"]
> [Tutorial: registrar um aplicativo para habilitar a inscrição e entrada usando o Azure AD B2C](tutorial-register-applications.md)
