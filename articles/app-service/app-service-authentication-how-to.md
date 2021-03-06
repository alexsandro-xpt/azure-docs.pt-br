---
title: Personalizar a autenticação e a autorização no Serviço de Aplicativo do Azure | Microsoft Docs
description: Mostra como personalizar a autenticação e a autorização no Serviço de Aplicativo e obter declarações de usuário e tokens diferentes.
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 03/14/2018
ms.author: cephalin
ms.openlocfilehash: 629a76ab5610625e14780d7b5c57d3979c2224c9
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344163"
---
# <a name="customize-authentication-and-authorization-in-azure-app-service"></a>Personalizar a autenticação e a autorização no Serviço de Aplicativo do Azure

Este artigo mostra como personalizar [a autenticação e a autorização no Serviço de Aplicativo](app-service-authentication-overview.md) e gerenciar a identidade do seu aplicativo. 

Para começar rapidamente, veja um dos seguintes tutoriais:

* [Tutorial: Autenticar e autorizar usuários de ponta a ponta no Serviço de Aplicativo do Azure (Windows)](app-service-web-tutorial-auth-aad.md)
* [Tutorial: Autenticar e autorizar usuários de ponta a ponta no Serviço de Aplicativo do Azure para Linux](containers/tutorial-auth-aad.md)
* [Como configurar seu aplicativo para usar o logon do Active Directory do Azure](app-service-mobile-how-to-configure-active-directory-authentication.md)
* [Como configurar seu aplicativo para usar o logon do Facebook](app-service-mobile-how-to-configure-facebook-authentication.md)
* [Como configurar seu aplicativo para usar o logon do Google](app-service-mobile-how-to-configure-google-authentication.md)
* [Como configurar seu aplicativo para usar o logon da Conta da Microsoft](app-service-mobile-how-to-configure-microsoft-authentication.md)
* [Como configurar seu aplicativo para usar o logon do Twitter](app-service-mobile-how-to-configure-twitter-authentication.md)

## <a name="use-multiple-sign-in-providers"></a>Usar vários provedores de entrada

A configuração do portal não oferece uma maneira prática turnkey para apresentar vários provedores de entrada aos usuários (como o Facebook e o Twitter). No entanto, não é difícil adicionar a funcionalidade ao seu aplicativo Web. As etapas são destacadas como a seguir:

Primeiro, na página **Autenticação/Autorização** no Portal do Azure, configure cada provedor de identidade que você deseja habilitar.

Em **Ação a tomar quando a solicitação não está autenticada**, selecione **Permitir solicitações anônimas (nenhuma ação)**.

Na página de entrada, ou na barra de navegação ou em qualquer outro local de seu aplicativo Web, adicione um link de entrada para cada um dos provedores habilitados por você (`/.auth/login/<provider>`). Por exemplo: 

```HTML
<a href="/.auth/login/aad">Log in with Azure AD</a>
<a href="/.auth/login/microsoftaccount">Log in with Microsoft Account</a>
<a href="/.auth/login/facebook">Log in with Facebook</a>
<a href="/.auth/login/google">Log in with Google</a>
<a href="/.auth/login/twitter">Log in with Twitter</a>
```

Quando o usuário clica em um dos links, a respectiva página de entrada é aberta para que ele entre.

Para redirecionar o usuário pós-entada para uma URL personalizada, use o parâmetro de cadeia de caracteres de consulta `post_login_redirect_url` (não deve ser confundido com o URI de redirecionamento na configuração do provedor de identidade). Por exemplo, para orientar o usuário para `/Home/Index` após entrar, use o seguinte código HTML:

```HTML
<a href="/.auth/login/<provider>?post_login_redirect_url=/Home/Index">Log in</a>
```

## <a name="sign-out-of-a-session"></a>Sair de uma sessão

Os usuários podem iniciar uma saída, enviando uma `GET` solicitação ao ponto de extremidade `/.auth/logout` do aplicativo. A solicitação `GET` faz o seguinte:

- Limpa os cookies de autenticação da sessão atual.
- Exclui os tokens do usuário atual do Token Store.
- Para o Azure Active Directory e o Google, executa uma saída do servidor no provedor de identidade.

