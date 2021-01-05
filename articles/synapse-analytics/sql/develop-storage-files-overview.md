---
title: Accéder aux fichiers sur le stockage dans le pool SQL serverless
description: Décrit l’interrogation de fichiers de stockage en utilisant le pool SQL serverless dans Azure Synapse Analytics.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/19/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 82a4ded3a64a8a8bbc62f99a8854eb6d2b5f0d0b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96446583"
---
# <a name="access-external-storage-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Accéder à un stockage externe en utilisant le pool SQL serverless dans Azure Synapse Analytics

Ce document décrit comment les utilisateurs peuvent lire des données dans des fichiers stockés sur Stockage Azure dans le pool SQL serverless. Les utilisateurs disposent des options suivantes pour accéder au stockage :

- Fonction [OPENROWSET](develop-openrowset.md), qui permet des requêtes ad hoc sur les fichiers dans le Stockage Azure.
- [Table externe](develop-tables-external-tables.md), qui est une structure de données prédéfinie reposant sur un ensemble de fichiers externes.

L’utilisateur peut recourir à [différentes méthodes d’authentification](develop-storage-files-storage-access-control.md) telles que l’authentification directe Azure AD (option par défaut pour les principaux Azure AD) et l’authentification SAS (option par défaut pour les principaux SQL).

## <a name="query-files-using-openrowset"></a>Interroger des fichiers à l’aide d’OPENROWSET

OPENROWSET permet aux utilisateurs d’interroger des fichiers externes sur Stockage Azure s’ils ont accès au stockage. L’utilisateur qui est connecté au pool SQL serverless doit utiliser la requête suivante pour lire le contenu des fichiers sur Stockage Azure :

```sql
SELECT * FROM
 OPENROWSET(BULK 'https://<storage_account>.dfs.core.windows.net/<container>/<path>/*.parquet', format= 'parquet') as rows
```

L’utilisateur peut accéder au stockage à l’aide des règles d’accès suivantes :

- Utilisateur Azure AD : `OPENROWSET` utilise l’identité Azure AD de l’appelant pour accéder au Stockage Azure ou pour accéder au stockage avec un accès anonyme.
- Utilisateur SQL : `OPENROWSET` utilise un accès anonyme pour accéder au stockage ou peut être représenté à l’aide d’un jeton SAP ou d’une identité managée de l’espace de travail.

### <a name="impersonation"></a>[Emprunt d'identité](#tab/impersonation)

Les principaux SQL peuvent également utiliser OPENROWSET pour interroger directement des fichiers protégés par des jetons SAS ou l’identité managée de l’espace de travail. Si un utilisateur SQL exécute cette fonction, un utilisateur avancé disposant de l’autorisation `ALTER ANY CREDENTIAL` doit créer des informations d’identification de portée serveur qui correspondent à l’URL dans la fonction (à l’aide du nom et du conteneur de stockage) et accorder l’autorisation REFERENCES pour ces informations d’identification à l’appelant de la fonction OPENROWSET :

```sql
EXECUTE AS somepoweruser

CREATE CREDENTIAL [https://<storage_account>.dfs.core.windows.net/<container>]
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 'sas token';

GRANT REFERENCES CREDENTIAL::[https://<storage_account>.dfs.core.windows.net/<container>] TO sqluser
```

S’il n’existe aucune information d’identification (CREDENTIAL) au niveau du serveur correspondant à l’URL, ou que l’utilisateur SQL ne dispose pas de l’autorisation REFERENCES pour ces informations d’identification, une erreur est retournée. Les principaux SQL ne peuvent pas emprunter d’identité à l’aide d’une identité Azure AD.

### <a name="direct-access"></a>[Accès direct](#tab/direct-access)

Aucune configuration supplémentaire n’est requise pour permettre aux utilisateurs Azure AD d’accéder aux fichiers à l’aide de leurs identités.
Tout utilisateur peut accéder au Stockage Azure qui autorise l’accès anonyme (aucune configuration supplémentaire n’est nécessaire).

---

> [!NOTE]
> Cette version d’OPENROWSET est conçue pour permettre d’explorer les données rapidement et aisément à l’aide de l’authentification par défaut. Pour tirer parti de l’emprunt d’identité ou de l’identité managée, utilisez OPENROWSET avec DATASOURCE, comme décrit dans la section suivante.

## <a name="query-data-sources-using-openrowset"></a>Interroger des sources de données avec OPENROWSET

