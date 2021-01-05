---
title: Développer le lecteur de système d’exploitation d’une machine virtuelle Windows dans Azure
description: Développer la taille du lecteur de système d’exploitation d’une machine virtuelle à l’aide d’Azure PowerShell dans le modèle de déploiement Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: kirpasingh
manager: roshar
editor: ''
tags: azure-resource-manager
ms.assetid: d9edfd9f-482f-4c0b-956c-0d2c2c30026c
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 09/02/2020
ms.author: kirpas
ms.subservice: disks
ms.openlocfilehash: df27d7b25010fa68fc86ffe093318b2b0b7f4e96
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93393827"
---
# <a name="how-to-expand-the-os-drive-of-a-virtual-machine"></a>Extension du lecteur de système d’exploitation d’une machine virtuelle

Lorsque vous créez une machine virtuelle (VM) dans un groupe de ressources en déployant une image à partir de la [Place de marché Azure](https://azure.microsoft.com/marketplace/), le lecteur du système d’exploitation par défaut a une capacité de 127 Go (par défaut, certaines images peuvent avoir des disques de système d’exploitation de plus petite taille). Même s’il est possible d’ajouter des disques de données à la machine virtuelle (le nombre dépend de la référence (SKU) choisie), et si nous recommandons d’installer des applications et des charges de travail nécessitant une utilisation importante du processeur sur ces disques supplémentaires, il peut arriver que les clients doivent développer le lecteur du système d’exploitation pour prendre en charge des scénarios spécifiques :

- Prendre en charge des applications héritées qui installent des composants sur le lecteur du système d’exploitation.
- Migrer un ordinateur physique ou une machine virtuelle à partir d’un emplacement local avec un lecteur de système d’exploitation plus volumineux.

> [!IMPORTANT]
> Le redimensionnement d’un disque de système d’exploitation ou de données d’une machine virtuelle Azure exige une désallocation (ou libération) de la machine virtuelle.
>
> La réduction d’un disque existant n’est pas prise en charge. Elle risque d’entraîner une perte de données.
> 
> Après avoir étendu les disques, vous devez [étendre le volume dans le système d’exploitation](#expand-the-volume-within-the-os) pour tirer parti du disque plus volumineux.

## <a name="resize-a-managed-disk-in-the-azure-portal"></a>Redimensionner un disque managé dans le portail Azure

1. Dans le [portail Azure](https://portal.azure.com), accédez à la machine virtuelle dans laquelle vous souhaitez étendre le disque. Sélectionnez **Arrêter** pour désallouer\libérer la machine virtuelle.
2. Une fois la machine virtuelle arrêtée, dans le menu de gauche, sous **Paramètres** , sélectionnez **Disques**.

    :::image type="content" source="./media/expand-os-disk/select-disks.png" alt-text="Capture d’écran montrant l’option Disques sélectionnée dans la section Paramètres du menu.":::

 
3. Sous **Nom du disque** , sélectionnez le disque que vous souhaitez redimensionner.

    :::image type="content" source="./media/expand-os-disk/disk-name.png" alt-text="Capture d’écran montrant le volet Disques avec un nom de disque sélectionné.":::

4. Dans le menu de gauche, sous **Paramètres** , sélectionnez **Configuration**.

    :::image type="content" source="./media/expand-os-disk/configuration.png" alt-text="Capture d’écran montrant l’option Configuration sélectionnée dans la section Paramètres du menu.":::

5. Dans **Taille (Gio)** , sélectionnez la taille de disque souhaitée.
   
   > [!WARNING]
   > La nouvelle taille doit être supérieure à la taille du disque actuelle. Le maximum autorisé est de 2 048 Go pour les disques du système d’exploitation (il est possible d’étendre la taille de l’objet blob du disque dur virtuel, mais le système d’exploitation ne peut utiliser que les premiers 2 048 Go de l’espace).
   > 

    :::image type="content" source="./media/expand-os-disk/size.png" alt-text="Capture d’écran montrant le volet Configuration avec la taille de disque sélectionnée.":::

6. Sélectionnez **Enregistrer**.

    :::image type="content" source="./media/expand-os-disk/save.png" alt-text="Capture d’écran montrant le volet Configuration avec le bouton Enregistrer sélectionné.":::


## <a name="resize-a-managed-disk-by-using-powershell"></a>Redimensionner un disque managé à l’aide de PowerShell

Ouvrez votre PowerShell ISE ou une fenêtre PowerShell en mode administrateur, puis procédez de la manière décrite ci-dessous :

1. Connectez-vous à votre compte Microsoft Azure en mode de gestion des ressources et sélectionnez votre abonnement :
   
    ```powershell
    Connect-AzAccount
    Select-AzSubscription –SubscriptionName 'my-subscription-name'
    ```

2. Définissez le nom de votre groupe de ressources et le nom de la machine virtuelle :
   
    ```powershell
    $rgName = 'my-resource-group-name'
    $vmName = 'my-vm-name'
    ```

3. Obtenez une référence à votre machine virtuelle :
   
    ```powershell
    $vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

4. Arrêtez la machine virtuelle avant de redimensionner le disque :
   
    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

5. Obtenez une référence vers le disque de système d’exploitation managé. Définissez la taille du disque du système d’exploitation managé sur la valeur souhaitée et mettez à jour le disque :
   
    ```powershell
    $disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
    $disk.DiskSizeGB = 1023
    Update-AzDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
    ```   
    > [!WARNING]
    > La nouvelle taille doit être supérieure à la taille du disque actuelle. Le maximum autorisé est de 2 048 Go pour les disques du système d’exploitation (il est possible d’étendre la taille de l’objet blob du disque dur virtuel, mais le système d’exploitation ne peut utiliser que les premiers 2 048 Go de l’espace).
    > 
         
6. La mise à jour de la machine virtuelle peut prendre quelques secondes. Une fois l’exécution de la commande terminée, redémarrez la machine virtuelle :
   
    ```powershell
    Start-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

Vous avez terminé. Connectez-vous via RDP à la machine virtuelle, ouvrez Gestion de l’ordinateur (ou Gestion des disques) et développez le lecteur à l’aide de l’espace qui vient d’être alloué.

## <a name="resize-an-unmanaged-disk-by-using-powershell"></a>Redimensionner un disque non managé à l’aide de PowerShell

Ouvrez votre PowerShell ISE ou une fenêtre PowerShell en mode administrateur, puis procédez de la manière décrite ci-dessous :

1. Connectez-vous à votre compte Microsoft Azure en mode de gestion des ressources et sélectionnez votre abonnement :
   
    ```powershell
    Connect-AzAccount
    Select-AzSubscription –SubscriptionName 'my-subscription-name'
    ```

2. Définissez le nom de votre groupe de ressources et les noms de machine virtuelle :
   
    ```powershell
    $rgName = 'my-resource-group-name'
    $vmName = 'my-vm-name'
    ```

3. Obtenez une référence à votre machine virtuelle :
   
    ```powershell
    $vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

4. Arrêtez la machine virtuelle avant de redimensionner le disque :
   
    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

5. Définissez la taille du disque du système d’exploitation non managé sur la valeur souhaitée et mettez à jour la machine virtuelle :
   
    ```powershell
    $vm.StorageProfile.OSDisk.DiskSizeGB = 1023
    Update-AzVM -ResourceGroupName $rgName -VM $vm
    ```
   
    > [!WARNING]
    > La nouvelle taille doit être supérieure à la taille du disque actuelle. Le maximum autorisé est de 2 048 Go pour les disques du système d’exploitation (Il est possible d’étendre la taille de l’objet blob du disque virtuel, mais le système d’exploitation ne peut utiliser que l’espace des premiers 2,048 Go).
    > 
    > 
   
6. La mise à jour de la machine virtuelle peut prendre quelques secondes. Une fois l’exécution de la commande terminée, redémarrez la machine virtuelle :
   
    ```powershell
    Start-AzVM -ResourceGroupName $rgName -Name $vmName
    ```


## <a name="scripts-for-os-disk"></a>Scripts pour le disque du système d’exploitation

Voici le script complet de référence pour les disques managés et non managés :


**Disques managés**

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
Stop-AzVM -ResourceGroupName $rgName -Name $vmName
$disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
$disk.DiskSizeGB = 1023
Update-AzDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

**Disques non managés**

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
Stop-AzVM -ResourceGroupName $rgName -Name $vmName
$vm.StorageProfile.OSDisk.DiskSizeGB = 1023
Update-AzVM -ResourceGroupName $rgName -VM $vm
Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

## <a name="resizing-data-disks"></a>Redimensionnement des disques de données

Cet article se concentre principalement sur l’extension du disque du système d’exploitation de la machine virtuelle, mais le script peut également servir à étendre les disques de données associés à la machine virtuelle. Par exemple, pour développer le premier disque de données associé à la machine virtuelle, remplacez l’objet `OSDisk` de `StorageProfile` par le tableau `DataDisks` et utilisez un index numérique pour obtenir une référence au premier disque de données associé, comme indiqué ci-dessous :

**Disque managé**

```powershell
$disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.DataDisks[0].Name
$disk.DiskSizeGB = 1023
```

**Disque non managé**

```powershell
$vm.StorageProfile.DataDisks[0].DiskSizeGB = 1023
```

Vous pouvez également référencer d’autres disques de données associés à la machine virtuelle en utilisant un index comme indiqué ci-dessus ou en utilisant la propriété **Nom** du disque :


**Disque managé**

```powershell
(Get-AzDisk -ResourceGroupName $rgName -DiskName ($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'})).Name).DiskSizeGB = 1023
```

**Disque non managé**

```powershell
($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'})).DiskSizeGB = 1023
```

## <a name="expand-the-volume-within-the-os"></a>Étendre le volume dans le système d’exploitation

Une fois que vous avez étendu le disque pour la machine virtuelle, vous devez accéder au système d’exploitation et étendre le volume pour qu’il englobe le nouvel espace. Il existe plusieurs méthodes pour étendre une partition. Cette section décrit la connexion de la machine virtuelle à l’aide d’une connexion RDP pour étendre la partition à l’aide de **DiskPart**.

1. Ouvrez une connexion RDP à votre machine virtuelle.

2. Ouvrez une invite de commandes et tapez **diskpart**.

3. À l’invite **DISKPART** , tapez `list volume`. Prenez note du volume que vous souhaitez étendre.

4. À l’invite **DISKPART** , tapez `select volume <volumenumber>`. Cette commande sélectionne le volume *volumenumber* que vous souhaitez étendre dans un espace vide contigu sur le même disque.

5. À l’invite **DISKPART** , tapez `extend [size=<size>]`. Cette commande étend le volume sélectionné d’après la taille ( *size* ) en mégaoctets (Mo) indiquée.


## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également attacher des disques à l’aide du [portail Azure](attach-managed-disk-portal.md).
