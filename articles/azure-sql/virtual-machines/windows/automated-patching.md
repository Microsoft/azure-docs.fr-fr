---
title: Mise à jour corrective automatisée pour SQL Server dans Azure Virtual Machines (Resource Manager) | Microsoft Docs
description: Cet article décrit la fonctionnalité de mise à jour corrective automatisée pour les machines virtuelles SQL Server s’exécutant dans Azure à l’aide de Resource Manager.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-resource-manager
ms.assetid: 58232e92-318f-456b-8f0a-2201a541e08d
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/07/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 429fe39f84a54c22fa97178b85f417d76dc84a8e
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359470"
---
# <a name="automated-patching-for-sql-server-on-azure-virtual-machines-resource-manager"></a>Mise à jour corrective automatisée pour SQL Server sur des machines virtuelles Azure (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

La mise à jour corrective automatisée établit une fenêtre de maintenance pour une machine virtuelle Azure exécutant SQL Server. Les mises à jour automatisées ne peuvent être installées qu’au cours de cette fenêtre de maintenance. Pour SQL Server, cette restriction garantit que les mises à jour système et les redémarrages associés ont lieu au meilleur moment possible pour la base de données. 

> [!IMPORTANT]
> Seules les mises à jour Windows et SQL Server **Importantes** et **Critiques** sont installées. D’autres mises à jour SQL Server, telles que les Service Packs et les mises à jour cumulatives qui ne sont pas marquées **Importantes** ni **Critiques**, doivent être installées manuellement. 

La mise à jour corrective automatisée dépend de l’[extension de l’agent d’infrastructure en tant que service (IaaS) SQL Server](sql-server-iaas-agent-extension-automate-management.md).

## <a name="prerequisites"></a>Prérequis
Pour utiliser la mise à jour corrective automatisée, prenez en compte les conditions préalables suivantes :

**Système d’exploitation** :

* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016
* Windows Server 2019

**Version de SQL Server**:

* SQL Server 2008 R2
* SQL Server 2012
* SQL Server 2014
* SQL Server 2016
* SQL Server 2017
* SQL Server 2019

**Azure PowerShell**:

* [Installez les dernières commandes Azure PowerShell](/powershell/azure/) si vous projetez de configurer la mise à jour corrective automatisée avec PowerShell.

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

> [!NOTE]
> La mise à jour corrective automatisée utilise l’extension de l’agent IaaS SQL Server. Les images actuelles de la galerie de machines virtuelles SQL ajoutent cette extension par défaut. Pour plus d’informations, consultez [SQL Server IaaS Agent Extension](sql-server-iaas-agent-extension-automate-management.md)(Extension de l’agent IaaS SQL Server).
> 
> 

## <a name="settings"></a>Paramètres
Le tableau suivant décrit les options qui peuvent être configurées pour une mise à jour corrective automatisée. Les étapes de la configuration varient selon que vous utilisez les commandes du portail Azure ou Azure Windows PowerShell.

| Paramètre | Valeurs possibles | Description |
| --- | --- | --- |
| **Mise à jour corrective automatisée** |Activer/Désactiver (désactivé) |Active ou désactive la mise à jour corrective automatisée pour une machine virtuelle Azure. |
| **Planification de la maintenance** |Tous les jours, lundi, mardi, mercredi, jeudi, vendredi, samedi et dimanche |Planification du téléchargement et de l’installation des mises à jour Windows, SQL Server et Microsoft pour votre machine virtuelle. |
| **Heure de début de la maintenance** |0-24 |Heure locale de début de la mise à jour de la machine virtuelle. |
| **Durée de la fenêtre de maintenance** |30 à 180 |Nombre de minutes autorisées pour terminer le téléchargement et l’installation des mises à jour. |
| **Catégorie de correctif** |Important | Catégorie des mises à jour Windows à télécharger et installer.|

## <a name="configure-in-the-azure-portal"></a>Configurer dans le portail Azure
Vous pouvez utiliser le portail Azure pour configurer une mise à jour corrective automatisée lors du déploiement ou pour des machines virtuelles existantes.

