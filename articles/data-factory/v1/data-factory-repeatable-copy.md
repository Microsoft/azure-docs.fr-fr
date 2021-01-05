---
title: Copie répétable dans Azure Data Factory
description: Découvrez comment éviter les doublons, même si une tranche qui copie des données est exécutée plusieurs fois.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: cd0f389615c95ef9b9bc8280b6486740ddba4fb4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96456825"
---
# <a name="repeatable-copy-in-azure-data-factory"></a>Copie répétable dans Azure Data Factory

## <a name="repeatable-read-from-relational-sources"></a>Lecture renouvelée de sources relationnelles
Lorsque vous copiez des données à partir de magasins de données relationnels, gardez à l’esprit la répétabilité de l’opération, afin d’éviter des résultats imprévus. Dans Azure Data Factory, vous pouvez réexécuter une tranche manuellement. Vous pouvez également configurer une stratégie de nouvelles tentatives pour un jeu de données, afin qu’une tranche soit réexécutée en cas de défaillance. Lorsqu’une tranche est réexécutée d’une manière ou d’une autre, vous devez vous assurer que les mêmes données sont lues et ce, quel que soit le nombre d’exécutions de la tranche.  
 
> [!NOTE]
> Les exemples suivants concernent SQL Azure, mais sont applicables à tout autre magasin de données prenant en charge les jeux de données rectangulaires. Vous pouvez avoir besoin d’ajuster le **type** de source et la propriété de **requête** (par exemple : query au lieu de sqlReaderQuery) pour le magasin de données.   

En général, vous souhaitez lire uniquement les données des magasins relationnels qui correspondent à cette tranche. Pour cela, vous pouvez utiliser les variables système WindowStart et WindowEnd disponibles dans Azure Data Factory. Pour en savoir plus sur les variables et les fonctions dans Azure Data Factory, lisez l’article intitulé [Azure Data Factory - Variables système et fonctions](data-factory-functions-variables.md). Exemple : 

```json
"source": {
    "type": "SqlSource",
    "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm\\'', WindowStart, WindowEnd)"
},
```
Cette requête lit des données qui figurent dans la plage de durée de tranche (WindowStart -> WindowEnd) à partir de la table MyTable. Par ailleurs, la réexécution de cette tranche garantit toujours la lecture des mêmes données. 

Dans d’autres cas, vous souhaitez lire l’intégralité de la table. Vous pouvez alors définir sqlReaderQuery comme suit :

```json
"source": 
{            
    "type": "SqlSource",
    "sqlReaderQuery": "select * from MyTable"
},
```

## <a name="repeatable-write-to-sqlsink"></a>Écriture répétable dans SqlSink
Lors de la copie de données vers **SQL Azure/SQL Server** à partir d’autres magasins de données, vous devez garder la répétabilité à l’esprit afin d’éviter des résultats inattendus. 

Lors de la copie de données sur une base de données Azure SQL/SQL Server, l’activité de copie ajoute des données à la table de récepteur par défaut. Par exemple, vous copiez des données à partir d’un fichier CSV (valeurs séparées par des virgules) contenant deux enregistrements et les collez dans la table suivante d’une base de données Azure SQL/SQL Server. Lorsqu’une tranche est exécutée, les deux enregistrements sont copiés dans la table SQL. 

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    2            2015-05-01 00:00:00
```

Supposons que vous trouviez des erreurs dans le fichier source et mettiez à jour la quantité de Down Tube, la faisant passer de 2 à 4. Si vous réexécutez manuellement la tranche de données pour cette période, vous trouvez deux nouveaux enregistrements ajoutés à la base de données SQL Azure/SQL Server. Cet exemple suppose qu’aucune des colonnes de la table ne présente de contrainte de clé primaire.

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    2            2015-05-01 00:00:00
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    4            2015-05-01 00:00:00
```

Pour éviter ce problème, vous devez spécifier la sémantique UPSERT en utilisant l’une des deux mécanismes suivants :

