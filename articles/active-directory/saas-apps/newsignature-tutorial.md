---
title: 'Tutorial: Integração do Azure Active Directory com o Cloud Management Portal for Microsoft Azure | Microsoft Docs'
description: Saiba como configurar o logon único entre o Azure Active Directory e o Cloud Management Portal for Microsoft Azure.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 4ea9f47c-25ca-42b0-a878-9e7aa6f34973
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/16/2017
ms.author: jeedes
ms.openlocfilehash: 0cd14a308758701e207e0b1ee6d3591c4b0347bd
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/02/2018
ms.locfileid: "39427628"
---
# <a name="tutorial-azure-active-directory-integration-with-cloud-management-portal-for-microsoft-azure"></a>Tutorial: Integração do Azure Active Directory com o Cloud Management Portal for Microsoft Azure

O objetivo deste tutorial é mostrar como integrar o Cloud Management Portal do Microsoft Azure ao Azure AD (Azure Active Directory).

A Integração do Cloud Management Portal for Microsoft Azure com o Azure AD oferece os seguintes benefícios:

- No Azure AD, você pode controlar quem tem acesso ao Cloud Management Portal for Microsoft Azure
- Você pode permitir que seus usuários façam logon automaticamente no Cloud Management Portal for Microsoft Azure (Logon Único) com suas contas do Azure AD
- Você pode gerenciar suas contas em um única localização: o Portal do Azure

