---
title: 'Tutorial: Integração do Azure Active Directory ao Egnyte | Microsoft Docs'
description: Saiba como configurar o logon único entre o Azure Active Directory e o Egnyte.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 8c2101d4-1779-4b36-8464-5c1ff780da18
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/18/2017
ms.author: jeedes
ms.openlocfilehash: 4f6f6ef12f5a8dd8a9f210e9b1f1ca978ec5a1ac
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/02/2018
ms.locfileid: "39440449"
---
# <a name="tutorial-azure-active-directory-integration-with-egnyte"></a>Tutorial: integração do Active Directory do Azure ao Egnyte

Neste tutorial, você aprenderá a integrar o Egnyte ao Azure AD (Azure Active Directory).

A integração do Egnyte ao Azure AD oferece os seguintes benefícios:

- Você pode controlar no Azure AD quem terá acesso ao Egnyte
- Você pode permitir que os usuários façam logon automaticamente no Egnyte (Logon Único) com as respectivas contas do Azure AD
- Você pode gerenciar suas contas em um única localização: o Portal do Azure

Para conhecer mais detalhadamente a integração de aplicativos de SaaS ao Azure AD, consulte [o que é o acesso a aplicativos e logon único com o Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Pré-requisitos

Para configurar a integração do Azure AD ao Egnyte, você precisará dos seguintes itens:

- Uma assinatura do AD do Azure
- Uma assinatura habilitada para logon único do Egnyte

> [!NOTE]
> Para testar as etapas deste tutorial, nós não recomendamos o uso de um ambiente de produção.

Para testar as etapas deste tutorial, você deve seguir estas recomendações:

- Não use o ambiente de produção, a menos que seja necessário.
- Se não tiver um ambiente de avaliação do AD do Azure, você pode obter uma versão de avaliação de um mês [aqui](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Descrição do cenário
Neste tutorial, você testará o logon único do Azure AD em um ambiente de teste. O cenário descrito neste tutorial consiste em dois blocos de construção principais:

1. Adicionando Egnyte da galeria
1. configurar e testar o logon único do AD do Azure

## <a name="adding-egnyte-from-the-gallery"></a>Adicionando Egnyte da galeria
Para configurar a integração do Egnyte ao Azure AD, você precisará adicionar o Egnyte da galeria à sua lista de aplicativos SaaS gerenciados.

**Para adicionar o Egnyte da galeria, execute as seguintes etapas:**

1. No **[Portal do Azure](https://portal.azure.com)**, no painel navegação à esquerda, clique no ícone **Azure Active Directory**. 

    ![Active Directory][1]

1. Navegue até **aplicativos empresariais**. Em seguida, vá para **todos os aplicativos**.

    ![APLICATIVOS][2]
    
1. Clique no botão **Novo aplicativo** na parte superior da caixa de diálogo para adicionar o novo aplicativo.

    ![APLICATIVOS][3]

1. Na caixa de pesquisa, digite **Egnyte**.

    ![Criação de um usuário de teste do AD do Azure](./media/egnyte-tutorial/tutorial_egnyte_search.png)

1. No painel de resultados, selecione **Egnyte** e, em seguida, clique no botão **Adicionar** para adicionar o aplicativo.

    ![Criação de um usuário de teste do AD do Azure](./media/egnyte-tutorial/tutorial_egnyte_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>configurar e testar o logon único do AD do Azure
Nesta seção, você configurará e testará o logon único do Azure AD com o Egnyte com base em um usuário de teste chamado “Brenda Fernandes”.

Para que o logon único funcione, o Azure AD precisa saber qual usuário do Egnyte é equivalente a um usuário do Azure AD. Em outras palavras, é necessário estabelecer uma relação de vínculo entre um usuário do Azure AD e o usuário relacionado do Egnyte.

No Egnyte, atribua o valor do **nome de usuário** no Azure AD como o valor do **Nome de usuário** para estabelecer a relação de vínculo.

Para configurar e testar o logon único do Azure AD com o Egnyte, você precisa concluir os seguintes blocos de construção:

1. **[Configuring Azure AD Single Sign-On](#configuring-azure-ad-single-sign-on)** - para habilitar seus usuários a usar esse recurso.
1. **[Criação de um usuário de teste do AD do Azure](#creating-an-azure-ad-test-user)** – para testar o logon único do AD do Azure com Brenda Fernandes.
1. **[Criando um usuário de teste do Egnyte](#creating-an-egnyte-test-user)** – para ter um equivalente de Brenda Fernandes no Egnyte que esteja vinculado à representação de usuário do Azure AD.
1. **[Atribuição do usuário de teste do AD do Azure](#assigning-the-azure-ad-test-user)** – para permitir que Brenda Fernandes use o logon único do AD do Azure.
1. **[Teste do logon único](#testing-single-sign-on)** : para verificar se a configuração funciona.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuração do logon único do Azure AD

Nesta seção, você habilita o logon único do Azure AD no portal do Azure e configura o logon único no aplicativo Egnyte.

**Para configurar o logon único do Azure AD com o Egnyte, execute as seguintes etapas:**

1. No portal do Azure, na página de integração de aplicativos do **Egnyte**, clique em **Logon único**.

    ![Configurar o logon único][4]

1. Na caixa de diálogo **Logon único**, selecione **Modo** como **Logon baseado em SAML** para habilitar o logon único.
 
    ![Configurar o logon único](./media/egnyte-tutorial/tutorial_egnyte_samlbase.png)

1. Na seção **URLs e Domínio do Egnyte**, execute as seguintes etapas:

    ![Configurar o logon único](./media/egnyte-tutorial/tutorial_egnyte_url.png)

    Na caixa de texto **URL de Logon**, digite uma URL usando o seguinte padrão: `https://<companyname>.egnyte.com`

    > [!NOTE] 
    > Esse valor não é real. Atualize esse valor com a URL de Logon real. Entre em contato com a [equipe de suporte do Cliente do Egnyte](https://www.egnyte.com/corp/contact_egnyte.html) para obter esse valor. 
 
1. Na seção **Certificado de Autenticação do SAML**, clique em **Certificado (Base64)** e, em seguida, salve o arquivo do certificado no computador.

    ![Configurar o logon único](./media/egnyte-tutorial/tutorial_egnyte_certificate.png) 

1. Clique no botão **Salvar** .

    ![Configurar o logon único](./media/egnyte-tutorial/tutorial_general_400.png)

1. Na seção **Configuração do Egnyte**, clique em **Configurar Egnyte** para abrir a janela **Configurar logon**. Copie a **ID da Entidade SAML e a URL do Serviço de Logon Único SAML** da **seção Referência Rápida.**

    ![Configurar o logon único](./media/egnyte-tutorial/tutorial_egnyte_configure.png) 

1. Em uma janela diferente do navegador da Web, faça logon no site da empresa do Egnyte como administrador.

1. Clique em **Configurações**.
   
   ![Configurações](./media/egnyte-tutorial/ic787819.png "Configurações")

1. No menu, clique em **Configurações**.

   ![Configurações](./media/egnyte-tutorial/ic787820.png "Configurações")

1. Clique na guia **Configuração** e, em seguida, clique em **Segurança**.

    ![Segurança](./media/egnyte-tutorial/ic787821.png "Segurança")

1. Na seção **Autenticação de Logon Único** , realize as seguintes etapas:

    ![Autenticação de Logon Único](./media/egnyte-tutorial/ic787822.png "Autenticação de Logon Único")   
    
    a. Para **Autenticação de logon único**, selecione **SAML 2.0**.
   
    b. Para **Provedor de identidade**, selecione **AzureAD**.
   
    c. Cole a **URL de Logon Único do serviço SAML** copiada do portal do Azure para o **URL de logon do provedor de identidade** caixa de texto.
   
    d. Cole a **ID da Entidade SAML** copiada do portal do Azure na caixa de texto **ID da entidade do provedor de identidade**.
      
    e. Abra seu certificado codificado em base-64 no bloco de notas baixado do portal do Azure, copie o conteúdo dele para a área de transferência e cole-o na caixa de texto **Certificado do provedor de identidade**.
   
    f. Para **Mapeamento de usuário padrão**, selecione **Endereço de email**.
   
    g. Para **Usar valor do emissor específico do domínio**, selecione **desabilitado**.
   
    h. Clique em **Salvar**.

> [!TIP]
> É possível ler uma versão concisa dessas instruções no [Portal do Azure](https://portal.azure.com), enquanto você estiver configurando o aplicativo!  Depois de adicionar esse aplicativo da seção **Active Directory > Aplicativos Empresariais**, basta clicar na guia **Logon Único** e acessar a documentação inserida por meio da seção **Configuração** na parte inferior. Saiba mais sobre a funcionalidade de documentação inserida aqui: [Documentação inserida do Azure AD]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>Criação de um usuário de teste do AD do Azure
O objetivo desta seção é criar um usuário de teste no Portal do Azure chamado Brenda Fernandes.

![Criar um usuário do AD do Azure][100]

**Para criar um usuário de teste no AD do Azure, execute as seguintes etapas:**

1. No **Portal do Azure**, no painel de navegação esquerdo, clique no ícone **Azure Active Directory**.

    ![Criação de um usuário de teste do AD do Azure](./media/egnyte-tutorial/create_aaduser_01.png) 

1. Vá para **Usuários e grupos** e clique em **Todos os usuários** para exibir a lista de usuários.
    
    ![Criação de um usuário de teste do AD do Azure](./media/egnyte-tutorial/create_aaduser_02.png) 

1. Para abrir a caixa de diálogo **Usuário**, clique em **Adicionar** na parte superior da caixa de diálogo.
 
    ![Criação de um usuário de teste do AD do Azure](./media/egnyte-tutorial/create_aaduser_03.png) 

1. Na página do diálogo **Usuário**, execute as seguintes etapas:
 
    ![Criação de um usuário de teste do AD do Azure](./media/egnyte-tutorial/create_aaduser_04.png) 

    a. Na caixa de texto **Nome**, digite **Brenda Fernandes**.

    b. Na caixa de texto **Nome de usuário**, digite o **endereço de email** da conta de Brenda Fernandes.

    c. Selecione **Mostrar senha** e anote o valor de **senha**.

    d. Clique em **Criar**.
 
### <a name="creating-an-egnyte-test-user"></a>Como criar um usuário de teste do Egnyte

Para permitir que os usuários do Azure AD façam logon no Egnyte, eles deverão ser provisionados no Egnyte. No caso do Egnyte, o provisionamento é uma tarefa manual.

**Para provisionar contas de usuário, execute as seguintes etapas:**

1. Faça logon no site de empresa do **Egnyte** como administrador.

1. Vá para **Configurações \> Usuários e Grupos**.

1. Clique em **Adicionar Novo Usuário**e selecione o tipo de usuário que deseja adicionar.
   
   ![Usuários](./media/egnyte-tutorial/ic787824.png "Usuários")

1. Na seção **Novo Usuário Padrão** , realize as seguintes etapas:
   
   ![Novo Usuário Padrão](./media/egnyte-tutorial/ic787825.png "Novo Usuário Padrão")   

   a. Digite o **Email**, o **Nome de usuário** e outros detalhes de uma conta válida do Azure Active Directory que você deseja provisionar.
   
   b. Clique em **Salvar**.
    
    >[!NOTE]
    >O titular da conta do Active Directory do Azure receberá um email de notificação.
    >

>[!NOTE]
>É possível usar qualquer outra ferramenta de criação da conta de usuário do Egnyte ou as APIs fornecidas pelo Egnyte para provisionar as contas de usuário do AAD.
> 

### <a name="assigning-the-azure-ad-test-user"></a>Atribuição do usuário de teste do AD do Azure

Nesta seção, você permitirá que Brenda Fernandes use o logon único do Azure concedendo a ela acesso ao Egnyte.

![Atribuir usuário][200] 

**Para atribuir Brenda Fernandes ao Egnyte, execute as seguintes etapas:**

1. No Portal do Azure, abra a exibição de aplicativos e, em seguida, navegue até a exibição de diretório e vá para **Aplicativos Empresariais** e clique em **Todos os aplicativos**.

    ![Atribuir usuário][201] 

1. Na lista de aplicativos, selecione **Egnyte**.

    ![Configurar o logon único](./media/egnyte-tutorial/tutorial_egnyte_app.png) 

1. No menu à esquerda, clique em **usuários e grupos**.

    ![Atribuir usuário][202] 

1. Clique no botão **Adicionar**. Em seguida, selecione **usuários e grupos** na **Adicionar atribuição** caixa de diálogo.

    ![Atribuir usuário][203]

1. Em **usuários e grupos** caixa de diálogo, selecione **Britta Simon** na lista de usuários.

1. Clique em **selecione** botão **usuários e grupos** caixa de diálogo.

1. Clique em **atribuir** botão **Adicionar atribuição** caixa de diálogo.
    
### <a name="testing-single-sign-on"></a>Teste do logon único

Nesta seção, você testará sua configuração de logon único do Azure AD usando o Painel de Acesso.

Ao clicar no bloco do Egnyte no Painel de Acesso, você deverá ser conectado automaticamente a seu aplicativo do Egnyte.
Para saber mais sobre o Painel de Acesso, veja [Introdução ao Painel de Acesso](../user-help/active-directory-saas-access-panel-introduction.md). 

## <a name="additional-resources"></a>Recursos adicionais

* [Lista de tutoriais sobre como integrar aplicativos SaaS com o Active Directory do Azure](tutorial-list.md)
* [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/egnyte-tutorial/tutorial_general_01.png
[2]: ./media/egnyte-tutorial/tutorial_general_02.png
[3]: ./media/egnyte-tutorial/tutorial_general_03.png
[4]: ./media/egnyte-tutorial/tutorial_general_04.png

[100]: ./media/egnyte-tutorial/tutorial_general_100.png

[200]: ./media/egnyte-tutorial/tutorial_general_200.png
[201]: ./media/egnyte-tutorial/tutorial_general_201.png
[202]: ./media/egnyte-tutorial/tutorial_general_202.png
[203]: ./media/egnyte-tutorial/tutorial_general_203.png

