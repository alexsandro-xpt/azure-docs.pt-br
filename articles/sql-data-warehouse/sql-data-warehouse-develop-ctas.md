---
title: CTAS (CREATE TABLE AS SELECT) no SQL Data Warehouse do Azure | Microsoft Docs
description: Dicas para codificação com a instrução CTAS (CREATE TABLE AS SELECT) no SQL Data Warehouse do Azure para desenvolvimento de soluções.
services: sql-data-warehouse
author: ckarst
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: cakarst
ms.reviewer: igorstan
ms.openlocfilehash: dad0b1570f54cde1b1d474d8ebfc78f793724ef4
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/30/2018
ms.locfileid: "43247746"
---
# <a name="using-create-table-as-select-ctas-in-azure-sql-data-warehouse"></a>Usando o CTAS (CREATE TABLE AS SELECT) no SQL Data Warehouse do Azure
Dicas para codificação com a instrução do T-SQL CTAS (CREATE TABLE AS SELECT) no SQL Data Warehouse do Azure para desenvolvimento de soluções.

## <a name="what-is-create-table-as-select-ctas"></a>O que é o CTAS (CREATE TABLE AS SELECT)?

A instrução CTAS ou [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) é um dos recursos do T-SQL mais importantes disponíveis. Esta é uma operação paralela que cria uma nova tabela com base na saída de uma instrução SELECT. CTASD é a maneira mais rápida e simples de criar uma cópia de uma tabela. 

## <a name="selectinto-vs-ctas"></a>SELECT..INTO versus CTAS
Você pode considerar a CTAS como uma versão aprimorada da instrução [SELECT...INTO](/sql/t-sql/queries/select-into-clause-transact-sql).

Veja a seguir um exemplo de uma SELECT...INTO simples:

```sql
SELECT *
INTO    [dbo].[FactInternetSales_new]
FROM    [dbo].[FactInternetSales]
```

No exemplo anterior, `[dbo].[FactInternetSales_new]` é criado como uma tabela distribuída por ROUND_ROBIN com um CLUSTERED COLUMNSTORE INDEX, visto que esses são os padrões de tabela no SQL Data Warehouse do Azure.

No entanto, SELECT...INTO não permite alterar o método de distribuição nem o tipo de índice como parte da operação. É aí que a CTAS entra em cena.

É muito simples converter o exemplo anterior em CTAS:

```sql
CREATE TABLE [dbo].[FactInternetSales_new]
WITH
(
    DISTRIBUTION = ROUND_ROBIN
,   CLUSTERED COLUMNSTORE INDEX
)
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
;
```

Com a CTAS, é possível alterar a distribuição dos dados da tabela, bem como o tipo de tabela. 

> [!NOTE]
> Se você estiver apenas tentando alterar o índice em sua operação `CTAS`, e a tabela de origem for distribuída por hash, sua operação `CTAS` executará melhor se você mantiver o mesmo tipo de coluna e dados de distribuição. Isso evitará a movimentação de dados de distribuição cruzada durante a operação, o que é mais eficiente.
> 
> 

## <a name="using-ctas-to-copy-a-table"></a>Usando o CTAS para copiar uma tabela
Talvez um dos usos mais comuns do `CTAS` seja criar uma cópia de uma tabela para que você possa alterar a DDL. Se, por exemplo, você originalmente tiver criado a tabela como `ROUND_ROBIN` e agora quiser alterá-la para uma tabela distribuída em uma coluna, `CTAS` é como você alteraria a coluna de distribuição. `CTAS` também pode ser usado para alterar os tipos de particionamento, indexação ou coluna.

Digamos que você tenha criado essa tabela usando o tipo de distribuição padrão do `ROUND_ROBIN` distribuído, já que nenhuma coluna de distribuição foi especificada em `CREATE TABLE`.