### <a name="mechanism-1-using-sqlwritercleanupscript"></a>Mécanisme 1 : utilisation de la propriété sqlWriterCleanupScript
Vous pouvez utiliser la propriété **sqlWriterCleanupScript** pour nettoyer les données à partir de la table du récepteur avant d’insérer les données lors de l’exécution d’une tranche. 

```json
"sink":  
{ 
  "type": "SqlSink", 
  "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM table WHERE ModifiedDate >= \\'{0:yyyy-MM-dd HH:mm}\\' AND ModifiedDate < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
}
```

Lorsqu’une tranche s’exécute, le script de nettoyage est d’abord exécuté pour supprimer les données correspondant à la tranche dans la table SQL. Ensuite, l’activité de copie insère les données dans la table SQL. Si la tranche est réexécutée, la quantité est mise à jour comme vous le souhaitez.

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    4            2015-05-01 00:00:00
```

Supposons que l’enregistrement Flat Washer soit supprimé du fichier csv d’origine. Une nouvelle exécution de la tranche entraînerait le résultat suivant : 

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
7     Down Tube    4            2015-05-01 00:00:00
```

L’activité de copie a exécuté le script de nettoyage pour supprimer les données correspondant à cette tranche. Elle a ensuite lu l’entrée du fichier csv (qui ne contenait qu’un enregistrement) avant de l’insérer dans la table. 

### <a name="mechanism-2-using-sliceidentifiercolumnname"></a>Mécanisme 2 : utilisation du paramètre sliceIdentifierColumnName
> [!IMPORTANT]
> Actuellement, sliceIdentifierColumnName n’est pas pris en charge pour Azure Synapse Analytics. 

Un deuxième mécanisme pour obtenir la répétabilité consiste à disposer d’une colonne dédiée (sliceIdentifierColumnName) dans la table cible. Cette colonne peut être utilisée par Azure Data Factory pour s’assurer que la source et la destination restent synchronisées. Cette approche fonctionne s’il existe une flexibilité dans la modification ou la définition du schéma de table SQL de destination. 

Cette colonne est utilisée par Azure Data Factory à des fins de répétabilité. Au cours du processus, Azure Data Factory n’apporte aucune modification de schéma à la table. Façon d’utiliser cette approche :

1. Définissez une colonne de type **binaire (32)** dans la table SQL de destination. Aucune contrainte ne doit exister sur cette colonne. Pour les besoins de cet exemple, nommons cette colonne « AdfSliceIdentifier ».


    Table source :

    ```sql
    CREATE TABLE [dbo].[Student](
       [Id] [varchar](32) NOT NULL,
       [Name] [nvarchar](256) NOT NULL
    )
    ```

    Table de destination : 

    ```sql
    CREATE TABLE [dbo].[Student](
       [Id] [varchar](32) NOT NULL,
       [Name] [nvarchar](256) NOT NULL,
       [AdfSliceIdentifier] [binary](32) NULL
    )
    ```

1. Utilisez-la dans l’activité de copie comme suit :
   
    ```json
    "sink":  
    { 
   
        "type": "SqlSink", 
        "sliceIdentifierColumnName": "AdfSliceIdentifier"
    }
    ```

Azure Data Factory renseigne cette colonne conformément à ses besoins pour s’assurer que la source et la destination restent synchronisées. Les valeurs de cette colonne ne doivent pas être utilisées en dehors de ce contexte. 

Comme pour le mécanisme 1, l’activité de copie nettoie automatiquement les données de la tranche spécifiée dans la table SQL de destination. Ensuite, elle insère les données de la source dans la table de destination. 

## <a name="next-steps"></a>Étapes suivantes
Pour accéder à des exemples JSON complets, consultez les articles suivants sur les connecteurs : 

- [Azure SQL Database](data-factory-azure-sql-connector.md)
- [Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md)
- [SQL Server](data-factory-sqlserver-connector.md)
