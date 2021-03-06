---
title: Modelo de dados de nível de compatibilidade no Azure Analysis Services | Microsoft Docs
description: Noções básicas sobre o nível de compatibilidade do modelo de dados de tabela.
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 17a75e86d5539427c8837b3077aacf85b4681ad6
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/03/2018
ms.locfileid: "37447516"
---
# <a name="compatibility-level-for-analysis-services-tabular-models"></a>Nível de compatibilidade para modelos de tabela do Analysis Services

*Nível de compatibilidade* refere-se a comportamentos específicos à versão no mecanismo do Analysis Services. As alterações para o nível de compatibilidade geralmente coincidem com as versões principais do SQL Server. Essas alterações também são implementadas no Azure Analysis Services para manter a paridade entre as duas plataformas. Alterações de nível de compatibilidade também afetam recursos disponíveis em modelos de tabela. Por exemplo, o DirectQuery e os metadados de objeto de tabela têm implementações diferentes dependendo do nível de compatibilidade. O nível de compatibilidade é especificado no projeto do modelo tabular no Visual Studio (SSDT). Modelos tabulares criados no Power BI Desktop e importados dele estão apenas no nível de compatibilidade 1400.

O Azure Analysis Services dá suporte a modelos de tabela no nível de compatibilidade 1200 e 1400. 

O nível de compatibilidade mais recente é 1400. Esse nível coincide com o Analysis Services do SQL Server 2017. Os principais recursos do nível de compatibilidade 1400 incluem:

*  Novos recursos de conectividade de dados e importação com suporte para APIs TOM e scripts de TMSL. 
*  Transformação de dados e recursos de mashup de dados usando expressões Obter dados e M.
*  Medidas que suportam uma propriedade de linhas de detalhes com uma expressão DAX. Essa propriedade permite que as ferramentas de cliente, como o Microsoft Excel, façam buscas detalhadas de dados detalhados do relatório agregado. Por exemplo, quando os usuários visualizam o total de vendas de uma região e mês, eles podem visualizar os detalhes do pedido associado. 
*  Segurança em nível de objeto para nomes de tabela e coluna, além dos dados dentro deles.
*  Suporte aprimorado para hierarquias desbalanceadas.
*  Monitoramento de desempenho e melhorias.
  
## <a name="set-compatibility-level"></a>Configuração de nível de compatibilidade 
 Ao criar um novo projeto de modelo de tabela no SSDT, você pode especificar o nível de compatibilidade na caixa de diálogo **Designer de modelo de tabela**. 
  
 ![ssas_tabularproject_compat1200](./media/analysis-services-compat-level/aas-tabularproject-compat.png)  
  
 Ao selecionar a opção **Não mostrar esta mensagem novamente**, todos os projetos subsequentes usam o nível de compatibilidade que você especificou como o padrão. Você pode alterar o nível de compatibilidade padrão no SSDT em **Ferramentas** > **Opções**.  
  
 Para atualizar um projeto de modelo de tabela existente no SSDT, defina a propriedade do **Nível de compatibilidade** na janela**Propriedades** do modelo. Não se esqueça de que o nível de compatibilidade de atualização é irreversível.
  
## <a name="check-compatibility-level-for-a-tabular-model-database-in-sql-server-management-studio"></a>Verifique o nível de compatibilidade de um banco de dados de modelo de tabela no SQL Server Management Studio 
 No SSMS, clique o botão direito do mouse no nome do banco de dados > **Propriedades** > **Nível de Compatibilidade**.  
  
## <a name="check-supported-compatibility-level-for-a-server-in-ssms"></a>Verifique o nível de compatibilidade com suporte para um servidor no SSMS  
 No SSMS, clique o botão direito do mouse no nome do servidor > **Propriedades** > **Nível de Compatibilidade com Suporte**.  
  
 Essa propriedade especifica o nível mais alto de compatibilidade de um banco de dados que será executado no servidor (exceto a visualização). O nível de compatibilidade com suporte não pode ser alterado.  

## <a name="next-steps"></a>Próximas etapas
  [Como criar um modelo no portal do Azure](analysis-services-create-model-portal.md)   
  [Gerenciar o Analysis Services](analysis-services-manage.md)  
