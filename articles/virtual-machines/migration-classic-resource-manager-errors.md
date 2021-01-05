---
title: Erreurs courantes lors de la migration de Classic vers Azure Resource Manager
description: Cet article répertorie les erreurs les plus courantes et leur atténuation lors la migration de ressources IaaS d’Azure Service Management vers Azure Resource Manager.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: troubleshooting
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: 110d7186db97f6fac91b8fd785384a1c2ed7a8cd
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96301679"
---
# <a name="errors-that-commonly-occur-during-classic-to-azure-resource-manager-migration"></a>Erreurs qui se produisent fréquemment pendant la migration de Classic vers Azure Resource Manager

> [!IMPORTANT]
> Aujourd'hui, environ 90 % des machines virtuelles IaaS utilisent [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). Depuis le 28 février 2020, les machines virtuelles classiques sont dépréciées. Elles seront entièrement mises hors service le 1er mars 2023. [Apprenez-en davantage]( https://aka.ms/classicvmretirement) sur cette désapprobation et [son impact sur vous](classic-vm-deprecation.md#how-does-this-affect-me).

Cet article répertorie les erreurs les plus courantes et leur atténuation lors la migration de ressources IaaS du modèle de déploiement Azure Classic vers la pile Azure Resource Manager.


## <a name="list-of-errors"></a>Liste d’erreurs

