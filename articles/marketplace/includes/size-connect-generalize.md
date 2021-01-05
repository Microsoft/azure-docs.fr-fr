---
title: Fichier include
description: fichier
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: include
author: mingshen-ms
ms.author: krsh
ms.date: 10/20/2020
ms.openlocfilehash: addc18a0ebf9e49d3474d3f40cb1e2a6e0f0b272
ms.sourcegitcommit: 28c93f364c51774e8fbde9afb5aa62f1299e649e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97826619"
---
## <a name="generalize-the-image"></a>Généraliser l’image

Toutes les images dans Azure Marketplace doivent être réutilisables de façon générale. Pour autoriser cela, le VHD du système d’exploitation doit être généralisé. Cette opération consiste à supprimer d’une machine virtuelle tous les pilotes logiciels et identificateurs propres à une instance.

### <a name="for-windows"></a>Pour Windows

Les disques de système d’exploitation Windows sont généralisés à l’aide de l’outil [Sysprep](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview). Si vous mettez à jour ou reconfigurez le système d’exploitation par la suite, vous devrez réexécuter Sysprep.

> [!WARNING]
> Après l’exécution de Sysprep, éteignez la machine virtuelle jusqu’à son déploiement, car les mises à jour peuvent s’exécuter automatiquement. Cet arrêt évite que des mises à jour ultérieures apportent des modifications propres à une instance au système d’exploitation ou aux services installés. Pour plus d’informations sur l’exécution de sysprep, consultez [Étapes de généralisation d’un disque dur virtuel](../../virtual-machines/windows/capture-image-resource.md#generalize-the-windows-vm-using-sysprep).

### <a name="for-linux"></a>Pour Linux

Le processus suivant généralise une machine virtuelle Linux et la redéploie sous la forme d’une machine virtuelle distincte. Pour plus d’informations, consultez [Guide pratique pour créer une image de machine virtuelle ou de disque dur virtuel](../../virtual-machines/linux/capture-image.md). Vous pouvez arrêter quand vous atteignez la section intitulée « Créer une machine virtuelle à partir de l’image capturée ».

1. Supprimez l’agent Linux Azure.
    1. Connectez-vous à votre machine virtuelle Linux en utilisant un client SSH.
    2. Dans la fenêtre SSH, saisissez la commande suivante : `sudo waagent –deprovision+user`.
    3. Tapez Y pour continuer (vous pouvez ajouter le paramètre -force à la commande précédente pour éviter l’étape de confirmation).
    4. Une fois la commande exécutée, entrez **Exit** pour fermer le client SSH.
2. Arrêtez la machine virtuelle.
    1. Dans le portail Azure, sélectionnez votre groupe de ressources et désallouez la machine virtuelle.
    2. Votre machine virtuelle est désormais généralisée, et vous pouvez créer une autre machine virtuelle à l’aide de ce disque de machine virtuelle.

### <a name="take-a-snapshot-of-the-vm-disk"></a>Effectuer une capture instantanée du disque de la machine virtuelle

1. Connectez-vous au [portail Azure](https://ms.portal.azure.com/).
2. À partir du coin supérieur gauche, sélectionnez **Créer une ressource**, puis recherchez et sélectionnez **Instantané**.
3. Dans le panneau Instantané, sélectionnez **Créer**.
4. Entrez un **nom** pour la capture instantanée.
5. Sélectionnez un groupe de ressources existant ou entrez le nom d’un nouveau.
6. Dans **Disque source**, sélectionnez le disque managé dont vous souhaitez obtenir une capture instantanée.
7. Sélectionnez le **type de compte** à utiliser pour stocker la capture instantanée. Utilisez **Standard HDD**, sauf si vous en avez besoin de la stocker sur un disque SSD hautes performances.
8. Sélectionnez **Create** (Créer).

#### <a name="extract-the-vhd"></a>Extraire le disque dur virtuel

Utilisez le script suivant pour exporter l’instantané dans un disque dur virtuel de votre compte de stockage.

```azurecli-interactive
#Provide the subscription Id where the snapshot is created
$subscriptionId=yourSubscriptionId

#Provide the name of your resource group where the snapshot is created
$resourceGroupName=myResourceGroupName

#Provide the snapshot name
$snapshotName=mySnapshot

#Provide Shared Access Signature (SAS) expiry duration in seconds (such as 3600)
#Know more about SAS here: https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-shared-access-signature-part-1
$sasExpiryDuration=3600

#Provide storage account name where you want to copy the underlying VHD file. 
$storageAccountName=mystorageaccountname

#Name of the storage container where the downloaded VHD will be stored.
$storageContainerName=mystoragecontainername

#Provide the key of the storage account where you want to copy the VHD 
$storageAccountKey=mystorageaccountkey

#Give a name to the destination VHD file to which the VHD will be copied.
$destinationVHDFileName=myvhdfilename.vhd

az account set --subscription $subscriptionId

sas=$(az snapshot grant-access --resource-group $resourceGroupName --name $snapshotName --duration-in-seconds $sasExpiryDuration --query [accessSas] -o tsv)

az storage blob copy start --destination-blob $destinationVHDFileName --destination-container $storageContainerName --account-name $storageAccountName --account-key $storageAccountKey --source-uri $sas
```

#### <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes afin de générer l’URI SAS pour un instantané et copie le VHD sous-jacent vers un compte de stockage à l’aide de l’URI SAS. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
| --- | --- |
| az disk grant-access | Génère l’URI SAP en lecture seule qui est utilisé pour copier le fichier de VHD sous-jacent vers un compte de stockage ou le télécharger en local
| az storage blob copy start | Copiez un objet blob de façon asynchrone à partir d’un compte de stockage vers un autre. Utilisez `az storage blob show` pour vérifier l’état du nouvel objet Blob. |
|