Para conhecer mais detalhadamente a integração de aplicativos de SaaS ao Azure AD, consulte [o que é o acesso a aplicativos e logon único com o Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Pré-requisitos

Para configurar a integração do Azure AD com o Cloud Management Portal for Microsoft Azure, você precisará dos seguintes itens:

- Uma assinatura do AD do Azure
- Uma assinatura habilitada para logon único do Cloud Management Portal for Microsoft Azure

> [!NOTE]
> Para testar as etapas deste tutorial, nós não recomendamos o uso de um ambiente de produção.

Para testar as etapas deste tutorial, você deve seguir estas recomendações:

- Não use o ambiente de produção, a menos que seja necessário.
- Se não tiver um ambiente de avaliação do AD do Azure, você pode obter uma versão de avaliação de um mês [aqui](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Descrição do cenário
Neste tutorial, você testará o logon único do Azure AD em um ambiente de teste. O cenário descrito neste tutorial consiste em dois blocos de construção principais:

1. Adição do Cloud Management Portal for Microsoft Azure a partir da galeria
1. configurar e testar o logon único do AD do Azure

## <a name="adding-cloud-management-portal-for-microsoft-azure-from-the-gallery"></a>Adição do Cloud Management Portal for Microsoft Azure a partir da galeria
Para configurar a integração do Cloud Management Portal for Microsoft Azure ao Azure AD, você precisará adicionar o Cloud Management Portal for Microsoft Azure da galeria para sua lista de aplicativos SaaS gerenciados.

**Para adicionar o Cloud Management Portal for Microsoft Azure a partir da galeria, execute as seguintes etapas:**

1. No **[Portal do Azure](https://portal.azure.com)**, no painel navegação à esquerda, clique no ícone **Azure Active Directory**. 

    ![Active Directory][1]

1. Navegue até **aplicativos empresariais**. Em seguida, vá para **todos os aplicativos**.

    ![APLICATIVOS][2]
    
1. Clique no botão **Novo aplicativo** na parte superior da caixa de diálogo para adicionar o novo aplicativo.

    ![APLICATIVOS][3]

1. Na caixa de pesquisa, digite **Cloud Management Portal for Microsoft Azure**.

    ![Criação de um usuário de teste do AD do Azure](./media/newsignature-tutorial/tutorial_newsignature_search.png)

1. No painel de resultados, selecione **Cloud Management Portal for Microsoft Azure** e clique em **Adicionar** para adicionar o aplicativo.

    ![Criação de um usuário de teste do AD do Azure](./media/newsignature-tutorial/tutorial_newsignature_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>configurar e testar o logon único do AD do Azure
O objetivo desta seção é mostrar como configurar e testar o logon único do Azure AD com o Cloud Management Portal for Microsoft Azure, com base em um usuário de teste chamado "Brenda Fernandes".

Para que o logon único funcione, o Azure AD precisa saber qual usuário do Cloud Management Portal for Microsoft Azure é equivalente a um usuário do Azure AD. Em outras palavras, é necessário estabelecer uma relação de vínculo entre um usuário do Azure AD e o usuário relacionado do Cloud Management Portal for Microsoft Azure.

No Cloud Management Portal for Microsoft Azure, atribua o valor de **nome de usuário** no Azure AD ao valor de **Nome de Usuário** para estabelecer a relação de vínculo.

Para configurar e testar o logon único do Azure AD com o Cloud Management Portal for Microsoft Azure, você precisará concluir os seguintes blocos de construção:

1. **[Configuring Azure AD Single Sign-On](#configuring-azure-ad-single-sign-on)** - para habilitar seus usuários a usar esse recurso.
1. **[Criação de um usuário de teste do AD do Azure](#creating-an-azure-ad-test-user)** – para testar o logon único do AD do Azure com Brenda Fernandes.
1. **[Criação de um usuário de teste do Cloud Management Portal for Microsoft Azure](#creating-a-cloud-management-portal-for-microsoft-azure-test-user)** – para ter um equivalente de Brenda Fernandes no Cloud Management Portal for Microsoft Azure vinculado à representação dela no Azure AD.
1. **[Atribuição do usuário de teste do AD do Azure](#assigning-the-azure-ad-test-user)** – para permitir que Brenda Fernandes use o logon único do AD do Azure.
1. **[Teste do logon único](#testing-single-sign-on)** : para verificar se a configuração funciona.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuração do logon único do Azure AD

Nesta seção, você habilitará o logon único do Azure AD no portal do Azure e configurará o logon único em seu aplicativo Cloud Management Portal for Microsoft Azure.

**Para configurar o logon único do Azure AD com o Cloud Management Portal for Microsoft Azure, execute as seguintes etapas:**

1. No portal do Azure, na página de integração de aplicativos do **Cloud Management Portal for Microsoft Azure**, clique em **Logon único**.

    ![Configurar o logon único][4]

1. Na caixa de diálogo **Logon único**, selecione **Modo** como **Logon baseado em SAML** para habilitar o logon único.
 
    ![Configurar o logon único](./media/newsignature-tutorial/tutorial_newsignature_samlbase.png)

1. Na seção **URLs e Domínio do Cloud Management Portal for Microsoft Azure**, execute as seguintes etapas:

    ![Configurar o logon único](./media/newsignature-tutorial/tutorial_newsignature_url.png)

    a. Na caixa de texto **URL de Logon**, digite uma URL usando os seguintes padrões: 
    
    | |
    |--|
    | `https://portal.newsignature.com/<instancename>` |   
    | `https://portal.igcm.com/<instancename>` |
    
    b. Na caixa de texto **Identificador**, digite uma URL usando os seguintes padrões: 
    
    | |
    |--|
    | `https://<subdomain>.igcm.com` |
    | `https://<subdomain>.newsignature.com` |

    c. Na caixa de texto **URL de Resposta** , digite uma URL nos seguintes padrões: 
    
    | |
    |--|
    | `https://<subdomain>.igcm.com/<instancename>` |
    | `https://<subdomain>.newsignature.com` |
    | `https://<subdomain>.newsignature.com/<instancename>` |

    > [!NOTE] 
    > Esses valores não são reais. Você precisa atualizar esses valores com a URL de Logon, o Identificador e a URL de Resposta reais. Entre em contato com [a equipe de suporte do cliente do Cloud Management Portal for Microsoft Azure](mailto:jczernuszka@newsignature.com) para obter esses valores. 
 
1. Na seção **Certificado de Autenticação do SAML**, clique em **Certificado (Base64)** e, em seguida, salve o arquivo do certificado no computador.

    ![Configurar o logon único](./media/newsignature-tutorial/tutorial_newsignature_certificate.png) 

1. Clique no botão **Salvar** .

    ![Configurar o logon único](./media/newsignature-tutorial/tutorial_general_400.png)

1. Na seção **Configuração do Cloud Management Portal for Microsoft Azure**, clique em **Configurar o Cloud Management Portal for Microsoft Azure** para abrir a janela **Configurar logon**. Copie a **URL de saída, a ID da Entidade SAML e a URL do Serviço de Logon Único SAML** da **seção de Referência Rápida.**

    ![Configurar o logon único](./media/newsignature-tutorial/tutorial_newsignature_configure.png) 

1. Para configurar o logon único no lado do **Cloud Management Portal for Microsoft Azure**, você precisa enviar o **Certificado** baixado, a **URL de Logout**, **a URL de Serviço de Logon Único do SAML** e a **ID da entidade do SAML** para a [equipe de suporte do Cloud Management Portal for Microsoft Azure](mailto:jczernuszka@newsignature.com). Eles definem essa configuração para ter a conexão de SSO de SAML definida corretamente em ambos os lados.

> [!TIP]
> É possível ler uma versão concisa dessas instruções no [Portal do Azure](https://portal.azure.com), enquanto você estiver configurando o aplicativo!  Depois de adicionar esse aplicativo da seção **Active Directory > Aplicativos Empresariais**, basta clicar na guia **Logon Único** e acessar a documentação inserida por meio da seção **Configuração** na parte inferior. Saiba mais sobre a funcionalidade de documentação inserida aqui: [Documentação inserida do Azure AD]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="creating-an-azure-ad-test-user"></a>Criação de um usuário de teste do AD do Azure
O objetivo desta seção é criar um usuário de teste no Portal do Azure chamado Brenda Fernandes.

![Criar um usuário do AD do Azure][100]

**Para criar um usuário de teste no AD do Azure, execute as seguintes etapas:**

1. No **Portal do Azure**, no painel de navegação esquerdo, clique no ícone **Azure Active Directory**.

    ![Criação de um usuário de teste do AD do Azure](./media/newsignature-tutorial/create_aaduser_01.png) 

1. Vá para **Usuários e grupos** e clique em **Todos os usuários** para exibir a lista de usuários.
    
    ![Criação de um usuário de teste do AD do Azure](./media/newsignature-tutorial/create_aaduser_02.png) 

1. Para abrir a caixa de diálogo **Usuário**, clique em **Adicionar** na parte superior da caixa de diálogo.
 
    ![Criação de um usuário de teste do AD do Azure](./media/newsignature-tutorial/create_aaduser_03.png) 

1. Na página do diálogo **Usuário**, execute as seguintes etapas:
 
    ![Criação de um usuário de teste do AD do Azure](./media/newsignature-tutorial/create_aaduser_04.png) 

    a. Na caixa de texto **Nome**, digite **Brenda Fernandes**.

    b. Na caixa de texto **Nome de usuário**, digite o **endereço de email** da conta de Brenda Fernandes.

    c. Selecione **Mostrar senha** e anote o valor de **senha**.

    d. Clique em **Criar**.
 
### <a name="creating-a-cloud-management-portal-for-microsoft-azure-test-user"></a>Criação de um usuário de teste do Cloud Management Portal for Microsoft Azure

O objetivo desta seção é criar um usuário chamado Brenda Fernandes no Cloud Management Portal for Microsoft Azure. Trabalhe com a [equipe de suporte do Cloud Management Portal for Microsoft Azure](mailto:jczernuszka@newsignature.com) para adicionar os usuários à conta do Cloud Management Portal for Microsoft Azure.


### <a name="assigning-the-azure-ad-test-user"></a>Atribuição do usuário de teste do AD do Azure

O objetivo desta seção é permitir que Brenda Fernandes use o logon único do Azure, concedendo a ela acesso ao Cloud Management Portal for Microsoft Azure.

![Atribuir usuário][200] 

**Para atribuir Brenda Fernandes ao Cloud Management Portal for Microsoft Azure, execute as seguintes etapas:**

1. No Portal do Azure, abra a exibição de aplicativos e, em seguida, navegue até a exibição de diretório e vá para **Aplicativos Empresariais** e clique em **Todos os aplicativos**.

    ![Atribuir usuário][201] 

1. Na lista de aplicativos, selecione **Cloud Management Portal for Microsoft Azure**.

    ![Configurar o logon único](./media/newsignature-tutorial/tutorial_newsignature_app.png) 

1. No menu à esquerda, clique em **usuários e grupos**.

    ![Atribuir usuário][202] 

1. Clique no botão **Adicionar**. Em seguida, selecione **usuários e grupos** na **Adicionar atribuição** caixa de diálogo.

    ![Atribuir usuário][203]

1. Em **usuários e grupos** caixa de diálogo, selecione **Britta Simon** na lista de usuários.

1. Clique em **selecione** botão **usuários e grupos** caixa de diálogo.

1. Clique em **atribuir** botão **Adicionar atribuição** caixa de diálogo.
    
### <a name="testing-single-sign-on"></a>Teste do logon único

O objetivo desta seção é testar sua configuração de logon único do Azure AD usando o Painel de Acesso.
Quando você clicar no bloco Cloud Management Portal for Microsoft Azure no Painel de Acesso, deverá fazer logon automaticamente no aplicativo do Cloud Management Portal for Microsoft Azure.

Para saber mais sobre o Painel de Acesso, veja [Introdução ao Painel de Acesso](../user-help/active-directory-saas-access-panel-introduction.md).

## <a name="additional-resources"></a>Recursos adicionais

* [Lista de tutoriais sobre como integrar aplicativos SaaS com o Active Directory do Azure](tutorial-list.md)
* [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/newsignature-tutorial/tutorial_general_01.png
[2]: ./media/newsignature-tutorial/tutorial_general_02.png
[3]: ./media/newsignature-tutorial/tutorial_general_03.png
[4]: ./media/newsignature-tutorial/tutorial_general_04.png

[100]: ./media/newsignature-tutorial/tutorial_general_100.png

[200]: ./media/newsignature-tutorial/tutorial_general_200.png
[201]: ./media/newsignature-tutorial/tutorial_general_201.png
[202]: ./media/newsignature-tutorial/tutorial_general_202.png
[203]: ./media/newsignature-tutorial/tutorial_general_203.png

