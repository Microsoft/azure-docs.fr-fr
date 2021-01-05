---
title: Automatiser les migrations VMware sans agent dans Azure Migrate
description: Décrit comment utiliser des scripts pour migrer un grand nombre de machines virtuelles VMware dans Azure Migrate
author: rahulg1190
ms.author: rahugup
ms.manager: bsiva
ms.topic: how-to
ms.date: 10/30/2020
ms.openlocfilehash: cdae1fe13f8e08cb6b817f8ec6431c77013020d7
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754264"
---
# <a name="scale-migration-of-vmware-vms"></a>Mettre à l’échelle une migration de machines virtuelles VMware 

Cet article vous aide à comprendre le processus d’utilisation de scripts pour migrer un grand nombre de machines virtuelles VMware à l’aide de la méthode sans agent. Pour mettre à l’échelle des migrations, utilisez le [module PowerShell pour Azure Migrate](./tutorial-migrate-vmware-powershell.md). 

Les scripts d’automatisation de migration VMware Azure Migrate sont disponibles en téléchargement dans le dépôt [Exemples Azure PowerShell](https://github.com/Azure/azure-docs-powershell-samples/tree/master/azure-migrate/migrate-at-scale-vmware-agentles) sur GitHub. Les scripts peuvent être utilisés pour migrer des machines virtuelles VMware vers Azure à l’aide de la méthode de migration sans agent. Les commandes PowerShell Azure Migrate utilisées dans ces scripts sont documentées [ici](./tutorial-migrate-vmware-powershell.md).

## <a name="current-limitations"></a>Limites actuelles
- Ces scripts prennent en charge la migration de machines virtuelles VMware avec tous les disques. Vous pouvez mettre à jour les scripts si vous souhaitez répliquer de manière sélective les disques attachés à une machine virtuelle VMware. 
- Les scripts prennent en charge l’utilisation des recommandations d’évaluation. Si les recommandations d’évaluation ne sont pas utilisées, tous les disques attachés à la machine virtuelle VMware sont migrés vers le même type de disque managé (Standard ou Premium). Vous pouvez mettre à jour les scripts si vous souhaitez utiliser plusieurs types de disques managés avec la même machine virtuelle.

## <a name="prerequisites"></a>Prérequis

- [Suivre le tutoriel sur la découverte](tutorial-discover-vmware.md) afin de configurer Azure et VMware pour la migration.
- Nous vous recommandons de suivre le deuxième tutoriel afin d’[évaluer les machines virtuelles VMware](./tutorial-assess-vmware-azure-vm.md) avant de les migrer vers Azure.
- Vous devez disposer du module Azure PowerShell `Az`. Si vous devez installer ou mettre à niveau Azure PowerShell, suivez le [guide d’installation et de configuration d’Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="install-azure-migrate-powershell-module"></a>Installer le module PowerShell Azure Migrate

Le module PowerShell Azure Migrate est disponible en préversion. Vous devez installer le module PowerShell à l’aide de la commande suivante. 

```azurepowershell
Install-Module -Name Az.Migrate 
```

## <a name="csv-input-file"></a>Fichier d’entrée CSV
Une fois tous les prérequis satisfaits, vous devez créer un fichier CSV qui comporte des données pour chaque machine virtuelle source que vous voulez migrer. Tous les scripts sont conçus pour fonctionner sur le même fichier CSV. À titre de référence, un exemple de modèle CSV est disponible dans le dossier des scripts. Le fichier CSV est configurable afin que vous puissiez utiliser des recommandations d’évaluation et même spécifier si certaines opérations ne doivent pas être déclenchées pour une machine virtuelle particulière. 

> [!NOTE]
> Le même fichier CSV peut être utilisé pour migrer des machines virtuelles dans plusieurs projets Azure Migrate.

### <a name="csv-file-schema"></a>Schéma de fichier CSV

**En-tête de colonne** | **Description**
--- | ---
AZMIGRATEPROJECT_SUBSCRIPTION_ID | Fournissez l’ID d’abonnement du projet Azure Migrate.
AZMIGRATEPROJECT_RESOURCE_GROUP_NAME | Indiquez le nom du groupe de ressources Azure Migrate.
AZMIGRATEPROJECT_NAME | Indiquez le nom du projet Azure Migrate vers lequel vous souhaitez migrer les serveurs. 
SOURCE_MACHINE_NAME | Indiquez le nom convivial (nom d’affichage) de la machine virtuelle détectée dans le projet Azure Migrate.
AZMIGRATEASSESSMENT_NAME | Indiquez le nom de l’évaluation qui doit être exploitée pour la migration.
AZMIGRATEGROUP_NAME | Indiquez le nom du groupe utilisé pour l’évaluation Azure Migrate.
TARGET_RESOURCE_GROUP_NAME | Indiquez le nom du groupe de ressources Azure vers lequel la machine virtuelle doit être migrée.
TARGET_VNET_NAME| Indiquez le nom du réseau virtuel Azure que la machine virtuelle migrée doit utiliser.
TARGET_SUBNET_NAME | Indiquez le nom du sous-réseau dans le réseau virtuel cible que la machine virtuelle migrée doit utiliser. Si ce champ n’est pas renseigné, le sous-réseau « par défaut » sera utilisé.
TARGET_MACHINE_NAME | Indiquez le nom que la machine virtuelle migrée doit utiliser dans Azure. Si ce champ n’est pas renseigné, le nom de l’ordinateur source est utilisé.  
TARGET_MACHINE_SIZE | Indiquez la référence SKU que la machine virtuelle doit utiliser dans Azure. Pour migrer une machine virtuelle vers la machine virtuelle D2_v2 dans Azure, spécifiez « Standard_D2_v2 » comme valeur dans ce champ. Si vous utilisez une évaluation, cette valeur sera alors calculée sur la base de la recommandation d’évaluation.
LICENSE_TYPE | Spécifiez si vous souhaitez utiliser Azure Hybrid Benefit pour les machines virtuelles Windows Server. Utilisez la valeur « WindowsServer » pour tirer parti d’Azure Hybrid Benefit. Sinon, laissez vide ou utilisez « NoLicenseType ».
OS_DISK_ID | Indiquez l’ID de disque du système d’exploitation pour la machine virtuelle à migrer. L’ID de disque à utiliser est la propriété UUID (identificateur unique) pour le disque récupéré à l’aide de la cmdlet Get-AzMigrateServer. Si aucune valeur n’est fournie, le script utilisera le premier disque de la machine virtuelle comme disque de système d’exploitation.
TARGET_DISKTYPE | Indiquez le type de disque à utiliser pour tous les disques de la machine virtuelle dans Azure. Utilisez « Premium_LRS » pour les disques managés Premium, « StandardSSD_LRS » pour les disques SSD Standard et « Standard_LRS » pour utiliser des disques HDD Standard. Si vous choisissez d’utiliser une évaluation, le script les classera par ordre de priorité à l’aide des types de disques recommandés pour chaque disque de la machine virtuelle. Si vous n’utilisez pas l’évaluation ou si vous spécifiez une valeur, le script utilisera des disques HDD standard par défaut.    
AVAILABILITYZONE_NUMBER | Spécifiez le nombre de zones de disponibilité à utiliser pour la machine virtuelle migrée. Vous pouvez laisser ce champ vide si vous ne souhaitez pas utiliser les zones de disponibilité. 
AVAILABILITYSET_NAME | Spécifiez le nom du groupe à haute disponibilité à utiliser pour la machine virtuelle migrée. Vous pouvez laisser ce champ vide si vous ne souhaitez pas utiliser le groupe à haute disponibilité.
TURNOFF_SOURCESERVER | Spécifiez « Y » si vous souhaitez désactiver la machine virtuelle source au moment de la migration. Utilisez « N » dans le cas contraire. Si le champ n’est pas renseigné, le script suppose que la valeur est « N ».
TESTMIGRATE_VNET_NAME | Spécifiez le nom du réseau virtuel à utiliser pour la migration de test.
UPDATED_TARGET_RESOURCE_GROUP_NAME | Si vous souhaitez mettre à jour le groupe de ressources que doit utiliser la machine virtuelle migrée dans Azure, spécifiez le nom du groupe de ressources Azure ; sinon, laissez vide. 
UPDATED_TARGET_VNET_NAME | Si vous souhaitez mettre à jour le réseau virtuel que doit utiliser la machine virtuelle migrée dans Azure, spécifiez le nom du réseau virtuel Azure ; sinon, laissez vide.
UPDATED_TARGET_MACHINE_NAME | Si vous souhaitez mettre à jour le nom que doit utiliser la machine virtuelle migrée dans Azure, spécifiez le nouveau nom à utiliser ; sinon, laissez vide.
UPDATED_TARGET_MACHINE_SIZE | Si vous souhaitez mettre à jour la SKU que doit utiliser la machine virtuelle migrée dans Azure, spécifiez la nouvelle SKU à utiliser ; sinon, laissez vide.
UPDATED_AVAILABILITYZONE_NUMBER | Si vous souhaitez mettre à jour la zone de disponibilité que doit utiliser la machine virtuelle migrée dans Azure, spécifiez la nouvelle zone de disponibilité à utiliser ; sinon, laissez vide.
UPDATED_AVAILABILITYSET_NAME | Si vous souhaitez mettre à jour le groupe à haute disponibilité que doit utiliser la machine virtuelle migrée dans Azure, spécifiez le nouveau groupe à haute disponibilité à utiliser ; sinon, laissez vide.
UPDATE_NIC1_ID | Spécifiez l’ID de la carte réseau à mettre à jour. Si le champ n’est pas renseigné, le script suppose que la valeur est la première carte réseau de la machine virtuelle détectée. Si vous ne souhaitez pas mettre à jour la carte d’interface a machine virtuelle, laissez vides tous les champs contenant le nom de la carte réseau. 
UPDATED_TARGET_NIC1_SELECTIONTYPE | Spécifiez la valeur à utiliser pour cette carte réseau. Utilisez les valeurs « Primary », « Secondary » ou « DoNotCreate » pour spécifier si cette carte réseau doit être principale ou secondaire ou si elle ne doit pas être créée sur la machine virtuelle migrée. Une seule carte réseau peut être spécifiée comme carte réseau principale pour la machine virtuelle. Laissez ce champ vide si vous ne souhaitez pas effectuer de mise à jour.
UPDATED_TARGET_NIC1_SUBNET_NAME | Spécifiez le nom du sous-réseau à utiliser pour la carte réseau sur la machine virtuelle migrée. Laissez ce champ vide si vous ne souhaitez pas effectuer de mise à jour.
UPDATED_TARGET_NIC1_IP | Si vous souhaitez utiliser une adresse IP statique, spécifiez l’adresse IPv4 que la carte réseau doit utiliser sur la machine virtuelle migrée. Utilisez « auto » si vous souhaitez attribuer automatiquement l’adresse IP. Laissez ce champ vide si vous ne souhaitez pas effectuer de mise à jour.
UPDATE_NIC2_ID | Spécifiez l’ID de la carte réseau à mettre à jour. Si le champ n’est pas renseigné, le script suppose que la valeur est la deuxième carte réseau de la machine virtuelle détectée. Si vous ne souhaitez pas mettre à jour la carte d’interface a machine virtuelle, laissez vides tous les champs contenant le nom de la carte réseau.
UPDATED_TARGET_NIC2_SELECTIONTYPE | Spécifiez la valeur à utiliser pour cette carte réseau. Utilisez les valeurs « Primary », « Secondary » ou « DoNotCreate » pour spécifier si cette carte réseau doit être principale ou secondaire ou si elle ne doit pas être créée sur la machine virtuelle migrée. Une seule carte réseau peut être spécifiée comme carte réseau principale pour la machine virtuelle. Laissez ce champ vide si vous ne souhaitez pas effectuer de mise à jour.
UPDATED_TARGET_NIC2_SUBNET_NAME | Spécifiez le nom du sous-réseau à utiliser pour la carte réseau sur la machine virtuelle migrée. Laissez ce champ vide si vous ne souhaitez pas effectuer de mise à jour.
UPDATED_TARGET_NIC2_IP | Si vous souhaitez utiliser une adresse IP statique, spécifiez l’adresse IPv4 que la carte réseau doit utiliser sur la machine virtuelle migrée. Utilisez « auto » si vous souhaitez attribuer automatiquement l’adresse IP. Laissez ce champ vide si vous ne souhaitez pas effectuer de mise à jour.
OK_TO_UPDATE | Utilisez « Y » pour indiquer que les propriétés de la machine virtuelle doivent être mises à jour lorsque vous exécutez le script AzMigrate_UpdateMachineProperties. Utilisez « N » ou laissez vide dans le cas contraire.
OK_TO_MIGRATE | Utilisez « Y » pour indiquer que la machine virtuelle doit être migrée lorsque vous exécutez le script AzMigrate_StartMigration. Utilisez « N » ou laissez vide si vous ne souhaitez pas migrer la machine virtuelle. 
OK_TO_USE_ASSESSMENT | Utilisez « Y » pour indiquer que la machine virtuelle doit démarrer la réplication en utilisant les recommandations d’évaluation lorsque vous exécutez le script AzMigrate_StartReplication. Cela remplacera les valeurs TARGET_MACHINE_SIZE et TARGET_DISKTYPE dans le fichier CSV. Utilisez « N » ou laissez vide si vous ne souhaitez pas utiliser les recommandations d’évaluation.
OK_TO_TESTMIGRATE | Utilisez « Y » pour indiquer que la machine virtuelle doit être migrée à titre de test lorsque vous exécutez le script AzMigrate_StartTestMigration. Utilisez « N » ou laissez vide si vous ne souhaitez pas migrer la machine virtuelle à titre de test. 
OK_TO_RETRIEVE_REPLICATIONSTATUS | Utilisez « Y » pour indiquer que l’état de réplication de la machine virtuelle doit être mis à jour lorsque vous exécutez le script AzMigrate_ReplicationStatus. Utilisez « N » ou laissez vide si vous ne souhaitez pas mettre à jour l’état de la réplication.
OK_TO_CLEANUP | Utilisez « Y » pour indiquer que la réplication de la machine virtuelle doit être supprimée lorsque vous exécutez le script AzMigrate_StopReplication. Utilisez « N » ou laissez vide dans le cas contraire.
OK_TO_TESTMIGRATE_CLEANUP | Utilisez « Y » pour indiquer que la migration de test pour la machine virtuelle doit être supprimée lorsque vous exécutez le script AzMigrate_CleanUpTestMigration. Utilisez « N » ou laissez vide dans le cas contraire.


## <a name="script-execution"></a>Exécution des scripts

Une fois que le fichier CSV prêt, vous pouvez exécuter les étapes suivantes pour migrer vos machines virtuelles VMware locales.

**N° de l’étape** | **Nom du script** | **Description**
--- | --- | ---
1 | AzMigrate_StartReplication.ps1 | Activez la réplication pour toutes les machines virtuelles listées dans le CSV. Le script crée une sortie CSV et un fichier journal pour la résolution des problèmes.
2 | AzMigrate_ReplicationStatus.ps1 | Vérifiez l’état de la réplication. Le script crée une sortie CSV avec l’état de chaque machine virtuelle et un fichier journal pour la résolution des problèmes.
3 | AzMigrate_UpdateMachineProperties.ps1 | Une fois que les machines virtuelles ont terminé la réplication initiale, utilisez ce script pour mettre à jour les propriétés cibles de la machine virtuelle (propriétés réseau et de calcul). Le script crée une sortie CSV avec les détails de la tâche pour chaque machine virtuelle.
4 | AzMigrate_StartTestMigration.ps1 |  Démarrez le test de basculement pour toutes les machines virtuelles répertoriées dans le fichier CSV qui sont configurées pour la migration de test. Le script crée une sortie CSV avec les détails de la tâche pour chaque machine virtuelle.
5 | AzMigrate_CleanUpTestMigration.ps1 | Une fois que vous avez validé manuellement les machines virtuelles qui ont été basculée à titre de test, utilisez ce script pour supprimer les machines virtuelles de basculement de test pour toutes les machines virtuelles répertoriées dans le fichier CSV qui sont configurées pour la suppression de la migration de test. Le script crée une sortie CSV avec les détails de la tâche pour chaque machine virtuelle.
6 | AzMigrate_StartMigration.ps1 | Démarrez la migration pour toutes les machines virtuelles répertoriées dans le fichier CSV qui sont configurées pour la migration. Le script crée une sortie CSV avec les détails de la tâche pour chaque machine virtuelle.
7 | AzMigrate_StopReplication.ps1 | Arrête la réplication de la machine virtuelle une fois qu’elle a été migrée ou si vous souhaitez annuler la réplication pour d’autres raisons. Le script crée une sortie CSV avec les détails de la tâche pour chaque machine virtuelle.


Les scripts suivants sont appelés par d’autres scripts pour toutes les opérations Azure Migrate telles que l’activation de la réplication, le démarrage de la migration de test, la mise à jour des propriétés de machine virtuelle, etc. Assurez-vous que tous les scripts sont présents dans le même dossier ou sous le même chemin d’accès. 

**N° de l’étape** | **Nom du script** | **Description**
--- | --- | ---
1 | AzMigrate_Shared.ps1 | Script courant contenant des fonctions permettant de récupérer des propriétés d’évaluation (par le biais d’API), des machines virtuelles découvertes et des machines virtuelles de réplication. 
2 | AzMigrate_CSV_Processor.ps1 | Script courant contenant les fonctions utilisées pour les opérations de fichier CSV, notamment le chargement, la lecture et l’impression des journaux. 
3 | AzMigrate_Logger.ps1 | Script courant appelé pour générer le fichier journal pour les opérations d’automatisation Azure Migrate. Le fichier journal sera au format log.Scriptname.Datetime.txt.

En plus de ce qui précède, le dossier comprend également AzMigrate_Template.ps1 qui contient l’infrastructure de base pour la création de scripts personnalisés pour différentes opérations Azure Migrate. 

### <a name="script-execution-syntax"></a>Syntaxe d’exécution de script

Une fois que vous avez téléchargé les scripts, vous pouvez les exécuter comme suit.

Si vous souhaitez exécuter le script pour démarrer la réplication des machines virtuelles en utilisant le fichier Input.csv, utilisez la syntaxe suivante. 

```azurepowershell
".\AzMigrate_StartReplication.ps1" .\Input.csv 
```

Pour en savoir plus sur l’utilisation d’Azure PowerShell pour la migration de machines virtuelles VMware avec Azure Migrate, suivez ce [tutoriel](./tutorial-migrate-vmware-powershell.md).