```sql
CREATE TABLE FactInternetSales
(
    ProductKey int NOT NULL,
    OrderDateKey int NOT NULL,
    DueDateKey int NOT NULL,
    ShipDateKey int NOT NULL,
    CustomerKey int NOT NULL,
    PromotionKey int NOT NULL,
    CurrencyKey int NOT NULL,
    SalesTerritoryKey int NOT NULL,
    SalesOrderNumber nvarchar(20) NOT NULL,
    SalesOrderLineNumber tinyint NOT NULL,
    RevisionNumber tinyint NOT NULL,
    OrderQuantity smallint NOT NULL,
    UnitPrice money NOT NULL,
    ExtendedAmount money NOT NULL,
    UnitPriceDiscountPct float NOT NULL,
    DiscountAmount float NOT NULL,
    ProductStandardCost money NOT NULL,
    TotalProductCost money NOT NULL,
    SalesAmount money NOT NULL,
    TaxAmt money NOT NULL,
    Freight money NOT NULL,
    CarrierTrackingNumber nvarchar(25),
    CustomerPONumber nvarchar(25)
);
```

Agora, você deseja criar uma nova cópia da tabela com um Índice Columnstore Clusterizado para que possa aproveitar o desempenho das tabelas Columnstore Clusterizado. Você também deseja distribuir essa tabela na ProductKey, uma vez que está antecipando as associações nessa coluna e quer evitar a movimentação dos dados durante as junções em ProductKey. Por fim, também deseja adicionar o particionamento em OrderDateKey para que possa excluir rapidamente os dados antigos descartando as partições antigas. Aqui está a declaração CTAS que copiaria sua tabela antiga para uma nova tabela.

```sql
CREATE TABLE FactInternetSales_new
WITH
(
    CLUSTERED COLUMNSTORE INDEX,
    DISTRIBUTION = HASH(ProductKey),
    PARTITION
    (
        OrderDateKey RANGE RIGHT FOR VALUES
        (
        20000101,20010101,20020101,20030101,20040101,20050101,20060101,20070101,20080101,20090101,
        20100101,20110101,20120101,20130101,20140101,20150101,20160101,20170101,20180101,20190101,
        20200101,20210101,20220101,20230101,20240101,20250101,20260101,20270101,20280101,20290101
        )
    )
)
AS SELECT * FROM FactInternetSales;
```

Finalmente, você pode renomear suas tabelas para trocar na nova tabela, em seguida, descartar a tabela antiga.

```sql
RENAME OBJECT FactInternetSales TO FactInternetSales_old;
RENAME OBJECT FactInternetSales_new TO FactInternetSales;

DROP TABLE FactInternetSales_old;
```

> [!NOTE]
> O SQL Data Warehouse do Azure ainda não dá suporte às estatísticas de criação ou atualização automática.  Para obter o melhor desempenho de suas consultas, é importante que as estatísticas sejam criadas em todas as colunas de todas as tabelas após o primeiro carregamento ou após uma alteração significativa nos dados.  Para obter uma explicação detalhada das estatísticas, confira o tópico [Statistics][Statistics] no grupo de tópicos Desenvolver.
> 
> 

## <a name="using-ctas-to-work-around-unsupported-features"></a>Usando o CTAS para resolver os recursos sem suporte
A CTAS também pode ser usada como solução alternativa dos diversos recursos incompatíveis listados abaixo. Isso pode ser geralmente uma situação de ganho mútuo pois não apenas o seu código estará compatível, mas ele será executado muitas vezes mais rapidamente no SQL Data Warehouse. Isso é devido ao seu design totalmente em paralelo. Os cenários que podem ser solucionados com CTAS incluem:

* ANSI JOINS em instruções UPDATE
* ANSI JOINs em instruções DELETE
* Instrução MERGE

> [!NOTE]
> Tente imaginar “CTAS primeiro”. Caso você ache que pode resolver um problema usando `CTAS` , que geralmente é a melhor maneira de abordá-lo, mesmo se você está escrevendo mais dados como um resultado.
> 
> 

## <a name="ansi-join-replacement-for-update-statements"></a>Substituição da junção ANSI para atualizar instruções
Você pode descobrir uma atualização complexa que une mais de duas tabelas usando a sintaxe da junção ANSI para executar a UPDATE ou DELETE.

