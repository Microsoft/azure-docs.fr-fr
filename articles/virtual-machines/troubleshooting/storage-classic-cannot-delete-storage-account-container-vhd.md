---
title: Résoudre les erreurs quand vous supprimez des comptes de stockage Azure classiques, des conteneurs ou des disques durs virtuels | Microsoft Docs
description: Comment résoudre les problèmes lors de la suppression de ressources de stockage contenant des VHD attachés.
services: storage
author: AngshumanNayakMSFT
tags: top-support-issue,azure-service-management
ms.service: storage
ms.topic: troubleshooting
ms.date: 01/11/2019
ms.author: annayak
ms.openlocfilehash: c74f2ef9eed25719e722970671406c850b6a59b2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002593"
---
# <a name="troubleshoot-classic-storage-resource-deletion-errors"></a>Résoudre les erreurs de suppression de ressources de stockage classiques
Cet article fournit des conseils pour résoudre les erreurs suivantes quand vous essayez de supprimer un compte de stockage Azure classique, un conteneur ou un fichier *.vhd d’objet blob de pages. 


Cet article aborde uniquement les erreurs des ressources de stockage classique. Si un utilisateur supprime une machine virtuelle classique à l’aide du portail Azure, de PowerShell ou de l’interface CLI, les disques ne sont pas automatiquement supprimés. L’utilisateur a la possibilité de supprimer la ressource « Disque ». Si l’option n’est pas sélectionnée, la ressource « Disque » empêche la suppression du compte de stockage, du conteneur et du fichier *.vhd réel d’objet blob de pages.

Vous trouverez plus d’informations sur les disques Azure [ici](../../virtual-machines/managed-disks-overview.md). Azure empêche la suppression d’un disque qui est joint à une machine virtuelle pour éviter une altération des données. Il empêche également la suppression des conteneurs et des comptes de stockage qui ont un objet blob de pages joint à une machine virtuelle. 

## <a name="what-is-a-disk"></a>Qu’est-ce qu’un « Disque » ?
Une ressource « Disque » est utilisée pour monter un fichier *.vhd d’objet blob de pages sur une machine virtuelle, comme un disque de système d’exploitation ou un disque de données. Une ressource de disque de système d’exploitation ou de disque de données, tant qu’elle n’est pas supprimée, conserve un bail sur le fichier *.vhd. Aucune ressource de stockage dans le chemin indiqué dans l’image ci-dessous ne peut être supprimée si une ressource « Disque » pointe vers elle.

![Capture d’écran du portail, avec le volet « Propriété » de disque (classique) ouvert](./media/storage-classic-cannot-delete-storage-account-container-vhd/Disk_Lease_Illustration.jpg) 


## <a name="steps-while-deleting-a-classic-virtual-machine"></a>Étapes de suppression d’une machine virtuelle classique 

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]


1. Supprimez la machine virtuelle classique.
2. Si la case « Disques » est cochée, le **bail du disque** (voir la figure ci-dessus) associé au fichier *.vhd d’objet blob de pages est résilié. Le fichier *.vhd réel d’objet blob de pages existe toujours dans le compte de stockage.
![Capture d’écran montrant une boîte de dialogue demandant de confirmer la suppression d’une machine virtuelle](./media/storage-classic-cannot-delete-storage-account-container-vhd/steps_while_deleting_classic_vm.jpg) 

3. Une fois le bail de disque résilié, le ou les objets blob de pages peuvent être supprimés. Un compte de stockage ou un conteneur peut être supprimé une fois que toutes les ressources « Disque » qu’il contient sont supprimées.

