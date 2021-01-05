---
title: Envoyer des données à un niveau de stockage d'objets blob chaud, froid ou archive via Azure Data Box/Azure Data Box Heavy
description: Décrit comment utiliser Azure Data Box ou Azure Data Box Heavy pour envoyer des données à un niveau de stockage d’objets blob de bloc approprié comme chaud, froid ou archive.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: how-to
ms.date: 05/24/2019
ms.author: alkohli
ms.openlocfilehash: 80c4d8a70454c007ac45f588e59c03ef45f10933
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92125062"
---
# <a name="use-azure-data-box-or-azure-data-box-heavy-to-send-data-to-appropriate-azure-storage-blob-tier"></a>Utiliser Azure Data Box ou Azure Data Box Heavy pour envoyer des données au niveau d’objets blob de Stockage Azure approprié

Azure Data Box déplace de grandes quantités de données vers Azure en vous fournissant un dispositif de stockage propriétaire. Vous remplissez le dispositif avec des données et vous le renvoyez. Les données de Data Box sont chargées sur un niveau par défaut associé au compte de stockage. Vous pouvez ensuite déplacer les données vers un autre niveau de stockage.

Cet article décrit comment déplacer les données qui sont chargées par Data Box vers un niveau d’objets blob chaud, froid ou archive. Cet article s’applique à Azure Data Box et à Azure Data Box Heavy.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="choose-the-correct-storage-tier-for-your-data"></a>Choisir le niveau de stockage approprié pour vos données

Le stockage Azure propose trois différents niveaux pour stocker les données de la manière la plus rentable : chaud, froid ou archive. Le niveau de stockage chaud est optimisé pour le stockage des données souvent sollicitées. Le stockage chaud possède des coûts de stockage plus élevés que les stockages froid et archive, mais également les coûts d’accès les plus faibles.

Le niveau de stockage froid concerne les données qui sont rarement sollicitées et qui doivent être stockées pour un minimum de 30 jours. Le coût de stockage du niveau froid est inférieur à celui du niveau de stockage chaud, mais les frais d’accès aux données sont plus élevés.

Le niveau archive Azure est hors connexion et offre les coûts de stockage les plus bas, mais également les coûts d’accès les plus élevés. Ce niveau est destiné aux données qui restent dans le stockage d’archivage pour un minimum de 180 jours. Pour plus d’informations de chacun de ces niveaux et sur le modèle de tarification, accédez à [Comparaison des niveaux de stockage](../storage/blobs/storage-blob-storage-tiers.md).

Les données de Data Box ou Data Box Heavy sont chargées sur un niveau de stockage associé au compte de stockage. Quand vous créez un compte de stockage, vous pouvez spécifier Froid ou Chaud comme niveau d’accès. En fonction du modèle d’accès de votre charge de travail et du coût, vous pouvez déplacer ces données du niveau par défaut vers un autre niveau de stockage.

Vous ne pouvez hiérarchiser vos données de stockage d’objets que dans des comptes de stockage d’objets blob ou v2 à usage général (GPv2). Les comptes Usage général v1 (GPv1) ne prennent pas en charge la hiérarchisation. Pour choisir le niveau de stockage approprié pour vos données, passez en revue les considérations détaillées dans [Stockage Blob Azure : niveaux de stockage Premium, chaud, froid et archive](../storage/blobs/storage-blob-storage-tiers.md).

## <a name="set-a-default-blob-tier"></a>Définir un niveau d’objets blob par défaut

Le niveau d’objets blob par défaut est spécifié lors de la création du compte de stockage dans le portail Azure. Une fois que vous avez sélectionné GPv2 ou Stockage Blob comme type de stockage, vous pouvez spécifier l’attribut de niveau d’accès. Par défaut, le niveau chaud est sélectionné.

Les niveaux ne peuvent pas être spécifiés si vous essayez de créer un compte lors de la commande d’une Data Box ou d’une Data Box Heavy. Une fois le compte créé, vous pouvez le modifier dans le portail pour définir le niveau d’accès par défaut.

