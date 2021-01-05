---
title: Bien démarrer avec Azure Data Lake Storage Gen1 - PowerShell | Microsoft Docs
description: Utilisez Azure PowerShell pour créer un compte Data Lake Storage Gen1 et effectuer des opérations de base.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 06/27/2018
ms.author: twooley
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 485b14f85d68290c5447c885b9bc4974318f7952
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103728"
---
# <a name="get-started-with-azure-data-lake-storage-gen1-using-azure-powershell"></a>Bien démarrer avec Azure Data Lake Storage Gen1 à l’aide de PowerShell

> [!div class="op_single_selector"]
> * [Portail](data-lake-store-get-started-portal.md)
> * [PowerShell](data-lake-store-get-started-powershell.md)
> * [Azure CLI](data-lake-store-get-started-cli-2.0.md)
>
>

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Découvrez comment utiliser Azure PowerShell pour créer un compte Azure Data Lake Storage Gen1 et effectuer des opérations de base comme créer des dossiers, charger et télécharger des fichiers de données, supprimer votre compte, etc. Pour plus d’informations sur Data Lake Store Gen1, consultez [Vue d’ensemble de Data Lake Storage Gen1](data-lake-store-overview.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1.0 ou version ultérieure**. Consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/).

## <a name="authentication"></a>Authentification

Pour l’authentification auprès de Data Lake Storage Gen1, cet article utilise une approche plus simple où vous êtes invité à entrer les informations d’identification de votre compte Azure. Le niveau d’accès au compte et au système de fichiers Data Lake Storage Gen1 est alors régi par le niveau d’accès de l’utilisateur connecté. Cependant, il existe d’autres approches pour l’authentification sur Data Lake Storage Gen1, à savoir l’authentification de l’utilisateur final ou l’authentification de service à service. Pour obtenir des instructions et plus d’informations sur l’authentification, consultez [l’authentification de l’utilisateur final](data-lake-store-end-user-authenticate-using-active-directory.md) ou [l’authentification de service à service](./data-lake-store-service-to-service-authenticate-using-active-directory.md).

## <a name="create-a-data-lake-storage-gen1-account"></a>Créer un compte Data Lake Storage Gen1

1. Sur votre Bureau, ouvrez une nouvelle fenêtre Windows PowerShell. Entrez l’extrait de code suivant pour vous connecter à votre compte Azure, définir l’abonnement et inscrire le fournisseur Data Lake Storage Gen1. À l’invite de connexion, vérifiez que vous vous connectez en tant qu’administrateur/propriétaire de l’abonnement :

    ```PowerShell
    # Log in to your Azure account
    Connect-AzAccount

    # List all the subscriptions associated to your account
    Get-AzSubscription

    # Select a subscription
    Set-AzContext -SubscriptionId <subscription ID>

    # Register for Azure Data Lake Storage Gen1
    Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
    ```

1. Un compte Data Lake Storage Gen1 est associé à un groupe de ressources Azure. Commencez par créer un groupe de ressources.

    ```PowerShell
    $resourceGroupName = "<your new resource group name>"
    New-AzResourceGroup -Name $resourceGroupName -Location "East US 2"
    ```

    ![Créer un groupe de ressources Azure](./media/data-lake-store-get-started-powershell/ADL.PS.CreateResourceGroup.png "Créer un groupe de ressources Azure")

1. Créer un compte Data Lake Storage Gen1. Le nom que vous spécifiez doit contenir uniquement des lettres minuscules et des chiffres.

    ```PowerShell
    $dataLakeStorageGen1Name = "<your new Data Lake Storage Gen1 account name>"
    New-AzDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStorageGen1Name -Location "East US 2"
    ```

    ![Créer un compte Data Lake Storage Gen1](./media/data-lake-store-get-started-powershell/ADL.PS.CreateADLAcc.png "Créer un compte Data Lake Storage Gen1")

1. Vérifiez que le compte a bien été créé.

    ```PowerShell
    Test-AzDataLakeStoreAccount -Name $dataLakeStorageGen1Name
    ```

    Le résultat de la cmdlet doit être **True**.

## <a name="create-directory-structures"></a>Créer des structures de répertoire

Vous pouvez créer des répertoires sous votre compte Data Lake Storage Gen1 pour gérer et stocker des données.

1. Spécifiez un répertoire racine.

    ```PowerShell
    $myrootdir = "/"
    ```

1. Créez un répertoire appelé **mynewdirectory** sous la racine spécifiée.

    ```PowerShell
    New-AzDataLakeStoreItem -Folder -AccountName $dataLakeStorageGen1Name -Path $myrootdir/mynewdirectory
    ```

1. Vérifiez que le répertoire a bien été créé.

    ```PowerShell
    Get-AzDataLakeStoreChildItem -AccountName $dataLakeStorageGen1Name -Path $myrootdir
    ```

    Ceci devrait afficher un résultat, comme illustré sur la capture d’écran suivante :

    ![Vérifier un répertoire](./media/data-lake-store-get-started-powershell/ADL.PS.Verify.Dir.Creation.png "Vérifier un répertoire")

## <a name="upload-data"></a>Télécharger des données

Vous pouvez charger des données sur Data Lake Storage Gen1 directement à la racine ou dans un répertoire que vous avez créé dans le compte. Les extraits de code de cette section montrent comment télécharger des exemples de données dans le répertoire (**mynewdirectory**) que vous avez créé dans la section précédente.

Si vous recherchez des exemples de données à charger, vous pouvez récupérer le dossier **Données Ambulance** dans le [Référentiel Git Azure Data Lake](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData). Téléchargez le fichier et stockez-le dans un répertoire local sur votre ordinateur, comme C:\sampledata\.

```PowerShell
Import-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
   -Path "C:\sampledata\vehicle1_09142014.csv" `
   -Destination $myrootdir\mynewdirectory\vehicle1_09142014.csv
```

## <a name="rename-download-and-delete-data"></a>Renommer, télécharger et supprimer des données

Utilisez la commande suivante pour renommer un fichier :

```PowerShell
Move-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Path $myrootdir\mynewdirectory\vehicle1_09142014.csv `
    -Destination $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv
```

Utilisez la commande suivante pour télécharger un fichier :

```PowerShell
Export-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Path $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv `
    -Destination "C:\sampledata\vehicle1_09142014_Copy.csv"
```

Utilisez la commande suivante pour supprimer un fichier :

```PowerShell
Remove-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Paths $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv
```

Quand vous y êtes invité, entrez **Y** pour supprimer l’élément. Si vous avez plusieurs fichiers à supprimer, vous pouvez fournir tous les chemins d'accès séparés par des virgules.

```PowerShell
Remove-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Paths $myrootdir\mynewdirectory\vehicle1_09142014.csv, $myrootdir\mynewdirectoryvehicle1_09142014_Copy.csv
```

## <a name="delete-your-account"></a>Supprimer votre compte

Utilisez la commande suivante pour supprimer votre compte Data Lake Storage Gen1.

```PowerShell
Remove-AzDataLakeStoreAccount -Name $dataLakeStorageGen1Name
```

Quand vous y êtes invité, entrez **Y** pour supprimer le compte.

## <a name="next-steps"></a>Étapes suivantes

* [Recommandations en matière d’optimisation des performances pour l’utilisation de PowerShell avec Azure Data Lake Storage Gen1](data-lake-store-performance-tuning-powershell.md)
* [Utiliser Azure Data Lake Storage Gen1 pour le Big Data](data-lake-store-data-scenarios.md)
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)