OPENROWSET permet à l’utilisateur d’interroger les fichiers placés sur une source de données externe :

```sql
SELECT * FROM
 OPENROWSET(BULK 'file/path/*.parquet',
 DATASOURCE = MyAzureInvoices,
 FORMAT= 'parquet') as rows
```

L’utilisateur qui exécute cette requête doit être en mesure d’accéder aux fichiers. Les utilisateurs doivent être emprunter l’identité à l’aide d’un [jeton SAS](develop-storage-files-storage-access-control.md?tabs=shared-access-signature) ou d’une [identité managée de l’espace de travail](develop-storage-files-storage-access-control.md?tabs=managed-identity) s’ils ne peuvent accéder directement aux fichiers avec leur [identité Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity) ou l’[accès anonyme](develop-storage-files-storage-access-control.md?tabs=public-access).

### <a name="impersonation"></a>[Emprunt d'identité](#tab/impersonation)

`DATABASE SCOPED CREDENTIAL` spécifie comment accéder aux fichiers sur la source de données référencée (SAS et identité managée). Un utilisateur avancé disposant de l’autorisation `CONTROL DATABASE` doit créer `DATABASE SCOPED CREDENTIAL` permettant d’accéder au stockage et à `EXTERNAL DATA SOURCE` qui spécifie l’URL de la source de données et les informations d’identification à utiliser :

```sql
EXECUTE AS somepoweruser;

-- Create MASTER KEY if it doesn't exists in database
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'some very strong password';

CREATE DATABASE SCOPED CREDENTIAL AccessAzureInvoices
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
 SECRET = '******srt=sco&amp;sp=rwac&amp;se=2017-02-01T00:55:34Z&amp;st=201********' ;

CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>/' ,
 CREDENTIAL = AccessAzureInvoices) ;
```

L’appelant doit disposer de l’une des autorisations suivantes pour exécuter la fonction OPENROWSET :

- L’une des autorisations pour exécuter OPENROWSET :
  - `ADMINISTER BULK OPERATIONS` permet à la connexion d’exécuter la fonction OPENROWSET.
  - `ADMINISTER DATABASE BULK OPERATIONS` permet à l’utilisateur de portée base de données d’exécuter la fonction OPENROWSET.
- `REFERENCES DATABASE SCOPED CREDENTIAL` pour les informations d’identification référencées dans `EXTERNAL DATA SOURCE`.

### <a name="direct-access"></a>[Accès direct](#tab/direct-access)

L’utilisateur peut créer une source de données externe (EXTERNAL DATA SOURCE) sans informations d’identification (CREDENTIAL) qui référence le stockage d’accès public ou utiliser l’authentification directe Azure AD :

```sql
CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>') ;
```
---
## <a name="external-table"></a>EXTERNAL TABLE

L’utilisateur disposant des autorisations de lecture de table peut accéder à des fichiers externes à l’aide d’une table externe (EXTERNAL TABLE) créée par-dessus l’ensemble de dossiers et de fichiers Stockage Azure.

