---
title: Adicionar o Google como provedor de identidade para o Azure Active Directory B2B | Microsoft Docs
description: Associe ao Google para permitir que usuários convidados entrem em seus aplicativos Azure AD usando as próprias contas do Gmail
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 08/20/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: mal
ms.openlocfilehash: 4ffe6cf3f1da4c149d1cb39856d02fc40acd20cf
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/18/2018
ms.locfileid: "45984785"
---
# <a name="add-google-as-an-identity-provider-for-b2b-guest-users"></a>Adicionar o Google como provedor de identidade para usuários convidados B2B

Ao configurar a federação com o Google, você pode permitir que os usuários convidados entrem em seus aplicativos e recursos compartilhados usando as próprias contas do Google, sem precisar criar Contas da Microsoft (contas de serviço gerenciado) ou contas do Azure AD.  
> [!NOTE]
> Seus usuários convidados do Google precisam entrar usando um link que inclui o contexto do locatário; por exemplo, `https://myapps.microsoft.com/?tenantid=<tenant id>`. Links diretos para aplicativos e recursos também funcionam desde que incluam o contexto do locatário. No momento, os usuários convidados não conseguem entrar usando pontos de extremidade sem contexto do locatário. Por exemplo, o uso de `https://myapps.microsoft.com`, `https://portal.azure.com` ou do ponto de extremidade comum das Equipes resultará em erro.
 
## <a name="what-is-the-experience-for-the-google-user"></a>Qual é a experiência do usuário do Google?
Quando você envia um convite para um usuário do Google Gmail, o usuário convidado deve acessar seus aplicativos ou recursos compartilhados utilizando um link que inclui o contexto do locatário. A experiência varia dependendo de a conexão ao Google já ter ocorrido ou não:
  - Se o usuário convidado não entrou no Google, a conexão ao Google será solicitada.
  - Se o usuário convidado já entrou no Google, ele poderá escolher a conta que deseja usar. Ele precisa escolher a conta que foi usada para convidá-lo.

Se o usuário convidado vir um erro de "cabeçalho muito longo", poderá tentar limpar os cookies ou abrir uma janela particular ou incógnita e tentar entrar novamente.

![Entrar com o Google](media/google-federation/google-sign-in.png)

## <a name="step-1-configure-a-google-developer-project"></a>Etapa 1: Configurar um projeto de desenvolvedor do Google
Primeiramente, crie um novo projeto no Console de Desenvolvedores do Google para obter um ID do cliente e um segredo do cliente que possa adicionar posteriormente ao Azure AD. 
1. Acesse as APIs do Google em https://console.developers.google.com e entre com sua conta do Google. É recomendável que você use uma conta do Google de equipe compartilhada.
2. Crie um novo projeto: no painel, selecione **Criar projeto** e, em seguida, selecione **Criar**. Na página Novo projeto, insira um **Nome do projeto** e, em seguida, selecione **Criar**.
   
   ![Novo projeto do Google](media/google-federation/google-new-project.png)

3. Verifique se seu novo projeto está selecionado no menu do projeto. Em seguida, abra o menu no canto superior esquerdo e selecione **APIs e serviços** > **Credenciais**.

   ![Credenciais de API do Google](media/google-federation/google-api.png)
 
4. Escolha a guia **Tela de consentimento do OAuth** e insira um **Nome do produto mostrado aos usuários**. (Deixe as outras configurações.) Clique em **Salvar**.

   ![Tela de consentimento do OAuth do Google](media/google-federation/google-oauth-consent-screen.png)

5. Escolha a guia **Credenciais**. No menu **Criar credenciais**, escolha **ID do cliente OAuth**.

   ![Credenciais de API do Google](media/google-federation/google-api-credentials.png)

6. Em **Tipo de aplicativo**, escolha **Aplicativo Web** e, em **URIs de redirecionamento autorizados**, insira estes URIs:
   - `https://login.microsoftonline.com` 
   - `https://login.microsoftonline.com/te/<directory id>/oauth2/authresp` <br>(em que `<directory id>` é a ID do seu diretório)
   
    > [!NOTE]
    > Para localizar a ID do seu diretório, acesse https://portal.azure.com e, em **Azure Active Directory**, escolha **Propriedades** e copie o **ID do diretório**.

   ![Criar ID do cliente OAuth](media/google-federation/google-create-oauth-client-id.png)

