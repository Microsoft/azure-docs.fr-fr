---
title: Sécuriser les informations d’identification d’accès avec les services liés dans Apache Spark pour Azure Synapse Analytics
description: Cet article explique comment intégrer de manière sécurisée Apache Spark pour Azure Synapse Analytics à d’autres services en utilisant les services liés et la bibliothèque de jetons
services: synapse-analytics
author: mlee3gsd
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: spark
ms.date: 11/19/2020
ms.author: martinle
ms.reviewer: nirav
zone_pivot_groups: programming-languages-spark-all-minus-sql
ms.openlocfilehash: 8c2e2327667ccb9284a22e65418a80f3066d22df
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96510792"
---
# <a name="secure-credentials-with-linked-services-using-the-tokenlibrary"></a>Sécuriser les informations d’identification avec les services liés à l’aide de TokenLibrary

L’accès aux données de sources externes est un modèle courant. À moins que la source de données externe autorise l’accès anonyme, il est probable que vous aurez besoin de sécuriser votre connexion avec des informations d’identification, un secret ou une chaîne de connexion.  

Synapse utilise passthrough, le transfert direct, Azure Active Directory (AAD) par défaut pour l’authentification entre les ressources.  Si vous devez vous connecter à une ressource avec d’autres informations d’identification, utilisez TokenLibrary directement.  TokenLibrary simplifie le processus de récupération des jetons SAS, des jetons AAD, des chaînes de connexion et des secrets stockés dans un service lié, ou à partir d’un coffre de clés Azure.

Lorsque vous récupérez des secrets à partir d’Azure Key Vault, nous vous conseillons de créer un service lié à votre coffre de clés Azure.  Vérifiez que les identités managées pour les ressources Azure de l’espace de travail Synapse disposent des privilèges Secret Get sur votre coffre de clés Azure.  Synapse s’authentifie auprès d’Azure Key Vault au moyen des identités managées pour les ressources Azure de l’espace de travail Synapse. Si vous vous connectez directement à Azure Key Vault sans service lié, vous devez vous authentifier avec vos informations d’identification d’utilisateur Azure Active Directory.