Aqui está um link de saída simples em uma página da Web:

```HTML
<a href="/.auth/logout">Sign out</a>
```

Por padrão, uma saída com êxito redireciona o cliente para a URL `/.auth/logout/done`. É possível alterar a página de redirecionamento pós-saída, adicionando o parâmetro de consulta `post_logout_redirect_uri`. Por exemplo: 

```
GET /.auth/logout?post_logout_redirect_uri=/index.html
```

É recomendável [codificar](https://wikipedia.org/wiki/Percent-encoding) o valor de `post_logout_redirect_uri`.

Ao usar URLs totalmente qualificadas, a URL deve ser hospedada no mesmo domínio ou configurada como uma URL de redirecionamento externo permitido para o aplicativo. No exemplo a seguir, para redirecionar para `https://myexternalurl.com` que não está hospedado no mesmo domínio:

```
GET /.auth/logout?post_logout_redirect_uri=https%3A%2F%2Fmyexternalurl.com
```

É necessário executar o comando a seguir no [Azure Cloud Shell](../cloud-shell/quickstart.md):

```azurecli-interactive
az webapp auth update --name <app_name> --resource-group <group_name> --allowed-external-redirect-urls "https://myexternalurl.com"
```

## <a name="preserve-url-fragments"></a>Preservar fragmentos de URL

Depois que os usuários entram no aplicativo, geralmente querem ser redirecionados para a mesma seção da mesma página, como `/wiki/Main_Page#SectionZ`. No entanto, como os [fragmentos de URL](https://wikipedia.org/wiki/Fragment_identifier) (por exemplo, `#SectionZ`) nunca são enviados ao servidor, consequentemente não são preservados por padrão depois que a assinatura OAuth é concluída e redirecionada de volta ao aplicativo. Portanto, os usuários obtêm uma experiência abaixo do ideal quando precisam navegar novamente para a âncora desejada. Essa limitação aplica-se a todas as soluções de autenticação do lado do servidor.

Na autenticação do Serviço de Aplicativo, é possível preservar os fragmentos de URL na assinatura OAuth. Para fazer isso, defina uma configuração de aplicativo chamada `WEBSITE_AUTH_PRESERVE_URL_FRAGMENT` para `true`. Você pode fazer isso no [portal do Azure](https://portal.azure.com) ou simplesmente executar o comando a seguir no [Azure Cloud Shell](../cloud-shell/quickstart.md):

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group <group_name> --settings WEBSITE_AUTH_PRESERVE_URL_FRAGMENT="true"
```

## <a name="access-user-claims"></a>Acessar declarações de usuários

O Serviço de Aplicativo transmite declarações do usuário para seu aplicativo usando cabeçalhos especiais. Solicitações externas não são permitidas para definir esses cabeçalhos, portanto são apresentadas somente se definido pelo Serviço de Aplicativo. Alguns cabeçalhos de exemplo incluem:

* X-MS-CLIENT-PRINCIPAL-NAME
* X-MS-CLIENT-PRINCIPAL-ID

Um código escrito em qualquer linguagem ou estrutura pode obter as informações necessárias desses cabeçalhos. Para aplicativos ASP.NET 4.6, **ClaimsPrincipal** é definido automaticamente com os valores apropriados.

Seu aplicativo também pode obter detalhes adicionais sobre o usuário autenticado chamando `/.auth/me`. Os SDKs do servidor dos Aplicativos Móveis fornecem métodos auxiliares para trabalhar com esses dados. Para saber mais, confira [Como usar o SDK do Node.js dos Aplicativos Móveis do Azure](../app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#howto-tables-getidentity) e [Trabalhar com o SDK do servidor de back-end .NET para Aplicativos Móveis do Azure](../app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#user-info).

## <a name="retrieve-tokens-in-app-code"></a>Recuperar tokens no código do aplicativo

No seu código de servidor, os tokens específicos do provedor são injetados no cabeçalho da solicitação, para que você possa acessá-los facilmente. A tabela a seguir mostra os possíveis nomes de cabeçalho do token:

| | |
|-|-|
| Azure Active Directory | `X-MS-TOKEN-AAD-ID-TOKEN` <br/> `X-MS-TOKEN-AAD-ACCESS-TOKEN` <br/> `X-MS-TOKEN-AAD-EXPIRES-ON`  <br/> `X-MS-TOKEN-AAD-REFRESH-TOKEN` |
| Token do Facebook | `X-MS-TOKEN-FACEBOOK-ACCESS-TOKEN` <br/> `X-MS-TOKEN-FACEBOOK-EXPIRES-ON` |
| Google | `X-MS-TOKEN-GOOGLE-ID-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-ACCESS-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-EXPIRES-ON` <br/> `X-MS-TOKEN-GOOGLE-REFRESH-TOKEN` |
| Conta da Microsoft | `X-MS-TOKEN-MICROSOFTACCOUNT-ACCESS-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-EXPIRES-ON` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-AUTHENTICATION-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-REFRESH-TOKEN` |
| Twitter | `X-MS-TOKEN-TWITTER-ACCESS-TOKEN` <br/> `X-MS-TOKEN-TWITTER-ACCESS-TOKEN-SECRET` |
|||

Do seu código do cliente (por exemplo, um aplicativo móvel ou JavaScript no navegador), envie uma solicitação HTTP `GET` para `/.auth/me`. O JSON retornado tem os tokens específicos do provedor.

> [!NOTE]
> Tokens de acesso são para acessar recursos do provedor, para que eles fiquem presentes somente se você configurar o provedor com um segredo do cliente. Para saber como obter tokens de atualização, consulte [Tokens de acesso de atualização](#refresh-access-tokens).

## <a name="refresh-access-tokens"></a>Tokens de acesso de atualização

Quando o token de acesso do provedor expira, você precisa autenticar novamente o usuário. Você pode evitar a expiração do token fazendo uma `GET` chamada para o `/.auth/refresh` ponto de extremidade de seu aplicativo. Quando chamado, o Serviço de Aplicativo atualiza automaticamente tokens de acesso no repositório de token para o usuário autenticado. As solicitações subsequentes de tokens do seu código do aplicativo obtêm tokens atualizados. No entanto, para que a atualização do token funcione, o repositório de token deve conter [tokens de atualização](https://auth0.com/learn/refresh-tokens/) para o seu provedor. A forma de obter tokens de atualização é documentada por cada provedor, mas a lista a seguir traz um breve resumo:

- **Google**: anexe um `access_type=offline` parâmetro de cadeia de caracteres para consulta a sua `/.auth/login/google` chamada à API. Se usar o SDK de Aplicativos Móveis, você pode adicionar o parâmetro a uma das `LogicAsync` sobrecargas (consulte [Tokens de atualização do Google](https://developers.google.com/identity/protocols/OpenIDConnect#refresh-tokens)).
- **Facebook**: não fornece tokens de atualização. Tokens de vida útil longa expiram em 60 dias (consulte [Expiração e extensão de tokens de acesso do Facebook](https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension)).
- **Twitter**: tokens de acesso não expiram (consulte [Perguntas frequentes sobre o OAuth do Twitter](https://developer.twitter.com/en/docs/basics/authentication/guides/oauth-faq)).
- **Microsoft Account**: quando [definir configurações de autenticação de conta Microsoft](app-service-mobile-how-to-configure-microsoft-authentication.md), selecione o escopo `wl.offline_access`.
- **Azure Active Directory**: em [https://resources.azure.com](https://resources.azure.com), execute as seguintes etapas:
    1. Na parte superior da página, selecione **Ler/Gravar**.
    1. No navegador à esquerda, navegue até **assinaturas** > **_\<assinatura\_nome_** > **resourceGroups** > _**\<recurso\_grupo\_nome >**_ > **provedores** > **Microsoft.web** > **sites** > _**\<aplicativo \_nome >**_ > **config** > **authsettings**. 
    1. Clique em **Editar**.
    1. Modifique a propriedade a seguir. Substitua _\<aplicativo\_id>_ pela ID do aplicativo do Azure Active Directory do serviço que você deseja acessar.

        ```json
        "additionalLoginParams": ["response_type=code id_token", "resource=<app_id>"]
        ```

    1. Clique em **Put**. 

Depois que seu provedor estiver configurado, você poderá [ encontrar o token de atualização e o tempo de expiração do token de acesso ](#retrieve-tokens-in-app-code) na loja do token. 

Para atualizar o token de acesso a qualquer momento, basta chamar `/.auth/refresh` em qualquer idioma. O trecho a seguir usa o jQuery para atualizar seus tokens de acesso de um cliente JavaScript.

```JavaScript
function refreshTokens() {
  var refreshUrl = "/.auth/refresh";
  $.ajax(refreshUrl) .done(function() {
    console.log("Token refresh completed successfully.");
  }) .fail(function() {
    console.log("Token refresh failed. See application logs for details.");
  });
}
```

Se um usuário revoga as permissões concedidas ao seu aplicativo, a chamada para `/.auth/me` pode falhar com um resposta `403 Forbidden`. Para diagnosticar erros, verifique os logs de aplicativo para obter detalhes.

## <a name="extend-session-expiration-grace-period"></a>Estender o período de cortesia da sessão

Depois que uma sessão autenticada expira, há um período de cortesia de 72 horas por padrão. Dentro desse período, você pode atualizar o cookie de sessão ou o token de sessão com o Serviço de Aplicativo sem reautenticar o usuário. Você pode simplesmente chamar `/.auth/refresh` quando o cookie de sessão ou token de sessão se torna inválido e não é necessário rastrear a expiração do token por conta própria. Após o período de cortesia de 72 horas, o usuário deverá entrar novamente para obter um token de sessão ou o cookie de sessão válido.

Se 72 horas não for tempo suficiente, você pode estender essa janela de expiração. Estender a expiração por um longo período pode ter implicações significativas de segurança (por exemplo, quando um token de autenticação é perdido ou roubado). Portanto, você deve deixar o padrão de 72 horas ou definir o período de extensão para o menor valor.

Para estender a janela de validade padrão, execute o seguinte comando no [Cloud Shell](../cloud-shell/overview.md).

```azurecli-interactive
az webapp auth update --resource-group <group_name> --name <app_name> --token-refresh-extension-hours <hours>
```

> [!NOTE]
> O período de cortesia se aplica somente à sessão autenticada do Serviço de Aplicativo, não aos tokens de provedores de identidade. Não há um período de cortesia para os tokens de provedor expirados. 
>

## <a name="limit-the-domain-of-sign-in-accounts"></a>Limite do domínio de contas de entrada

A Conta da Microsoft e o Microsoft Azure Active Directory permitem a entrada de vários domínios. Por exemplo, a Conta da Microsoft permite contas de _outlook.com_, _live.com_ e _hotmail.com_. O Microsoft Azure Active Directory permite qualquer número de domínios personalizados para as contas de entrada. Esse comportamento pode ser indesejável para um aplicativo interno, que você não deseja que qualquer pessoa com uma conta de _outlook.com_ acesse. Para limitar o nome de domínio das contas de entrada, siga estas etapas.

Em [https://resources.azure.com](https://resources.azure.com), navegue até **assinaturas** > **_\<assinatura\_nome_** > **resourceGroups** > _**\<recurso\_grupo\_nome>**_ > **provedores** > **Microsoft.Web** > **sites** > _**\<aplicativo\_nome>**_ > **config** > **authsettings**. 

Clique em **Editar**, modifique a propriedade a seguir e, em seguida, clique em **Put**. Certifique-se de substituir _\<domínio\_nome >_ pelo domínio que você deseja.

```json
"additionalLoginParams": ["domain_hint=<domain_name>"]
```
## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Tutorial: Autenticar e autorizar usuários ponta a ponta (Windows)](app-service-web-tutorial-auth-aad.md)
> [Tutorial: Autenticar e autorizar usuários de ponta a ponta (Linux)](containers/tutorial-auth-aad.md)