7. Selecione **Criar**. Copie a ID do cliente e o segredo do cliente, que você usará quando adicionar o provedor de identidade no portal do Azure AD.

   ![ID do cliente e segredo do cliente OAuth](media/google-federation/google-auth-client-id-secret.png)

## <a name="step-2-configure-google-federation-in-azure-ad"></a>Etapa 2: Configurar a federação do Google no Azure AD 
Agora, você definirá a ID do cliente e o segredo do cliente do Google, seja inserindo-o no portal do Azure AD ou usando o PowerShell. Lembre-se de testar sua configuração da federação do Google. Convide-se usando um endereço do Gmail e tente resgatar o convite com sua conta do Google convidada. 

#### <a name="to-configure-google-federation-in-the-azure-ad-portal"></a>Como configurar a federação do Google no portal do Azure AD 
1. Vá para o [Portal do Azure](https://portal.azure.com). No painel esquerdo, selecione **Azure Active Directory**. 
2. Selecione **Relações organizacionais**.
3. Selecione **Provedores de identidade** e, em seguida, clique no botão **Google**.
4. Insira um nome. Em seguida, insira a ID do cliente e o segredo do cliente obtidos anteriormente. Clique em **Salvar**. 

   ![Adicionar provedor de identidade do Google](media/google-federation/google-identity-provider.png)

#### <a name="to-configure-google-federation-by-using-powershell"></a>Como configurar a federação do Google usando o PowerShell
1. Instale a versão mais recente do módulo PowerShell for Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)) do Azure AD.
2. Execute o seguinte comando: `Connect-AzureAD`.
3. No prompt de entrada, entre com a conta de Administrador Global gerenciada.  
4. Execute o comando a seguir: 
   
   `New-AzureADMSIdentityProvider -Type Google -Name Google -ClientId [Client ID] -ClientSecret [Client secret]`
 
   > [!NOTE]
   > Use a ID do cliente e o segredo do cliente do aplicativo criado na "Etapa 1: Configurar um projeto de desenvolvedor do Google". Para obter mais informações, consulte o artigo [New-AzureADMSIdentityProvider](https://docs.microsoft.com/en-us/powershell/module/azuread/new-azureadmsidentityprovider?view=azureadps-2.0-preview). 
 
## <a name="how-do-i-remove-google-federation"></a>Como fazer a remoção da federação do Google?
É possível excluir sua configuração da federação do Google. Se fizer isso, os usuários convidados do Google que já resgataram seu convite não conseguirão entrar, mas você poderá conceder novamente a eles o acesso a seus recursos se os excluir do diretório e convidá-los outra vez. 
 
### <a name="to-delete-google-federation-in-the-azure-ad-portal"></a>Para excluir a federação do Google no portal do Azure AD: 
1. Vá para o [Portal do Azure](https://portal.azure.com). No painel esquerdo, selecione **Azure Active Directory**. 
2. Selecione **Relações organizacionais**.
3. Selecione **Provedores de identidade** e, em seguida, clique no botão **Google**.
4. Selecione **Google** e, em seguida, selecione **Excluir**. 
   
   ![Provedor de identidade social excluído](media/google-federation/google-social-identity-providers.png)

1. Clique em **Sim** para confirmar a exclusão. 

### <a name="to-delete-google-federation-by-using-powershell"></a>Para excluir a federação do Google usando o PowerShell: 
1. Instale a versão mais recente do módulo PowerShell for Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)) do Azure AD.
2. Execute `Connect-AzureAD`.  
4. No prompt de logon, entre com a conta de Administrador Global gerenciada.  
5. Digite o seguinte comando:

    `Remove-AzureADMSIdentityProvider -Id Google-OAUTH`

   > [!NOTE]
   > Para obter mais informações, consulte [AzureADMSIdentityProvider remover](https://docs.microsoft.com/en-us/powershell/module/azuread/Remove-AzureADMSIdentityProvider?view=azureadps-2.0-preview). 