Imagine que você precisava atualizar esta tabela:

```sql
CREATE TABLE [dbo].[AnnualCategorySales]
(    [EnglishProductCategoryName]    NVARCHAR(50)    NOT NULL
,    [CalendarYear]                    SMALLINT        NOT NULL
,    [TotalSalesAmount]                MONEY            NOT NULL
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN
)
;
```

A consulta original pode se parecer com isso:

```sql
UPDATE    acs
SET        [TotalSalesAmount] = [fis].[TotalSalesAmount]
FROM    [dbo].[AnnualCategorySales]     AS acs
JOIN    (
        SELECT    [EnglishProductCategoryName]
        ,        [CalendarYear]
        ,        SUM([SalesAmount])                AS [TotalSalesAmount]
        FROM    [dbo].[FactInternetSales]        AS s
        JOIN    [dbo].[DimDate]                    AS d    ON s.[OrderDateKey]                = d.[DateKey]
        JOIN    [dbo].[DimProduct]                AS p    ON s.[ProductKey]                = p.[ProductKey]
        JOIN    [dbo].[DimProductSubCategory]    AS u    ON p.[ProductSubcategoryKey]    = u.[ProductSubcategoryKey]
        JOIN    [dbo].[DimProductCategory]        AS c    ON u.[ProductCategoryKey]        = c.[ProductCategoryKey]
        WHERE     [CalendarYear] = 2004
        GROUP BY
                [EnglishProductCategoryName]
        ,        [CalendarYear]
        ) AS fis
ON    [acs].[EnglishProductCategoryName]    = [fis].[EnglishProductCategoryName]
AND    [acs].[CalendarYear]                = [fis].[CalendarYear]
;
```

Já que o SQL Data Warehouse não dá suporte a junções ANSI na cláusula `FROM` da instrução `UPDATE`, você não pode copiar esse código sem alterá-lo um pouco.

Você pode usar uma combinação de um `CTAS` e uma junção implícita para substituir este código:

```sql
-- Create an interim table
CREATE TABLE CTAS_acs
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT    ISNULL(CAST([EnglishProductCategoryName] AS NVARCHAR(50)),0)    AS [EnglishProductCategoryName]
,        ISNULL(CAST([CalendarYear] AS SMALLINT),0)                         AS [CalendarYear]
,        ISNULL(CAST(SUM([SalesAmount]) AS MONEY),0)                        AS [TotalSalesAmount]
FROM    [dbo].[FactInternetSales]        AS s
JOIN    [dbo].[DimDate]                    AS d    ON s.[OrderDateKey]                = d.[DateKey]
JOIN    [dbo].[DimProduct]                AS p    ON s.[ProductKey]                = p.[ProductKey]
JOIN    [dbo].[DimProductSubCategory]    AS u    ON p.[ProductSubcategoryKey]    = u.[ProductSubcategoryKey]
JOIN    [dbo].[DimProductCategory]        AS c    ON u.[ProductCategoryKey]        = c.[ProductCategoryKey]
WHERE     [CalendarYear] = 2004
GROUP BY
        [EnglishProductCategoryName]
,        [CalendarYear]
;

-- Use an implicit join to perform the update
UPDATE  AnnualCategorySales
SET     AnnualCategorySales.TotalSalesAmount = CTAS_ACS.TotalSalesAmount
FROM    CTAS_acs
WHERE   CTAS_acs.[EnglishProductCategoryName] = AnnualCategorySales.[EnglishProductCategoryName]
AND     CTAS_acs.[CalendarYear]               = AnnualCategorySales.[CalendarYear]
;

--Drop the interim table
DROP TABLE CTAS_acs
;
```

## <a name="ansi-join-replacement-for-delete-statements"></a>Substituição de junção ANSI para instruções delete
Às vezes, a melhor abordagem para a exclusão de dados é usar `CTAS`. Em vez de excluir os dados, simplesmente selecione os dados que você deseja manter. Isso é especialmente verdadeiro para instruções `DELETE` que usam sintaxe de junção ansi, já que o SQL Data Warehouse não dá suporte a junções ANSI na cláusula `FROM` de uma instrução `DELETE`.