### <a name="new-vms"></a>Nouvelles machines virtuelles
Utilisez le portail Azure pour configurer la mise à jour corrective automatisée lorsque vous créez une machine virtuelle SQL Server dans le modèle de déploiement Resource Manager.

Sous l’onglet **Paramètres SQL Server**, sélectionnez **Modifier la configuration** sous **Mise à jour corrective automatisée**. La capture d’écran suivante du portail Azure illustre le panneau **Mise à jour corrective automatisée SQL** .

![Mise à jour corrective automatisée SQL dans le portail Azure](./media/automated-patching/azure-sql-arm-patching.png)

Pour plus d’informations, consultez [Provisionner une machine virtuelle SQL Server sur Azure](create-sql-vm-portal.md).

### <a name="existing-vms"></a>Machines virtuelles existantes

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Pour les machines virtuelles SQL Server existantes, ouvrez votre [ressource Machines virtuelles SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) et sélectionnez **Mise à jour corrective** sous **Paramètres**. 

![Mise à jour corrective automatisée SQL pour les machines virtuelles existantes](./media/automated-patching/azure-sql-rm-patching-existing-vms.png)


Lorsque vous avez terminé, cliquez sur le bouton **OK** au bas du panneau **Configuration de SQL Server** pour enregistrer vos modifications.

Si vous activez la mise à jour corrective automatisée pour la première fois, Azure configure l’agent IaaS de SQL Server en arrière-plan. Pendant ce temps, le portail Azure n’indique pas que la mise à jour corrective automatisée est configurée. Patientez quelques minutes jusqu’à ce que l’agent soit installé et configuré. Le portail Azure reflète ensuite les nouveaux paramètres.

## <a name="configure-with-powershell"></a>Configurer avec PowerShell
Après avoir approvisionné votre machine virtuelle SQL, utilisez PowerShell pour configurer une mise à jour corrective automatisée.

Dans l’exemple suivant, PowerShell permet de configurer une mise à jour corrective automatisée sur une machine virtuelle SQL Server existante. La commande **New-AzVMSqlServerAutoPatchingConfig** configure une nouvelle fenêtre de maintenance pour les mises à jour automatiques.

```azurepowershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"
$aps = New-AzVMSqlServerAutoPatchingConfig -Enable -DayOfWeek "Thursday" -MaintenanceWindowStartingHour 11 -MaintenanceWindowDuration 120  -PatchCategory "Important"
s
Set-AzVMSqlServerExtension -AutoPatchingSettings $aps -VMName $vmname -ResourceGroupName $resourcegroupname
```

> [!IMPORTANT]
> Si l’extension n’a pas encore été installée, son installation redémarre SQL Server.

En s’appuyant sur cet exemple, le tableau suivant décrit les effets concrets sur la machine virtuelle Azure cible :

| Paramètre | Résultat |
| --- | --- |
| **DayOfWeek** |Les correctifs sont installés tous les jeudis. |
| **MaintenanceWindowStartingHour** |Les mises à jour démarrent à 11 h 00. |
| **MaintenanceWindowsDuration** |Les correctifs doivent être installés dans les 120 minutes. Selon l’heure de début, leur installation doit être terminée à 13 h 00 au plus tard. |
| **PatchCategory** |La seule définition possible pour ce paramètre est **Important**. Cela installe les mises à jour Windows marquées Important ; et pas les mises à jour de SQL Server qui ne sont pas incluses dans cette catégorie. |

L’installation et la configuration de l’agent IaaS de SQL Server peuvent prendre plusieurs minutes.

Pour désactiver la mise à jour corrective automatisée, exécutez le même script sans le paramètre **-Enable** pour la commande **New-AzVMSqlServerAutoPatchingConfig**. L’absence du paramètre **-Enable** indique à la commande de désactiver la fonctionnalité.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les autres tâches d’automatisation disponibles, voir [Extension de l’agent IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md).

Pour plus d’informations sur l’exécution de SQL Server sur des machines virtuelles Azure, consultez [Vue d’ensemble de SQL Server sur les machines virtuelles Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).