>[!NOTE] 
>Si l’utilisateur supprime la machine virtuelle, mais pas le disque dur virtuel, les frais de stockage restent appliqués au fichier *.vhd d’objet blob de pages. Les frais sont calculés en fonction du type de compte de stockage, consultez la [page des prix](https://azure.microsoft.com/pricing/details/storage/) pour plus d’informations. Si l’utilisateur ne veut plus utiliser les disques durs virtuels, supprimez-les pour éviter d’engendrer des frais. 

## <a name="unable-to-delete-storage-account"></a>Impossible de supprimer le compte de stockage 

Quand l’utilisateur tente de supprimer un compte de stockage classique qui n’est plus nécessaire, le comportement suivant peut se produire.

#### <a name="azure-portal"></a>Portail Azure 
L’utilisateur accède au compte de stockage classique dans le [portail Azure](https://portal.azure.com) et clique sur **Supprimer**, le message suivant s’affiche : 

Avec des disques « attachés » à une machine virtuelle

![Capture d’écran montrant un message expliquant pourquoi un compte de stockage ne peut pas être supprimé.](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_storage_account_disks_attached_portal.jpg) 


Avec des disques « non attachés » à une machine virtuelle

![Capture d’écran du portail, avec le volet sans erreur « Supprimer » de la machine virtuelle (classique) ouvert](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_storage_account_disks_unattached_portal.jpg)


#### <a name="azure-powershell"></a>Azure PowerShell
L’utilisateur tente de supprimer un compte de stockage qui n’est plus utilisé à l’aide des applets de commande PowerShell classiques. Le message suivant s’affiche :

> <span style="color:cyan">**Remove-AzureStorageAccount -StorageAccountName myclassicaccount**</span>
> 
> <span style="color:red">Remove-AzureStorageAccount : BadRequest : Le compte de stockage myclassicaccount a une ou plusieurs images et/ou disques actifs, par ex.  
> myclassicaccount. Supprimez ces images et/ou disques avant de supprimer ce compte de stockage.</span>

## <a name="unable-to-delete-storage-container"></a>Impossible de supprimer le conteneur de stockage

Quand l’utilisateur tente de supprimer un conteneur d’objets blob de stockage classique qui n’est plus nécessaire, le comportement suivant peut se produire.

#### <a name="azure-portal"></a>Portail Azure 
Le portail Azure n’autorise pas l’utilisateur à supprimer un conteneur si un bail de « Disques » pointe vers un fichier *.vhd d’objet blob de pages dans le conteneur. C’est un comportement normal qui permet d’empêcher la suppression accidentelle d’un fichier VHD avec un bail de disque actif. 

![Capture d’écran du portail, avec le volet « Liste » du conteneur de stockage ouvert](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_container_portal.jpg)


#### <a name="azure-powershell"></a>Azure PowerShell
Si l’utilisateur choisit d’effectuer la suppression à l’aide de PowerShell, l’erreur suivante se produit. 

> <span style="color:cyan">**Remove-AzureStorageContainer -Context $context -Name vhds**</span>
> 
> <span style="color:red">Remove-AzureStorageContainer : Le serveur distant a retourné une erreur : (412) Il existe actuellement un bail sur le conteneur et aucun ID de bail n’a été spécifié dans la demande. Code d’état HTTP : 412 - Message d’erreur HTTP : Il existe actuellement un bail sur le conteneur et aucun ID de bail n’a été spécifié dans la demande.</span>

## <a name="unable-to-delete-a-vhd"></a>Impossible de supprimer un VHD 

Après avoir supprimé la machine virtuelle Azure, l’utilisateur tente de supprimer le fichier VHD (objet blob de pages) et reçoit le message ci-dessous :

#### <a name="azure-portal"></a>Portail Azure 
Dans le portail, vous avez deux expériences possibles selon la liste d’objets blob sélectionnée pour suppression.

1. Si vous avez sélectionné seulement des objets blob « Alloués », le bouton Supprimer ne s’affiche pas.
![Capture d’écran du portail, avec le volet des objets blob de conteneur ouvert et seuls les objets blob alloués sélectionnés](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_vhd_leased_portal.jpg)


2. Si une combinaison d’objets blob « Alloués » et « Disponibles » est sélectionnée, le bouton « Supprimer » apparaît. Toutefois, l’opération « Supprimer » ne s’applique pas aux objets blob de pages, car ils ont un bail de disque actif. 
![Capture d’écran du portail, avec le volet des objets blob de conteneur ouvert, et à la fois les objets blob alloués et les objets blob disponibles sélectionnés](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_vhd_leased_and_unleased_portal_1.jpg)
![Capture d’écran du portail, avec le volet de suppression ouvert pour l’objet blob sélectionné](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_vhd_leased_and_unleased_portal_2.jpg)

#### <a name="azure-powershell"></a>Azure PowerShell 
Si l’utilisateur choisit d’effectuer la suppression à l’aide de PowerShell, l’erreur suivante se produit. 

> <span style="color:cyan">**Remove-AzureStorageBlob -Context $context -Container vhds -Blob "classicvm-os-8698.vhd"**</span>
> 
> <span style="color:red">Remove-AzureStorageBlob : Le serveur distant a retourné une erreur : (412) Il existe actuellement un bail sur l’objet blob et aucun ID de bail n’a été spécifié dans la demande. Code d’état HTTP : 412 - Message d’erreur HTTP : Il existe actuellement un bail sur l’objet blob et aucun ID de bail n’a été spécifié dans la demande.</span>


## <a name="resolution-steps"></a>Étapes de résolution

### <a name="to-remove-classic-disks"></a>Pour supprimer des disques classiques
Suivez ces étapes dans le portail Azure :
1.  Accédez au [portail Azure](https://portal.azure.com).
2.  Accédez aux Disques (classiques). 
3.  Cliquez sur l’onglet Disques. ![Capture d’écran montrant le portail Azure avec l’option Disques (classiques) sélectionnée, ainsi qu’un nom de disque et un compte de stockage classiques sélectionnés](./media/storage-classic-cannot-delete-storage-account-container-vhd/resolution_click_disks_tab.jpg)
 
4.  Sélectionnez votre disque de données, puis cliquez sur Supprimer le disque.
 ![Capture d’écran montrant le portail Azure avec l’option Disques (classiques) sélectionnée, ainsi qu’un disque de données sélectionné et l’option de suppression](./media/storage-classic-cannot-delete-storage-account-container-vhd/resolution_click_delete_disk.jpg)
 
5.  Recommencez l’opération de suppression qui a échoué précédemment.
6.  Un compte de stockage ou un conteneur ne peut pas être supprimé tant qu’il a un disque.

### <a name="to-remove-classic-images"></a>Pour supprimer des images classiques   
Suivez ces étapes dans le portail Azure :
1.  Accédez au [portail Azure](https://portal.azure.com).
2.  Accédez aux Images de système d’exploitation (classiques).
3.  Supprimez l’image.
4.  Recommencez l’opération de suppression qui a échoué précédemment.
5.  Un compte de stockage ou un conteneur ne peut pas être supprimé tant qu’il a une image.