Um exemplo de uma instrução DELETE convertida está disponível abaixo:

```sql
CREATE TABLE dbo.DimProduct_upsert
WITH
(   Distribution=HASH(ProductKey)
,   CLUSTERED INDEX (ProductKey)
)
AS -- Select Data you wish to keep
SELECT     p.ProductKey
,          p.EnglishProductName
,          p.Color
FROM       dbo.DimProduct p
RIGHT JOIN dbo.stg_DimProduct s
ON         p.ProductKey = s.ProductKey
;

RENAME OBJECT dbo.DimProduct        TO DimProduct_old;
RENAME OBJECT dbo.DimProduct_upsert TO DimProduct;
```

## <a name="replace-merge-statements"></a>Substituir instruções merge
Instruções de mesclagem podem ser substituídas, pelo menos em parte, usando CTAS. É possível consolidar INSERT e UPDATE em uma única instrução. Quaisquer registros excluídos precisariam ser fechados em uma segunda instrução.

Veja a seguir um exemplo de um UPSERT:

```sql
CREATE TABLE dbo.[DimProduct_upsert]
WITH
(   DISTRIBUTION = HASH([ProductKey])
,   CLUSTERED INDEX ([ProductKey])
)
AS
-- New rows and new versions of rows
SELECT      s.[ProductKey]
,           s.[EnglishProductName]
,           s.[Color]
FROM      dbo.[stg_DimProduct] AS s
UNION ALL  
-- Keep rows that are not being touched
SELECT      p.[ProductKey]
,           p.[EnglishProductName]
,           p.[Color]
FROM      dbo.[DimProduct] AS p
WHERE NOT EXISTS
(   SELECT  *
    FROM    [dbo].[stg_DimProduct] s
    WHERE   s.[ProductKey] = p.[ProductKey]
)
;

RENAME OBJECT dbo.[DimProduct]          TO [DimProduct_old];
RENAME OBJECT dbo.[DimpProduct_upsert]  TO [DimProduct];

```

## <a name="ctas-recommendation-explicitly-state-data-type-and-nullability-of-output"></a>Recomendação de CTAS: declarar explicitamente o tipo de dados e a nulidade da saída
Ao migrar o código, você pode achar executar por esse tipo de padrão de codificação:

```sql
DECLARE @d decimal(7,2) = 85.455
,       @f float(24)    = 85.455

CREATE TABLE result
(result DECIMAL(7,2) NOT NULL
)
WITH (DISTRIBUTION = ROUND_ROBIN)

INSERT INTO result
SELECT @d*@f
;
```

Instintivamente, é possível pensar que é necessário migrar esse código para um CTAS e você estaria certo. No entanto, há um problema oculto aqui.

O código a seguir NÃO produz o mesmo resultado:

```sql
DECLARE @d decimal(7,2) = 85.455
,       @f float(24)    = 85.455
;

CREATE TABLE ctas_r
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT @d*@f as result
;
```

Observe que a coluna “resultado” transporta os valores de tipo e a nulidade de dados da expressão. Se você não tiver cuidado, isso pode levar a variações sutis em valores.

Tente o seguinte exemplo:

```sql
SELECT result,result*@d
from result
;

SELECT result,result*@d
from ctas_r
;
```

O valor armazenado para o resultado é diferente. Como o valor persistente na coluna de resultado é usado em outras expressões, o erro se torna ainda mais significativo.

![Resultados de CTAS](media/sql-data-warehouse-develop-ctas/ctas-results.png)

Isso é particularmente importante para migrações de dados. Mesmo que a segunda consulta seja indiscutivelmente mais precisa, há um problema. Os dados seriam diferentes em comparação com o sistema de origem e que leva a questões de integridade da migração. Esse é um dos casos raros em que a resposta “incorreta” é realmente a melhor!

