---
title: Copier des données dans le cloud à l’aide de PowerShell
description: Ce script PowerShell copie des données d’un emplacement dans un Stockage Blob Azure vers un autre dans le même Stockage Blob Azure.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.service: data-factory
ms.workload: data-services
ms.topic: article
ms.custom: seo-lt-2019
ms.date: 03/12/2020
ms.openlocfilehash: 9550556aa022a9211072fd4c5fb18acb4ee7882e
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637987"
---
# <a name="use-powershell-to-create-a-data-factory-pipeline-to-copy-data-in-the-cloud"></a>Utiliser PowerShell pour créer un pipeline de fabrique de données afin de copier des données dans le cloud

Cet exemple de script PowerShell crée dans Azure Data Factory un pipeline qui copie les données d’un emplacement vers un autre emplacement dans un stockage Blob Azure.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Prérequis
* **Compte Stockage Azure** . Vous utilisez le stockage Blob à la fois comme magasins de données **source** et **récepteur** . Si vous ne possédez pas de compte de stockage Azure, consultez [Créer un compte de stockage](../../storage/common/storage-account-create.md) pour en créer un. 
* Créez un **conteneur d’objets blob** dans le stockage Blob, créez un **dossier** d’entrée dans le conteneur et chargez des fichiers sur le dossier. Vous pouvez utiliser des outils tels que l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) pour vous connecter au stockage Blob Azure, créer un conteneur d’objets blob, charger le fichier d’entrée et vérifier le fichier de sortie.

## <a name="sample-script"></a>Exemple de script

> [!IMPORTANT]
> Ce script crée des fichiers JSON qui définissent des entités Data Factory (service lié, jeu de données et pipeline) sur votre disque dur dans le dossier c:\.

[!code-powershell[main](../../../powershell_scripts/data-factory/copy-from-azure-blob-to-blob/copy-from-azure-blob-to-blob.ps1 "Copy from Blob Storage -> Blob Storage")]


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
| [Set-AzDataFactoryV2LinkedService](/powershell/module/az.datafactory/Set-Azdatafactoryv2linkedservice) | Crée un service lié dans la fabrique de données. Un service lié rattache une banque de données ou une ressource de calcul à une fabrique de données. |
| [Set-AzDataFactoryV2Dataset](/powershell/module/az.datafactory/Set-Azdatafactoryv2dataset) | Crée un jeu de données dans la fabrique de données. Un jeu de données représente les entrées/sorties pour une activité dans un pipeline. |
| [Set-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/Set-Azdatafactoryv2pipeline) | Crée un pipeline dans la fabrique de données. Un pipeline contient une ou plusieurs activités effectuant une opération donnée. Dans ce pipeline, l’activité de copie a pour effet de copier les données d’un emplacement vers un autre dans le Stockage Blob Azure. |
| [Invoke-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/Invoke-Azdatafactoryv2pipeline) | Crée une exécution pour le pipeline. En d’autres termes, exécute le pipeline. |
| [Get-AzDataFactoryV2ActivityRun](/powershell/module/az.datafactory/get-Azdatafactoryv2activityrun) | Obtient les détails de l’exécution de l’activité dans le pipeline.
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |
|||

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure PowerShell, consultez la [documentation Azure PowerShell](/powershell/).

Des exemples supplémentaires de scripts PowerShell pour Azure Data Factory sont à votre disposition dans [Exemples PowerShell pour Azure Data Factory](../samples-powershell.md).