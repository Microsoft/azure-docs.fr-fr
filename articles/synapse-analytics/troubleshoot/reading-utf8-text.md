---
title: 'Résoudre les problèmes : Lecture de texte UTF-8 à partir de fichiers CSV ou PARQUET à l’aide d’un pool SQL serverless'
description: Lecture de texte UTF-8 à partir de fichiers CSV ou PARQUET à l’aide d’un pool SQL serverless dans Azure Synapse Analytics
author: julieMSFT
ms.author: jrasnick
ms.topic: troubleshooting
ms.service: synapse-analytics
ms.subservice: sql
ms.date: 12/03/2020
ms.openlocfilehash: 70ce3c82790db0296d5359b5db2e6a323306c309
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576415"
---
# <a name="troubleshoot-reading-utf-8-text-from-csv-or-parquet-files-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Résolution des problèmes de lecture de texte UTF-8 à partir de fichiers CSV ou Parquet à l’aide d’un pool SQL serverless dans Azure Synapse Analytics

Cet article décrit les étapes de résolution des problèmes de lecture de texte UTF-8 à partir de fichiers CSV ou Parquet à l’aide d’un pool SQL serverless dans Azure Synapse Analytics.

Lorsque le texte UTF-8 est lu à partir d’un fichier CSV ou PARQUET à l’aide d’un pool SQL serverless, certains caractères spéciaux, tels que ü et ö, sont convertis de manière incorrecte si la requête retourne des colonnes VARCHAR avec des classements non-UTF8. Il s’agit d’un problème connu dans SQL Server et SQL Azure. Le classement non-UTF8 étant l’option par défaut dans Synapse SQL, les requêtes des clients seront affectées. Il se peut que des clients utilisant des caractères anglais standard et certains sous-ensembles de caractères latins étendus ne remarquent pas les erreurs de conversion. La conversion incorrecte est expliquée plus en détail dans l’article [Always use UTF-8 collations to read UTF-8 text in serverless SQL pool](https://techcommunity.microsoft.com/t5/azure-synapse-analytics/always-use-utf-8-collations-to-read-utf-8-text-in-serverless-sql/ba-p/1883633) (Toujours utiliser les classements UTF-8 pour lire du texte UTF-8 dans un pool SQL serverless)

## <a name="workaround"></a>Solution de contournement

La solution à ce problème consiste à toujours utiliser le classement UTF-8 lors de la lecture de texte UTF-8 à partir de fichiers CSV ou PARQUET.

- Dans de nombreux cas, vous devez simplement définir le classement UTF8 sur la base de données (opération sur les métadonnées).

   ```sql
   alter database MyDB
         COLLATE Latin1_General_100_BIN2_UTF8;
   ```

- Vous pouvez définir explicitement le classement sur une colonne VARCHAR dans OPENROWSET ou dans une table externe :

   ```sql
   select geo_id, cases = sum(cases)
   from openrowset(
           bulk 'latest/ecdc_cases.parquet', data_source = 'covid', format = 'parquet'
       ) with ( cases int,
                geo_id VARCHAR(6) COLLATE Latin1_General_100_BIN2_UTF8 ) as rows
   group by geo_id
   ```
 
- Si vous n’avez pas spécifié le classement UTF8 sur les tables externes qui lisent les données UTF8, vous devez recréer les tables externes affectées et définir le classement UTF8 sur les colonnes VARCHAR (opération sur les métadonnées).


## <a name="next-steps"></a>Étapes suivantes

* [Interroger des fichiers Parquet avec Synapse SQL](../sql/query-parquet-files.md)
* [Interroger des fichiers CSV avec Synapse SQL](../sql/query-single-csv-file.md)
* [CETAS avec Synapse SQL](../sql/develop-tables-cetas.md)
* [Démarrage rapide : Utiliser un pool SQL serverless](../quickstart-sql-on-demand.md)
