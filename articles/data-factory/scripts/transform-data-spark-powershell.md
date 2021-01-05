---
title: Transformer des données dans le cloud à l’aide de PowerShell
description: Ce script PowerShell transforme les données dans le cloud en exécutant le programme Spark sur un cluster Azure HDInsight Spark.
author: dcstwh
ms.author: weetok
manager: anandsub
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/12/2017
ms.openlocfilehash: 9fe0efcb14c1da84ca5284d55366b6c7b79727b1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96497008"
---
# <a name="powershell-script---transform-data-in-cloud-using-azure-data-factory"></a>Script PowerShell - Transformer des données dans le cloud à l’aide d’Azure Data Factory

Cet exemple de script PowerShell crée un pipeline qui transforme les données dans le cloud en exécutant le programme Spark sur un cluster Azure HDInsight Spark. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Conditions préalables requises
* **Compte Stockage Azure**. Créez un script Python et un fichier d’entrée, puis téléchargez-les sur le stockage Azure. La sortie du programme Spark est stockée dans ce compte de stockage. Le cluster Spark sur demande utilise le même compte de stockage comme stockage principal.  

### <a name="upload-python-script-to-your-blob-storage-account"></a>Télécharger un script Python dans votre compte de stockage d’objets Blob
1. Créez un fichier Python nommé **WordCount_Spark.py** avec le contenu suivant : 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
2. Remplacez **&lt;storageAccountName&gt;** par le nom de votre compte de stockage Azure. Puis enregistrez le fichier. 
3. Dans votre stockage Blob Azure, créez un conteneur nommé **adftutorial** s’il n’existe pas. 
4. Créez un dossier nommé **spark**.
5. Créer un sous-dossier nommé **script** sous le dossier **spark**. 
6. Téléchargez le fichier **WordCount_Spark.py** dans le sous-dossier **script**. 


### <a name="upload-the-input-file"></a>Télécharger le fichier d’entrée
1. Créez un fichier nommé **minecraftstory.txt** avec du texte. Le programme Spark compte le nombre de mots dans ce texte. 
2. Créez un sous-dossier nommé `inputfiles` dans le dossier `spark` du conteneur blob. 
3. Téléchargez le fichier `minecraftstory.txt` dans le sous-dossier `inputfiles`. 

## <a name="sample-script"></a>Exemple de script
> [!IMPORTANT]
> Ce script crée des fichiers JSON qui définissent des entités Data Factory (service lié, jeu de données et pipeline) sur votre disque dur dans le dossier c:\.

[!code-powershell[main](../../../powershell_scripts/data-factory/transform-data-using-spark/transform-data-using-spark.ps1 "Transform data using Spark")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Après avoir exécuté l’exemple de script, vous pouvez utiliser la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées :

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```
Pour supprimer la fabrique de données du groupe de ressources, exécutez la commande suivante : 

```powershell
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes :

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [Set-AzDataFactoryV2](/powershell/module/az.datafactory/set-Azdatafactoryv2) | Créer une fabrique de données. |
| [Set-AzDataFactoryV2LinkedService](/powershell/module/az.datafactory/set-Azdatafactoryv2linkedservice) | Crée un service lié dans la fabrique de données. Un service lié rattache une banque de données ou une ressource de calcul à une fabrique de données. |
| [Set-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/set-Azdatafactoryv2pipeline) | Crée un pipeline dans la fabrique de données. Un pipeline contient une ou plusieurs activités effectuant une opération donnée. Dans ce pipeline, une activité spark transforme les données en exécutant un programme sur un cluster Azure HDInsight Spark. |
| [Invoke-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/invoke-Azdatafactoryv2pipeline) | Crée une exécution pour le pipeline. En d’autres termes, exécute le pipeline. |
| [Get-AzDataFactoryV2ActivityRun](/powershell/module/az.datafactory/get-Azdatafactoryv2activityrun) | Obtient les détails de l’exécution de l’activité dans le pipeline. 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |
|||

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure PowerShell, consultez la [documentation Azure PowerShell](/powershell/).

Des exemples supplémentaires de scripts PowerShell pour Azure Data Factory sont à votre disposition dans [Exemples PowerShell pour Azure Data Factory](../samples-powershell.md).