Pour plus d’informations, consultez [services liés](../../data-factory/concepts-linked-services.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="usage"></a>Usage

### <a name="tokenlibraryhelp"></a>TokenLibrary.help()
Cette fonction affiche la documentation d’aide de TokenLibrary.

::: zone pivot = "programming-language-scala"

```scala
TokenLibrary.help()
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
TokenLibrary.help()
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
Console.WriteLine(TokenLibrary.help());
```

::: zone-end

## <a name="tokenlibrary-for-azure-data-lake-storage-gen2"></a>TokenLibrary pour Azure Data Lake Storage Gen2

#### <a name="adls-gen2-primary-storage"></a>Stockage principal ADLS Gen2

L’accès aux fichiers depuis l’instance Azure Data Lake Storage principale utilise le passthrough Azure Active Directory pour l’authentification par défaut, et n’a pas besoin de l’utilisation explicite de TokenLibrary.

::: zone pivot = "programming-language-scala"

```scala
val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")
display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')
display(df.limit(10))
```

::: zone-end

#### <a name="adls-gen2-storage-with-linked-services"></a>Stockage ADLS Gen2 avec les services liés

Synapse propose une expérience de services liés intégrée lors de la connexion à Azure Data Lake Storage Gen2.  Les services liés peuvent être configurés pour s’authentifier à l’aide d’une **clé de compte**, d’un **principal du service**, d’une **identité managée** ou d’**informations d’identification**.

Lorsque la méthode d’authentification du service lié est définie sur **Clé de compte**, le service lié s’authentifie au moyen de la clé de compte de stockage fournie, il demande une clé SAS et l’applique automatiquement à la requête de stockage à l’aide de **LinkedServiceBasedSASProvider**.

::: zone pivot = "programming-language-scala"

```scala
val sc = spark.sparkContext
spark.conf.set("spark.storage.synapse.linkedServiceName", "<LINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedSASProvider")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# Python code
spark.conf.set("spark.storage.synapse.linkedServiceName", "<lINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedSASProvider")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<DIRECTORY PATH>')

df.show()
```

::: zone-end

Lorsque la méthode d’authentification du service lié est définie sur **Identité managée** ou **Principal du service**, le service lié utilise l’identité managée ou le jeton du principal du service avec le fournisseur **LinkedServiceBasedTokenProvider**.  


::: zone pivot = "programming-language-scala"

```scala
val sc = spark.sparkContext
spark.conf.set("spark.storage.synapse.linkedServiceName", "<LINKED SERVICE NAME>")
spark.conf.set("fs.azure.account.oauth.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedTokenProvider") 
val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# Python code
spark.conf.set("spark.storage.synapse.linkedServiceName", "<lINKED SERVICE NAME>")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedTokenProvider")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<DIRECTORY PATH>')

df.show()
```

::: zone-end

#### <a name="adls-gen2-storage-without-linked-services"></a>Stockage ADLS Gen2 (sans service lié)

Connectez-vous au stockage ADLS Gen2 directement à l’aide d’une clé SAS, utilisez **ConfBasedSASProvider** et fournissez la clé SAS au paramètre de configuration **spark.storage.synapse.sas**.

::: zone pivot = "programming-language-scala"

```scala
%%spark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.ConfBasedSASProvider")
spark.conf.set("spark.storage.synapse.sas", "<SAS KEY>")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark

spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.ConfBasedSASProvider")
spark.conf.set("spark.storage.synapse.sas", "<SAS KEY>")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')

display(df.limit(10))
```

::: zone-end

#### <a name="adls-gen2-storage-with-azure-key-vault"></a>Stockage ADLS Gen2 avec Azure Key Vault

Connectez-vous au stockage ADLS Gen2 à l’aide d’un jeton SAS stocké dans le secret Azure Key Vault.  

::: zone pivot = "programming-language-scala"

```scala
%%spark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.AkvBasedSASProvider")
spark.conf.set("spark.storage.synapse.akv", "<AZURE KEY VAULT NAME>")
spark.conf.set("spark.storage.akv.secret", "<SECRET KEY>")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>")

display(df.limit(10))
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
spark.conf.set("fs.azure.account.auth.type", "SAS")
spark.conf.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.AkvBasedSASProvider")
spark.conf.set("spark.storage.synapse.akv", "<AZURE KEY VAULT NAME>")
spark.conf.set("spark.storage.akv.secret", "<SECRET KEY>")

df = spark.read.csv('abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<FILE PATH>')

display(df.limit(10))
```

::: zone-end

## <a name="tokenlibrary-for-other-linked-services"></a>TokenLibrary pour d’autres services liés

Pour vous connecter à d’autres services liés, vous pouvez effectuer un appel direct à TokenLibrary.

#### <a name="getconnectionstring"></a>getConnectionString()

 Pour récupérer la chaîne de connexion, utilisez la fonction **getConnectionString** et transmettez le **nom du service lié**.

::: zone pivot = "programming-language-scala"

```scala
%%spark
// retrieve connectionstring from TokenLibrary

import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val connectionString: String = TokenLibrary.getConnectionString("<LINKED SERVICE NAME>")
println(connectionString)
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
%%pyspark
# retrieve connectionstring from TokenLibrary

from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary
connection_string = token_library.getConnectionString("<LINKED SERVICE NAME>")
print(connection_string)
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
%%csharp
// retrieve connectionstring from TokenLibrary

using Microsoft.Spark.Extensions.Azure.Synapse.Analytics.Utils;

string connectionString = TokenLibrary.GetConnectionString(<LINKED SERVICE NAME>);
Console.WriteLine(connectionString);
```

::: zone-end

#### <a name="getconnectionstringasmap"></a>getConnectionStringAsMap()

getConnectionStringAsMap est une fonction d’assistance disponible dans Scala et Python pour analyser des valeurs spécifiques à partir d’une paire _key=value_ dans la chaîne de connexion, par exemple

_`DefaultEndpointsProtocol=https;AccountName=\<ACCOUNT NAME>;AccountKey=\<ACCOUNT KEY>`_

utilisez la fonction **getConnectionStringAsMap** et transmettez la clé pour retourner la valeur.  Dans l’exemple de chaîne de connexion ci-dessus, 

_**TokenLibrary.getConnectionStringAsMap("DefaultEndpointsProtocol")**_

retournerait

**_"https"_**

::: zone pivot = "programming-language-scala"

```scala
// Linked services can be used for storing and retrieving credentials (e.g, account key)
// Example connection string (for storage): "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val accountKey: String = TokenLibrary.getConnectionStringAsMap("<LINKED SERVICE NAME">).get("<KEY NAME>")
println(accountKey)
```
::: zone-end

::: zone pivot = "programming-language-python"

```python
# Linked services can be used for storing and retrieving credentials (e.g, account key)
# Example connection string (for storage): "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary
accountKey = token_library.getConnectionStringAsMap("<LINKED SERVICE NAME>").get("<KEY NAME>")
print(accountKey)
```

::: zone-end

#### <a name="getsecret"></a>getSecret()

Pour récupérer un secret stocké depuis Azure Key Vault, nous vous recommandons de créer un service lié à Azure Key Vault au sein de l’espace de travail Synapse. L’autorisation **GET** Secrets au coffre de clés Azure devra être accordée aux identités managées pour les ressources Azure de l’espace de travail Synapse.  Le service lié utilisera les identités managées pour les ressources Azure afin de se connecter au service Azure Key Vault et de récupérer le secret.  Sinon, la connexion directe à Azure Key Vault utilisera les informations d’identification de l’utilisateur Azure Active Directory (AAD).  Dans ce cas, l’utilisateur devra disposer des autorisations Get Secret dans Azure Key Vault.

`TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>" [, <LINKED SERVICE NAME>])`

Pour récupérer un secret à partir d’Azure Key Vault, utilisez la fonction **TokenLibrary.getSecret()** .

::: zone pivot = "programming-language-scala"

```scala
import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val connectionString: String = TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>")
println(connectionString)
```

::: zone-end

::: zone pivot = "programming-language-python"

```python
import sys
from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

connection_string = token_library.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>")
print(connection_string)
```

::: zone-end

::: zone pivot = "programming-language-csharp"

```csharp
using Microsoft.Spark.Extensions.Azure.Synapse.Analytics.Utils;

string connectionString = TokenLibrary.getSecret("<AZURE KEY VAULT NAME>", "<SECRET KEY>", "<LINKED SERVICE NAME>");
Console.WriteLine(connectionString);
```

::: zone-end

## <a name="next-steps"></a>Étapes suivantes

- [Écrire dans un pool SQL dédié](./synapse-spark-sql-pool-import-export.md)
