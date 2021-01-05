---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: 9447cec55c53861ca57d5416a91ffefd35fdd20b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91571931"
---
## <a name="repeatability-during-copy"></a>Répétabilité pendant la copie
Lors de la copie de données vers SQL Azure/SQL Server à partir d’autres magasins de données, vous devez garder la répétabilité à l’esprit afin d’éviter des résultats inattendus. 

Lors de la copie de données dans une base de données SQL Azure/SQL Server, l'activité de copie AJOUTE par défaut le jeu de données à la table de récepteur par défaut. Par exemple, lors de la copie de données à partir d’une source de fichier CSV (valeurs de données séparées par des virgules) contenant deux enregistrements pour la base de données SQL Azure/SQL Server, la table ressemble à ce qui suit :

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    2            2015-05-01 00:00:00
```

Supposons que vous trouviez des erreurs dans le fichier source et mettiez à jour la quantité de Down Tube de 2 à 4 dans le fichier source. Si vous réexécutez la tranche de données pour cette période, vous trouvez deux nouveaux enregistrements ajoutés à la base de données SQL Azure/SQL Server. L’exemple ci-dessous suppose qu’aucune des colonnes de la table ne dispose de contrainte de clé primaire.

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    2            2015-05-01 00:00:00
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    4            2015-05-01 00:00:00
```

Pour éviter cela, vous devez spécifier une sémantique UPSERT en tirant parti d’un des 2 mécanismes indiqués ci-dessous.

> [!NOTE]
> Une tranche peut être réexécutée automatiquement dans Azure Data Factory en fonction de la stratégie relative aux nouvelles tentatives qui a été spécifiée.
> 
> 

### <a name="mechanism-1"></a>Mécanisme 1
Vous pouvez tirer parti de la propriété **sqlWriterCleanupScript** pour effectuer d’abord une action de nettoyage lors de l’exécution d’une tranche. 

```json
"sink":  
{ 
  "type": "SqlSink", 
  "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM table WHERE ModifiedDate >= \\'{0:yyyy-MM-dd HH:mm}\\' AND ModifiedDate < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
}
```

Le script de nettoyage est d’abord exécuté pendant la copie d’une tranche donnée, ce qui supprime les données de la table SQL correspondant à cette tranche. L'activité insère ensuite les données dans la table SQL. 

Si la tranche est réexécutée à présent, la quantité est alors mise à jour comme vous le souhaitez.

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
Rien d’autre n’a dû être effectué. L’activité de copie a exécuté le script de nettoyage pour supprimer les données correspondant à cette tranche. Elle a ensuite lu l'entrée du fichier csv (qui ne contenait qu'un enregistrement) avant de l'insérer dans la table. 

### <a name="mechanism-2"></a>Mécanisme 2
> [!IMPORTANT]
> sliceIdentifierColumnName n’est pas pris en charge pour Azure Synapse Analytics actuellement. 

Un autre mécanisme pour obtenir la répétabilité consiste à disposer d’une colonne dédiée (**sliceIdentifierColumnName**) dans la table cible. Cette colonne peut être utilisée par Azure Data Factory pour s’assurer que la source et la destination restent synchronisées. Cette approche fonctionne s’il existe une flexibilité dans la modification ou la définition du schéma de table SQL de destination. 

Cette colonne est utilisée par Azure Data Factory à des fins de répétabilité, et au cours du processus, Azure Data Factory n’apporte pas de modifications de schéma à la table. Façon d’utiliser cette approche :

1. Définissez une colonne de type binaire (32) dans la table SQL de destination. Aucune contrainte ne doit exister sur cette colonne. Pour cet exemple, nommons cette colonne « ColumnForADFuseOnly ».
2. Utilisez-la dans l’activité de copie comme suit :
   
    ```json
    "sink":  
    { 
   
        "type": "SqlSink", 
        "sliceIdentifierColumnName": "ColumnForADFuseOnly"
    }
    ```

Azure Data Factory renseigne cette colonne conformément à ses besoins pour s’assurer que la source et la destination restent synchronisées. Les valeurs de cette colonne ne doivent pas être utilisées en dehors de ce contexte par l’utilisateur. 

Comme pour le mécanisme 1, l’activité de copie nettoie tout d’abord automatiquement les données de la tranche donnée dans la table SQL de destination, puis exécute l’activité de copie normalement pour insérer les données de la source dans la destination correspondant à cette tranche. 