O motivo que vemos esta disparidade entre os dois resultados é a conversão de tipo implícito. No primeiro exemplo, a tabela define a definição de coluna. Quando a linha é inserida uma conversão implícita de tipo ocorre. No segundo exemplo não há nenhuma conversão implícita de tipo como a expressão define o tipo de dados da coluna. Observe também que a coluna no segundo exemplo tenha sido definida como uma coluna anulável, enquanto no primeiro exemplo não tem. Quando a tabela foi criada no primeiro exemplo, a nulidade da coluna foi explicitamente definida. No segundo exemplo, ela estava à esquerda da expressão e, por padrão isso resultaria em uma definição NULL.  

Para resolver esses problemas, é necessário definir explicitamente a conversão de tipo e a nulidade na parte SELECT da instrução CTAS. Você não pode definir essas propriedades na parte criar tabela.

O exemplo a seguir demonstra como corrigir o código:

```sql
DECLARE @d decimal(7,2) = 85.455
,       @f float(24)    = 85.455

CREATE TABLE ctas_r
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT ISNULL(CAST(@d*@f AS DECIMAL(7,2)),0) as result
```

Observe o seguinte:

* CAST ou CONVERT poderia ter sido usado
* ISNULL é usado para forçar NULLability não COALESCE
* ISNULL é a função mais externa
* A segunda parte do ISNULL é uma constante, ou seja, 0

> [!NOTE]
> Para a nulidade ser definida corretamente, é vital usar ISNULL, e não COALESCE. COALESCE não é uma função determinística e, por isso, o resultado da expressão sempre será anulável. Com ISNULL, é diferente. É determinístico. Portanto, quando a segunda parte da função ISNULL for uma constante ou um literal, o valor resultante será NOT NULL.
> 
> 

Esta dica não é apenas útil para assegurar a integridade de seus cálculos. Também é importante para a alternância de partição de tabela. Imagine que você tenha essa tabela definida como o caso:

```sql
CREATE TABLE [dbo].[Sales]
(
    [date]      INT     NOT NULL
,   [product]   INT     NOT NULL
,   [store]     INT     NOT NULL
,   [quantity]  INT     NOT NULL
,   [price]     MONEY   NOT NULL
,   [amount]    MONEY   NOT NULL
)
WITH
(   DISTRIBUTION = HASH([product])
,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101,20020101
                    ,20030101,20040101,20050101
                    )
                )
)
;
```

No entanto, o campo de valor é uma expressão calculada que não é parte dos dados de origem.

Para criar o conjunto de dados particionado, você poderia fazer isso:

```sql
CREATE TABLE [dbo].[Sales_in]
WITH    
(   DISTRIBUTION = HASH([product])
,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101
                    )
                )
)
AS
SELECT
    [date]    
,   [product]
,   [store]
,   [quantity]
,   [price]   
,   [quantity]*[price]  AS [amount]
FROM [stg].[source]
OPTION (LABEL = 'CTAS : Partition IN table : Create')
;
```

A consulta seria executada perfeitamente bem. O problema surge quando você tenta executar a opção de partição. As definições de tabela não correspondem. Para fazer com que as definições de tabela correspondam, a CTAS precisa ser modificada.

```sql
CREATE TABLE [dbo].[Sales_in]
WITH    
(   DISTRIBUTION = HASH([product])
,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101
                    )
                )
)
AS
SELECT
    [date]    
,   [product]
,   [store]
,   [quantity]
,   [price]   
,   ISNULL(CAST([quantity]*[price] AS MONEY),0) AS [amount]
FROM [stg].[source]
OPTION (LABEL = 'CTAS : Partition IN table : Create');
```

Você pode ver, portanto, que a consistência de tipo e manutenção das propriedades de nulidade em um CTAS é uma boa prática recomendada de desenvolvimento. Ela ajuda a manter a integridade de seus cálculos e também garante que a alternância de partição é possível.

Consulte a documentação de [CTAS](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse). Ela é uma das instruções mais importantes no SQL Data Warehouse do Azure. Certifique-se compreendê-la totalmente.

## <a name="next-steps"></a>Próximas etapas
Para obter mais dicas de desenvolvimento, confira [visão geral de desenvolvimento](sql-data-warehouse-overview-develop.md).