| Chaîne d’erreur | Limitation des risques |
| --- | --- |
| Erreur interne du serveur |Dans certains cas, il s’agit d’une erreur temporaire qui disparaît à la tentative suivante. Si elle persiste, [contactez le support technique Azure](../azure-portal/supportability/how-to-create-azure-support-request.md), car un examen des journaux d’activité de la plateforme est nécessaire. <br><br> **REMARQUE :** une fois l'incident pris en charge par l'équipe du support technique, ne tentez aucune mesure d'atténuation car cela pourrait avoir des conséquences inattendues sur votre environnement. |
| La migration n’est pas prise en charge pour le déploiement {deployment-name} dans le service hébergé {hosted-service-name} car il s’agit d’un déploiement PaaS (Web/Worker). |Cela se produit lorsque le déploiement contient un rôle web/de travail. Étant donné que la migration n’est prise en charge que pour les Machines Virtuelles, veuillez supprimer le rôle web/de travail du déploiement et effectuer une nouvelle tentative de migration. |
| Échec du déploiement du modèle {template-name}. CorrelationId={guid} |Sur le serveur principal du service de migration, nous utilisons des modèles Azure Resource Manager pour créer des ressources dans la pile Azure Resource Manager. Les modèles étant idempotents, vous pouvez généralement réessayer en toute sécurité l’opération de migration pour passer outre cette erreur. Si elle persiste, veuillez [contactez le support Azure](../azure-portal/supportability/how-to-create-azure-support-request.md) et lui donner le CorrelationId. <br><br> **REMARQUE :** une fois l'incident pris en charge par l'équipe du support technique, ne tentez aucune mesure d'atténuation car cela pourrait avoir des conséquences inattendues sur votre environnement. |
| Le réseau virtuel {virtual-network-name} n’existe pas. |Cela peut se produire si vous avez créé le réseau virtuel dans le nouveau Portail Azure. Le nom réel du réseau virtuel suit le modèle « Group * \<VNET name>». |
| La machine virtuelle {vm-name} du service hébergé {hosted-service-name} contient une extension {extension-name} qui n’est pas prise en charge dans Azure Resource Manager. Il est recommandé de la désinstaller de la machine virtuelle avant de poursuivre la migration. |Les extensions XML telles que BGInfo 1.\* ne sont pas prises en charge dans Azure Resource Manager. Par conséquent, elles ne peuvent pas faire l’objet d’une migration. Si elles sont toujours installées sur la machine virtuelle, elles sont automatiquement désinstallées avant la fin de la migration. |
| La machine virtuelle {vm-name} du service hébergé {hosted-service-name} contient l’extension VMSnapshot/VMSnapshotLinux, dont la migration n’est actuellement pas prise en charge. Désinstallez-la de la machine virtuelle et rajoutez-la à l’aide d’Azure Resource Manager une fois la migration terminée. |C’est le scénario dans lequel la machine virtuelle est configurée pour la Sauvegarde Azure. Étant donné qu’il n’est pas pris en charge pour le moment, suivez la solution de contournement à la page https://aka.ms/vmbackupmigration. |
| La machine virtuelle {vm-name} du service hébergé {hosted-service-name} contient une extension {extension-name} dont l’état n’est pas généré à partir de la machine virtuelle. Par conséquent, elle ne peut pas faire l’objet d’une migration. Assurez-vous que l’état de l’extension est généré ou désinstallez l’extension de la machine virtuelle et effectuez une nouvelle tentative de migration. <br><br> La machine virtuelle {vm-name} du service hébergé {hosted-service-name} contient une extension {extension-name} qui génère l’état de gestionnaire : {handler-status}. Par conséquent, la machine virtuelle ne peut pas être migrée. Assurez-vous que l’état du gestionnaire de l’extension généré est {handler-status} ou désinstallez-la de la machine virtuelle et effectuez une nouvelle tentative de migration. <br><br> L’agent de la machine virtuelle {vm-name} du service hébergé {hosted-service-name} génère l’état global « Pas prêt ». Par conséquent, la machine virtuelle ne peut pas faire l’objet d’une migration, si elle a une extension migrable. Assurez-vous que l’agent de la machine virtuelle génère l’état global de l’agent « Prêt ». Consultez la page https://aka.ms/classiciaasmigrationfaqs. |L’agent invité Azure et les extensions de machines virtuelles ont besoin d’un accès Internet sortant au compte de stockage de la machine virtuelle pour remplir leur état. Causes les plus courantes d’échec de l’état : <li> un groupe de sécurité réseau bloque l’accès sortant à Internet ; <li> si le réseau virtuel a des serveurs DNS locaux et que la connectivité DNS est perdue. <br><br> Si vous continuez à voir l’état « non pris en charge », vous pouvez désinstaller les extensions pour ignorer cette vérification et poursuivre la migration. |
| La migration n’est pas prise en charge pour le déploiement {deployment-name} dans le service hébergé {hosted-service-name} car il contient plusieurs groupes à haute disponibilité. |Actuellement, seuls les services hébergés contenant au maximum un groupe à haute disponibilité peuvent faire l’objet d’une migration. Pour contourner ce problème, déplacez les groupes à haute disponibilité excédentaires et les machines virtuelles qu’ils contiennent vers un autre service hébergé. |
| La migration n’est pas prise en charge pour le déploiement {deployment-name} dans le service hébergé {hosted-service-name} car il contient des machines virtuelles qui ne font pas partie du groupe à haute disponibilité, même si le service hébergé en contient une. |La solution de contournement de ce scénario consiste à déplacer toutes les machines virtuelles dans un même groupe à haute disponibilité ou à supprimer toutes les machines virtuelles du groupe à haute disponibilité dans le service hébergé. |
| Le compte de stockage/service hébergé/réseau virtuel {virtual-network-name} est en cours de migration et, par conséquent, n’est pas modifiable. |Cette erreur se produit lorsque l’opération de migration « Préparer » a été effectuée sur la ressource et qu’une opération susceptible d’apporter une modification à la ressource est déclenchée. En raison du verrouillage du plan de gestion après l’opération « Préparer », les modifications apportées à la ressource sont bloquées. Pour déverrouiller le plan de gestion, vous pouvez exécuter l’opération de migration « Valider » pour terminer la migration ou l’opération de migration « Abandonner » pour annuler l’opération « Préparer ». |
| La migration n'est pas autorisée pour le service hébergé {hosted-service-name} car il contient une machine virtuelle {vm-name} à l'état : RoleStateUnknown. La migration n’est autorisée que lorsque la machine virtuelle est à l’un des états suivants : En cours d’exécution, Arrêtée, Arrêtée libérée. |La machine virtuelle peut être en cours de transition d’état, ce qui se produit habituellement lors d’une opération de mise à jour sur le service hébergé, comme un redémarrage, l’installation d’une extension, etc. Il est recommandé d’attendre que l’opération de mise à jour soit terminée sur le service hébergé avant d’essayer d’effectuer la migration. |
| Le déploiement {nom} dans le service hébergé {nom} contient une machine virtuelle {nom} avec un disque de données {nom} dont les octets de taille d’objet Blob physique {taille} ne correspondent pas aux octets de taille logique de disque de données de la machine virtuelle {taille}. La migration se déroule sans spécifier une taille pour le disque de données pour la machine virtuelle Azure Resource Manager. | Cette erreur se produit si vous avez redimensionné l’objet Blob de disque dur virtuel sans mettre à jour la taille du modèle de l’API de machine virtuelle. Vous trouverez les étapes de correction détaillées [ci-dessous](#vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes).|
| Une exception de stockage s’est produite lors de la validation du disque de données {nom du disque de données} avec le lien du support {URI du disque de données} pour la machine virtuelle {nom de la machine virtuelle} dans le service cloud {nom du service cloud}. Veuillez vous assurer que le lien du support de disque dur virtuel est accessible pour cette machine virtuelle | Cette erreur peut se produire si les disques de la machine virtuelle ont été supprimés ou ne sont pas plus accessibles. Vérifiez que les disques de la machine virtuelle existent.|
| La machine virtuelle {nom de la machine virtuelle} dans le service hébergé {nom du service cloud} contient un disque avec un élément MediaLink {vhd-uri} ayant le nom d’objet blob {vhd-nom d’objet blob} qui n’est pas pris en charge dans Azure Resource Manager. | Cette erreur se produit lorsque le nom de l’objet blob comporte un « / » lequel n’est pas pris en charge dans le fournisseur de ressources de calcul actuellement. |
| La migration n’est pas autorisée pour le déploiement {deployment-name} dans le service hébergé {cloud-service-name}, car celui-ci ne se trouve pas dans l’étendue régionale. Consultez l’adresse https:\//aka.ms/regionalscope pour déplacer ce déploiement vers l’étendue régionale. | En 2014, Azure a annoncé que les ressources réseau allaient être déplacées d’une étendue de niveau cluster vers une étendue régionale. Pour plus d’informations, consultez [https://aka.ms/regionalscope](https://aka.ms/regionalscope). Cette erreur se produit lorsque le déploiement en cours de migration n’a pas fait l’objet d’une opération de mise à jour qui le déplace automatiquement vers une étendue régionale. Le meilleur moyen de contourner ce problème consiste à ajouter un point de terminaison à une machine virtuelle ou à un disque de données dans la machine virtuelle, puis à retenter la migration. <br> Pour plus d’informations, consultez l’article [Comment configurer des points de terminaison sur une machine virtuelle Windows classique dans Azure](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints#create-an-endpoint) ou [Attacher un disque de données à une machine virtuelle Windows créée avec le modèle de déploiement classique](/azure/virtual-machines/linux/attach-disk-portal).|
| La migration n’est pas prise en charge pour le réseau virtuel {vnet-name} car celui-ci a des déploiements PaaS autres que des déploiements de passerelles. | Cette erreur se produit lorsque vous avez des déploiements PaaS autres que des déploiements de passerelles, comme les services Gestion des API ou Application Gateway qui sont connectés au réseau virtuel.|


## <a name="detailed-mitigations"></a>Étapes de correction détaillées

### <a name="vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes"></a>Une machine virtuelle avec un disque de données dont les octets de taille d’objet Blob physique ne correspondent pas à ceux de la taille logique du disque de données de la machine virtuelle.

Cela se produit lorsque la taille logique du disque de données est désynchronisée avec celle de l’objet Blob de disque dur virtuel réel. Vous pouvez facilement vérifier cela à l’aide des commandes suivantes :

#### <a name="verifying-the-issue"></a>Vérification du problème

```powershell
# Store the VM details in the VM object
$vm = Get-AzureVM -ServiceName $servicename -Name $vmname

# Display the data disk properties
# NOTE the data disk LogicalDiskSizeInGB below which is 11GB. Also note the MediaLink Uri of the VHD blob as we'll use this in the next step
$vm.VM.DataVirtualHardDisks


HostCaching         : None
DiskLabel           : 
DiskName            : coreosvm-coreosvm-0-201611230636240687
Lun                 : 0
LogicalDiskSizeInGB : 11
MediaLink           : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
SourceMediaLink     : 
IOType              : Standard
ExtensionData       : 

# Now get the properties of the blob backing the data disk above
# NOTE the size of the blob is about 15 GB which is different from LogicalDiskSizeInGB above
$blob = Get-AzStorageblob -Blob "coreosvm-dd1.vhd" -Container vhds 

$blob

ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudPageBlob
BlobType          : PageBlob
Length            : 16106127872
ContentType       : application/octet-stream
LastModified      : 11/23/2016 7:16:22 AM +00:00
SnapshotTime      : 
ContinuationToken : 
Context           : Microsoft.WindowsAzure.Commands.Common.Storage.AzureStorageContext
Name              : coreosvm-dd1.vhd
```

#### <a name="mitigating-the-issue"></a>Correction du problème

```powershell
# Convert the blob size in bytes to GB into a variable which we'll use later
$newSize = [int]($blob.Length / 1GB)

# See the calculated size in GB
$newSize

15

# Store the disk name of the data disk as we'll use this to identify the disk to be updated
$diskName = $vm.VM.DataVirtualHardDisks[0].DiskName

# Identify the LUN of the data disk to remove
$lunToRemove = $vm.VM.DataVirtualHardDisks[0].Lun

# Now remove the data disk from the VM so that the disk isn't leased by the VM and it's size can be updated
Remove-AzureDataDisk -LUN $lunToRemove -VM $vm | Update-AzureVm -Name $vmname -ServiceName $servicename

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureVM       213xx1-b44b-1v6n-23gg-591f2a13cd16   Succeeded  

# Verify we have the right disk that's going to be updated
Get-AzureDisk -DiskName $diskName

AffinityGroup        : 
AttachedTo           : 
IsCorrupted          : False
Label                : 
Location             : East US
DiskSizeInGB         : 11
MediaLink            : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
DiskName             : coreosvm-coreosvm-0-201611230636240687
SourceImageName      : 
OS                   : 
IOType               : Standard
OperationDescription : Get-AzureDisk
OperationId          : 0c56a2b7-a325-123b-7043-74c27d5a61fd
OperationStatus      : Succeeded

# Now update the disk to the new size
Update-AzureDisk -DiskName $diskName -ResizedSizeInGB $newSize -Label $diskName

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureDisk     cv134b65-1b6n-8908-abuo-ce9e395ac3e7 Succeeded 

# Now verify that the "DiskSizeInGB" property of the disk matches the size of the blob 
Get-AzureDisk -DiskName $diskName


AffinityGroup        : 
AttachedTo           : 
IsCorrupted          : False
Label                : coreosvm-coreosvm-0-201611230636240687
Location             : East US
DiskSizeInGB         : 15
MediaLink            : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
DiskName             : coreosvm-coreosvm-0-201611230636240687
SourceImageName      : 
OS                   : 
IOType               : Standard
OperationDescription : Get-AzureDisk
OperationId          : 1v53bde5-cv56-5621-9078-16b9c8a0bad2
OperationStatus      : Succeeded

# Now we'll add the disk back to the VM as a data disk. First we need to get an updated VM object
$vm = Get-AzureVM -ServiceName $servicename -Name $vmname

Add-AzureDataDisk -Import -DiskName $diskName -LUN 0 -VM $vm -HostCaching ReadWrite | Update-AzureVm -Name $vmname -ServiceName $servicename

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureVM       b0ad3d4c-4v68-45vb-xxc1-134fd010d0f8 Succeeded      
```

### <a name="moving-a-vm-to-a-different-subscription-after-completing-migration"></a>Déplacer une machine virtuelle vers un autre abonnement après la fin de la migration

Après avoir terminé le processus de migration, vous souhaiterez peut-être déplacer la machine virtuelle vers un autre abonnement. Toutefois, si vous avez une clé secrète ou un certificat sur la machine virtuelle qui fait référence à une ressource Key Vault, le déplacement n’est actuellement pas pris en charge. Les instructions ci-dessous vous permettent de contourner le problème. 

#### <a name="powershell"></a>PowerShell

```powershell
$vm = Get-AzVM -ResourceGroupName "MyRG" -Name "MyVM"
Remove-AzVMSecret -VM $vm
Update-AzVM -ResourceGroupName "MyRG" -VM $vm
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az vm update -g "myrg" -n "myvm" --set osProfile.Secrets=[]
```


## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure](migration-classic-resource-manager-cli.md)
* [Outils de la communauté pour aider à la migration de ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-community-tools.md)
* [Passez en revue les questions fréquemment posées sur la migration des ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-faq.md)