L’utilisateur qui dispose des [autorisations de créer une table externe](https://docs.microsoft.com/sql/t-sql/statements/create-external-table-transact-sql?view=sql-server-ver15#permissions&preserve-view=true) (par exemple CREATE TABLE et ALTER ANY CREDENTIAL ou REFERENCES DATABASE SCOPED CREDENTIAL) peut utiliser le script suivant pour créer une table sur la source de données du Stockage Azure :

```sql
CREATE EXTERNAL TABLE [dbo].[DimProductexternal]
( ProductKey int, ProductLabel nvarchar, ProductName nvarchar )
WITH
(
LOCATION='/DimProduct/year=*/month=*' ,
DATA_SOURCE = AzureDataLakeStore ,
FILE_FORMAT = TextFileFormat
) ;
```

L’utilisateur qui lit les données de cette table doit être en mesure d’accéder aux fichiers. Les utilisateurs doivent être impersonnés à l’aide d'un [jeton SAS](develop-storage-files-storage-access-control.md?tabs=shared-access-signature) ou d’une [identité managée de l’espace de travail](develop-storage-files-storage-access-control.md?tabs=managed-identity) s’ils ne peuvent accéder directement aux fichiers avec leur [identité Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity) ou l’[accès anonyme](develop-storage-files-storage-access-control.md?tabs=public-access).

### <a name="impersonation"></a>[Emprunt d'identité](#tab/impersonation)

DATABASE SCOPED CREDENTIAL spécifie comment accéder aux fichiers sur la source de données référencée. L’utilisateur disposant de l’autorisation CONTROL DATABASE doit créer des informations d’identification de portée base de données (DATABASE SCOPED CREDENTIAL) qui permettent d’accéder au stockage et à la source de données externe (EXTERNAL DATA SOURCE) qui spécifie l’URL de la source de données et les informations d’identification à utiliser :

```sql
EXECUTE AS somepoweruser;

-- Create MASTER KEY if it doesn't exists in database
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'some very strong password';

CREATE DATABASE SCOPED CREDENTIAL cred
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
 SECRET = '******srt=sco&sp=rwac&se=2017-02-01T00:55:34Z&st=201********' ;

CREATE EXTERNAL DATA SOURCE AzureDataLakeStore
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>' ,
 CREDENTIAL = cred
 ) ;
```

### <a name="direct-access"></a>[Accès direct](#tab/direct-access)

L’utilisateur peut créer une source de données externe (EXTERNAL DATA SOURCE) sans informations d’identification (CREDENTIAL) qui référence le stockage d’accès public ou utiliser l’authentification directe Azure AD :

```sql
CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>') ;
```
---

### <a name="read-external-files-with-external-table"></a>Lire des fichiers externes avec EXTERNAL TABLE

EXTERNAL TABLE vous permet de lire des données à partir des fichiers référencés via une source de données à l’aide d’une instruction SQL SELECT standard :

```sql
SELECT *
FROM dbo.DimProductsExternal
```

L’appelant doit disposer des autorisations suivantes pour lire les données :
- Autorisation `SELECT` sur la table externe
- Autorisation `REFERENCES DATABASE SCOPED CREDENTIAL` si `DATA SOURCE` a `CREDENTIAL`

## <a name="permissions"></a>Autorisations

Le tableau suivant présente les autorisations nécessaires pour les opérations listées ci-dessus.

| Requête | Autorisations requises|
| --- | --- |
| OPENROWSET(BULK) sans source de données | `ADMINISTER BULK OPERATIONS`, `ADMINISTER DATABASE BULK OPERATIONS`ou la connexion SQL doit avoir REFERENCES CREDENTIAL::\<URL> pour le stockage protégé par SAS |
| OPENROWSET(BULK) avec source de données sans informations d’identification | `ADMINISTER BULK OPERATIONS` ou `ADMINISTER DATABASE BULK OPERATIONS` |
| OPENROWSET(BULK) avec source de données avec informations d’identification | `REFERENCES DATABASE SCOPED CREDENTIAL` et soit `ADMINISTER BULK OPERATIONS` soit `ADMINISTER DATABASE BULK OPERATIONS` |
| CREATE EXTERNAL DATA SOURCE | `ALTER ANY EXTERNAL DATA SOURCE` et `REFERENCES DATABASE SCOPED CREDENTIAL` |
| CREATE EXTERNAL TABLE | `CREATE TABLE`, `ALTER ANY SCHEMA`, `ALTER ANY EXTERNAL FILE FORMAT` et `ALTER ANY EXTERNAL DATA SOURCE` |
| SELECT FROM EXTERNAL TABLE | `SELECT TABLE` et `REFERENCES DATABASE SCOPED CREDENTIAL` |
| CETAS | Pour créer une table : `CREATE TABLE`, `ALTER ANY SCHEMA`, `ALTER ANY DATA SOURCE` et `ALTER ANY EXTERNAL FILE FORMAT`. Pour lire les données : `ADMINISTER BULK OPERATIONS` ou `REFERENCES CREDENTIAL` ou `SELECT TABLE` pour chaque table/vue/fonction dans la requête + autorisation de lecture/écriture sur le stockage |

## <a name="next-steps"></a>Étapes suivantes

Vous êtes maintenant prêt à poursuivre avec les articles de guide pratique suivants :

- [Interroger des données sur le stockage](query-data-storage.md)

- [Interroger un fichier CSV](query-single-csv-file.md)

- [Interroger des fichiers Parquet](query-parquet-files.md)

- [Interroger des fichiers JSON](query-json-files.md)

- [Interroger des dossiers et plusieurs fichiers](query-folders-multiple-csv-files.md)

- [Utiliser les fonctions de partitionnement et de métadonnées](query-specific-files.md)

- [Interroger des types imbriqués](query-parquet-nested-types.md)

- [Création et utilisation de vues](create-use-views.md)
