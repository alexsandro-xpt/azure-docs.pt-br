---
title: Criar seu primeiro banco de dados SQL do Azure - C# | Microsoft Docs
description: Aprenda a criar seu primeiro banco de dados SQL do Azure e conectá-lo a um programa em C# usando o ADO.NET.
services: sql-database
author: MightyPen
manager: craigg-msft
ms.reviewer: CarlRabeler
ms.service: sql-database
ms.custom: develop databases, mvc, devcenter
ms.topic: tutorial
ms.author: genemi
ms.date: 06/07/2018
ms.openlocfilehash: 9fdc23eb2e607d1264e1bd2bc9dc85ef8d9be642
ms.sourcegitcommit: 8b694bf803806b2f237494cd3b69f13751de9926
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/20/2018
ms.locfileid: "46498142"
---
# <a name="design-an-azure-sql-database-and-connect-with-cx23-and-adonet"></a>Criar um banco de dados SQL do Azure e conectar-se com C&#x23 e o ADO.NET

O Banco de Dados SQL do Azure é um DBaaS (banco de dados como serviço) no Microsoft Cloud (Azure). Neste tutorial, você aprenderá a usar o portal do Azure e o ADO.NET com o Visual Studio para: 

> [!div class="checklist"]
> * Criar um banco de dados no Portal do Azure
> * Configurar uma regra de firewall de nível de servidor no Portal do Azure
> * Conecte-se ao banco de dados com o ADO.NET e o Visual Studio
> * Criar tabelas com o ADO.NET
> * Inserir, atualizar e excluir dados com o ADO.NET 
> * Consultar ADO.NET de dados

Se você não tiver uma assinatura do Azure, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.

## <a name="prerequisites"></a>Pré-requisitos

Uma instalação do [Visual Studio Community 2017, Visual Studio Professional 2017 ou Visual Studio Enterprise 2017](https://www.visualstudio.com/downloads/).

<!-- The following included .md, sql-database-tutorial-portal-create-firewall-connection-1.md, is long.
And it starts with a ## H2.
-->

[!INCLUDE [sql-database-tutorial-portal-create-firewall-connection-1](../../includes/sql-database-tutorial-portal-create-firewall-connection-1.md)]


<!-- The following included .md, sql-database-csharp-adonet-create-query-2.md, is long.
And it starts with a ## H2.
-->

[!INCLUDE [sql-database-csharp-adonet-create-query-2](../../includes/sql-database-csharp-adonet-create-query-2.md)]


## <a name="next-steps"></a>Próximas etapas

Neste tutorial, você aprendeu as tarefas básicas de banco de dados, como criar um banco de dados e tabelas, carregar e consultar dados e restaurar o banco de dados para um ponto anterior no tempo. Você aprendeu como:
> [!div class="checklist"]
> * Criar um banco de dados
> * Configurar uma regra de firewall
> * Conectar-se ao banco de dados com o [Visual Studio e C#](sql-database-connect-query-dotnet-visual-studio.md)
> * Criar tabelas
> * Inserir, atualizar e excluir dados
> * Consultar dados

Avance para o próximo tutorial para saber mais sobre como migrar seus dados.

> [!div class="nextstepaction"]
> [Migrar seu banco de dados do SQL Server para o banco de dados SQL do Azure](sql-database-migrate-your-sql-server-database.md)