Vous pouvez aussi créer d’abord un compte de stockage avec l’attribut de niveau d’accès spécifié. Lors de la création de la commande Data Box ou Data Box Heavy, sélectionnez le compte de stockage existant. Pour plus d’informations sur la façon de définir le niveau d’objets blob par défaut lors de la création d’un compte de stockage, accédez à [Créer un compte de stockage dans le portail Azure](../storage/common/storage-account-create.md?tabs=portal).

## <a name="move-data-to-a-non-default-tier"></a>Déplacer des données vers un niveau autre que le niveau par défaut

Une fois les données d’un appareil Data Box chargées sur le niveau par défaut, vous souhaiterez peut-être les déplacer vers un autre niveau. Il existe deux façons de déplacer ces données vers un niveau autre que le niveau par défaut.

- **Gestion du cycle de vie du Stockage Blob Azure** : vous pouvez utiliser une approche basée sur la stratégie pour hiérarchiser automatiquement les données ou les faire expirer à la fin de leur cycle de vie. Pour plus d’informations, consultez [Gestion du cycle de vie de Stockage Blob Azure](../storage/blobs/storage-lifecycle-management-concepts.md).
- **Écriture de scripts** : vous pouvez utiliser des scripts Azure PowerShell pour activer la hiérarchisation des objets blob. Vous pouvez appeler l’opération `SetBlobTier` afin de définir le niveau de l’objet blob.

## <a name="use-azure-powershell-to-set-the-blob-tier"></a>Utiliser Azure PowerShell pour définir le niveau des objets blob

Les étapes suivantes décrivent comment affecter les objets blob au niveau Archive à l’aide d’un script Azure PowerShell.

1. Ouvrez une session Windows PowerShell avec des privilèges élevés. Vérifiez que vous exécutez PowerShell 5.0 ou version ultérieure. Tapez :

   `$PSVersionTable.PSVersion`     

2. Connectez-vous à Azure PowerShell. 

   `Login-AzAccount`  

3. Définissez les variables de compte de stockage, de clé d’accès, de conteneur et de contexte de stockage.

    ```powershell
    $StorageAccountName = "<enter account name>"
    $StorageAccountKey = "<enter account key>"
    $ContainerName = "<enter container name>"
    $ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey
    ```

4. Obtenez tous les objets blob du conteneur.

    `$blobs = Get-AzStorageBlob -Container "<enter container name>" -Context $ctx`
 
5. Définissez le niveau de tous les objets blob dans le conteneur sur « Archive ».

    ```powershell
    Foreach ($blob in $blobs) {
    $blob.ICloudBlob.SetStandardBlobTier("Archive")
    }
    ```

    Voici un exemple de sortie obtenue :

    ```
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.
    PS C:\WINDOWS\system32> $PSVersionTable.PSVersion

    Major  Minor  Build  Revision
    -----  -----  -----  --------
    5      1      17763  134
    PS C:\WINDOWS\system32> Login-AzAccount

    Account          : gus@contoso.com
    SubscriptionName : MySubscription
    SubscriptionId   : subscription-id
    TenantId         : tenant-id
    Environment      : AzureCloud

    PS C:\WINDOWS\system32> $StorageAccountName = "mygpv2storacct"
    PS C:\WINDOWS\system32> $StorageAccountKey = "mystorageacctkey"
    PS C:\WINDOWS\system32> $ContainerName = "test"
    PS C:\WINDOWS\system32> $ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey
    PS C:\WINDOWS\system32> $blobs = Get-AzStorageBlob -Container "test" -Context $ctx
    PS C:\WINDOWS\system32> Foreach ($blob in $blobs) {
    >> $blob.ICloudBlob.SetStandardBlobTier("Archive")
    >> }
    PS C:\WINDOWS\system32>
    ```
   > [!TIP]
   > Si vous souhaitez que les données soient archivées au moment de l’ingestion, définissez le niveau par défaut du compte sur Chaud. Si le niveau par défaut est Froid, il existe une pénalité de suppression anticipée de 30 jours si les données sont déplacées immédiatement vers le niveau Archive.

## <a name="next-steps"></a>Étapes suivantes

-  Découvrez comment gérer les [scénarios courants de hiérarchisation des données avec des règles de stratégie de cycle de vie](../storage/blobs/storage-lifecycle-management-concepts.md#examples).