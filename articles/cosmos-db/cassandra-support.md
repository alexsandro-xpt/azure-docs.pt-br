---
title: Recursos do Apache Cassandra e comandos compatíveis com a API do Cassandra do Azure Cosmos DB
description: Saiba mais sobre o suporte a recursos do Apache Cassandra na API do Cassandra do Azure Cosmos DB
services: cosmos-db
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.topic: overview
ms.date: 09/24/2018
ms.openlocfilehash: 70b2c3252380440a491e6d21a8b8a96a2390525b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986261"
---
# <a name="apache-cassandra-features-supported-by-azure-cosmos-db-cassandra-api"></a>Recursos do Apache Cassandra compatíveis com a API do Cassandra do Azure Cosmos DB 

O Azure Cosmos DB é o serviço de banco de dados multimodelo distribuído globalmente da Microsoft. Você pode se comunicar com a API do Cassandra do Azure Cosmos DB por meio de [drivers](http://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver) de cliente do Cassandra de software livre em conformidade com o [protocolo de transmissão](https://github.com/apache/cassandra/blob/trunk/doc/native_protocol_v4.spec) da CQL (linguagem de consulta do Cassandra) v4. 

Ao usar a API do Cassandra do Azure Cosmos DB, você pode aproveitar os benefícios das APIs do Apache Cassandra, bem como as funcionalidades empresariais que o Azure Cosmos DB fornece. As funcionalidades empresariais incluem [distribuição global](distribute-data-globally.md), [particionamento com expansão automática](partition-data.md), garantias de disponibilidade e latência, criptografia em repouso, backups e muito mais.

## <a name="cassandra-protocol"></a>Protocolo do Cassandra 

A API do Cassandra do Azure Cosmos DB é compatível com a versão do CQL **v4**. Os comandos, ferramentas, limitações e exceções do CQL compatíveis estão listados abaixo. Qualquer driver de cliente que compreenda esses protocolos deve ser capaz de conectar-se à API do Cassandra do Azure Cosmos DB.

## <a name="cassandra-driver"></a>Driver do Cassandra

As seguintes versões de drivers datastax do Cassandra são compatíveis com a API do Cassandra do Azure Cosmos DB:

* [Java 3.5+](https://github.com/datastax/java-driver)  
* [C# 3.5+](https://github.com/datastax/csharp-driver)  
* [Nodejs 3.5+](https://github.com/datastax/nodejs-driver)  
* [Python 3.15+](https://github.com/datastax/python-driver)  
* [C++ 2.9](https://github.com/datastax/cpp-driver)   
* [PHP 1.3](https://github.com/datastax/php-driver)  
* [Gocql](https://github.com/gocql/gocql)  
 
## <a name="cql-data-types"></a>Tipos de dados CQL 

A API do Cassandra do Azure Cosmos DB é compatível com os seguintes tipos de dados do CQL:

* ascii  
* bigint  
* blob  
* booleano  
* contador  
* data  
* decimal  
* double  
* flutuante  
* frozen  
* inet  
* int  
* list  
* set  
* smallint  
* text  
* tempo real  
* timestamp  
* timeuuid  
* tinyint  
* tuple  
* uuid  
* varchar  
* varint  
* tuples  
* udts  
* map  

## <a name="cql-functions"></a>Funções do CQL

A API do Cassandra do Azure Cosmos DB é compatível com as seguintes funções do CQL:

* A criptografia do token  
* Funções de conversão de blob 
  * typeAsBlob(value)  
  * blobAsType(value)
* Funções UUID e timeuuid 
  * dateOf()  
  * now()  
  * minTimeuuid()  
  * unixTimestampOf()  
  * toDate(timeuuid)  
  * toTimestamp(timeuuid)  
  * toUnixTimestamp(timeuuid)  
  * toDate(timestamp)  
  * toUnixTimestamp(timestamp)  
  * toTimestamp(date)  
  * toUnixTimestamp(date)  


## <a name="cassandra-query-language-limits"></a>Limites da Linguagem de Consulta do Cassandra

A API do Cassandra do Azure Cosmos DB não tem limites para o tamanho dos dados armazenados em uma tabela. Centenas de terabytes ou petabytes de dados podem ser armazenados, com a garantia de que os limites de chave de partição sejam respeitados. Da mesma forma, cada entidade ou linha equivalente não tem nenhum limite para o número de colunas. No entanto, o tamanho total da entidade não deve exceder 2 MB.

## <a name="tools"></a>Ferramentas 

A API do Cassandra do Azure Cosmos DB é uma plataforma de serviço gerenciado. Ela não requer nenhuma sobrecarga de gerenciamento nem utilitários como o Coletor de Lixo, a JVM (Máquina Virtual Java) e o nodetool para gerenciar o cluster. Ela dá suporte a ferramentas como o cqlsh, que utiliza a compatibilidade binária CQLv4. 

* O Data Explorer do portal do Azure, as Métricas, os Logs de Diagnóstico, o PowerShell e a CLI são outros mecanismos compatíveis para gerenciar a conta.

## <a name="cql-shell"></a>Shell do CQL  

O utilitário de linha de comando CQLSH vem com o Apache Cassandra 3.1.1 e é pronto para uso, funcionando imediatamente e com as seguintes variáveis de ambiente habilitadas:

**Windows:** 

```bash
set SSL_VERSION=TLSv1_2 
SSL_CERTIFICATE=<path to balitmore root ca cert>
set CQLSH_PORT=10350 
cqlsh <YOUR_ACCOUNT_NAME>.cassandra.cosmosdb.azure.com 10350 -u <YOUR_ACCOUNT_NAME> -p <YOUR_ACCOUNT_PASSWORD> –ssl 
```
**UNIX/Linux/Mac:**

```bash
export SSL_VERSION=TLSv1_2 
SSL_CERTIFICATE=<path to balitmore root ca cert>
cqlsh <YOUR_ACCOUNT_NAME>.cassandra.cosmosdb.azure.com 10350 -u <YOUR_ACCOUNT_NAME> -p <YOUR_ACCOUNT_PASSWORD> –ssl 
```

## <a name="cql-commands"></a>Comandos da CQL

O Azure Cosmos DB dá suporte aos seguintes comandos de banco de dados em contas da API do Cassandra.

* CREATE KEYSPACE 

* CREATE TABLE 

* ALTER TABLE 

* USE 

* INSERT 

* SELECIONAR 

* UPDATE 

* BATCH – Somente comandos não registrados são compatíveis 

* EXCLUIR

Todas as operações CRUD, quando executadas por meio de um SDK compatível com CQLV4, retornarão informações adicionais sobre erros, unidades de solicitação consumidas e ID da atividade. Os comandos delete e update precisam ser tratados considerando-se a governança de recursos, para evitar o uso de recursos provisionados. 

```csharp
var tableInsertStatement = table.Insert(sampleEntity); 
var insertResult = await tableInsertStatement.ExecuteAsync(); 
 
foreach (string key in insertResult.Info.IncomingPayload) 
        { 
            byte[] valueInBytes = customPayload[key]; 
            string value = Encoding.UTF8.GetString(valueInBytes); 
            Console.WriteLine($“CustomPayload:  {key}: {value}”); 
        } 
```

## <a name="consistency-mapping"></a>Mapeamento de consistência 

A API do Cassandra do Azure Cosmos DB fornece a opção de consistência para operações de leitura. Todas as operações de gravação, independentemente da consistência de conta, são sempre gravadas com SLAs de desempenho de gravação.

## <a name="permission-and-role-management"></a>Gerenciamento de funções e de permissões

O Azure Cosmos DB dá suporte ao controle de acesso baseado em função (RBAC) e senhas/chaves de leitura/gravação e somente leitura que podem ser obtidas por meio do [portal do Azure](https://portal.azure.com. O Azure Cosmos DB ainda não oferece suporte para usuários e funções para atividades do plano de dados. 

## <a name="next-steps"></a>Próximas etapas

- Introdução à [criação de uma conta de API do Cassandra, banco de dados e uma tabela](create-cassandra-api-account-java.md) pelo uso de um aplicativo Java

