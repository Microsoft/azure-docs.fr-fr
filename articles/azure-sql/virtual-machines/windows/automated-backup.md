---
title: Sauvegarde automatisée version 2 pour les machines virtuelles SQL Server 2016/2017 Azure | Microsoft Docs
description: Cet article décrit la fonctionnalité de sauvegarde automatisée pour les machines virtuelles SQL Server 2016/2017 s’exécutant sur Azure. Cet article est spécifique aux machines virtuelles utilisant Resource Manager.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: ebd23868-821c-475b-b867-06d4a2e310c7
ms.service: virtual-machines-sql
ms.subservice: backup
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/03/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: f41614d54dc4320f683f406b2882a7b388bb4c3d
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358416"
---
# <a name="automated-backup-v2-for-azure-virtual-machines-resource-manager"></a>Sauvegarde automatisée v2 pour les machines virtuelles Azure (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [SQL Server 2014](automated-backup-sql-2014.md)
> * [SQL Server 2016 +](automated-backup.md)

La sauvegarde automatisée version 2 configure automatiquement une [sauvegarde managée sur Microsoft Azure](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure) pour toutes les bases de données nouvelles et existantes sur une machine virtuelle Azure exécutant les éditions Standard, Entreprise ou Développeur de SQL Server 2016 ou version ultérieure. Cela vous permet de configurer des sauvegardes régulières de base de données utilisant le stockage d’objets blob Azure durable. La sauvegarde automatisée v2 dépend de l’[extension de l’agent IaaS (infrastructure as a service) SQL Server](sql-server-iaas-agent-extension-automate-management.md).

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>Prérequis
Pour utiliser la sauvegarde automatisée version 2, passez en revue les conditions préalables suivantes :

**Système d’exploitation** :

- Windows Server 2012 R2 ou version ultérieure

**Édition/version de SQL Server**:

- SQL Server 2016 ou version ultérieure : Développeur, Standard ou Entreprise

> [!NOTE]
> Pour SQL Server 2014, consultez l’article [Sauvegarde automatisée pour les machines virtuelles SQL Server 2014](automated-backup-sql-2014.md).

**Configuration de la base de données**:

- Les bases de données _utilisateur_ cibles doivent utiliser le modèle de récupération complète. Les bases de données système n’ont pas besoin d’utiliser le mode de récupération complète. Toutefois, si vous avez besoin de sauvegardes de fichier journal pour un modèle ou MSDB, vous devez utiliser le mode de récupération complète. Pour plus d’informations sur l’impact du mode de récupération complète sur les sauvegardes, consultez [Sauvegarde en mode de récupération complète](/previous-versions/sql/sql-server-2008-r2/ms190217(v=sql.105)). 
- La machine virtuelle SQL Server a été enregistrée avec l’extension SQL IaaS Agent en [mode de gestion complet](sql-agent-extension-manually-register-single-vm.md#upgrade-to-full). 
-  La sauvegarde automatisée utilise l’[extension complète de l’agent IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md). Par conséquent, la sauvegarde automatisée est uniquement prise en charge sur les bases de données cibles de l’instance par défaut ou sur une instance nommée unique. S’il n’existe aucune instance par défaut et plusieurs instances nommées, l’extension IaaS SQL échoue et la sauvegarde automatisée ne fonctionnera pas. 

## <a name="settings"></a>Paramètres
Le tableau suivant décrit les options qui peuvent être configurées pour une sauvegarde automatisée v2. Les étapes de la configuration varient selon que vous utilisez les commandes du portail Azure ou Azure Windows PowerShell.

### <a name="basic-settings"></a>Paramètres de base

| Paramètre | Plage (par défaut) | Description |
| --- | --- | --- |
| **Sauvegarde automatisée** | Activer/Désactiver (désactivé) | Active ou désactive la sauvegarde automatisée d’une machine virtuelle Azure exécutant SQL Server 2016/2017 Développeur, Standard ou Entreprise. |
| **Période de rétention** | 1 à 30 jours (30 jours) | Nombre de jours durant lesquels les sauvegardes sont conservées. |
| **Compte de stockage** | Compte Azure Storage | Compte de stockage Azure à utiliser pour stocker les fichiers de sauvegarde automatisée dans le stockage d’objets blob. Un conteneur est créé à cet emplacement pour stocker tous les fichiers de sauvegarde. La convention de dénomination des fichiers de sauvegarde inclut la date, l’heure et le GUID de base de données. |
| **Chiffrement** |Activer/Désactiver (désactivé) | Active ou désactive le chiffrement. Quand le chiffrement est activé, les certificats utilisés pour restaurer la sauvegarde se trouvent dans le compte de stockage spécifié. Il utilise le même conteneur de **sauvegarde automatique** avec la même convention de nommage. Si le mot de passe change, un nouveau certificat est généré avec ce mot de passe, mais l’ancien certificat est conservé pour restaurer les sauvegardes antérieures. |
| **Mot de passe** |Texte du mot de passe | Mot de passe pour les clés de chiffrement. Le mot de passe est uniquement requis si le chiffrement est activé. Pour restaurer une sauvegarde chiffrée, vous devez disposer du mot de passe correct et du certificat associé qui a été utilisé lorsque la sauvegarde a été effectuée. |

### <a name="advanced-settings"></a>Paramètres avancés

| Paramètre | Plage (par défaut) | Description |
| --- | --- | --- |
| **Sauvegardes de bases de données système** | Activer/Désactiver (désactivé) | Lorsqu'elle et activée, cette fonctionnalité sauvegarde également les bases de données système : Master, MSDB et Modèle. Pour les bases de données MSDB et Modèle, vérifiez qu’elles sont en mode de récupération complète si vous souhaitez effectuer des sauvegardes de journaux. Les sauvegardes de journaux ne sont jamais effectuées pour Master. Et aucune sauvegarde n’est effectuée pour TempDB. |
| **Planification de sauvegarde** | Manuelle/automatisée (automatisée) | Par défaut, la planification de la sauvegarde varie automatiquement selon la croissance du journal. Une planification de sauvegarde manuelle permet à l’utilisateur de spécifier la fenêtre de temps des sauvegardes. Dans ce cas, les sauvegardes sont effectuées uniquement à une fréquence spécifiée et pendant la fenêtre de temps définie sur un jour donné. |
| **Fréquence de sauvegarde complète** | Quotidienne/hebdomadaire | Fréquence des sauvegardes complètes. Dans les deux cas, les sauvegardes complètes commencent à la prochaine fenêtre de temps planifiée. Si vous sélectionnez l’option Hebdomadaire, les sauvegardes peuvent s’étaler sur plusieurs jours jusqu'à ce que toutes les bases de données aient été sauvegardées avec succès. |
| **Heure de début de la sauvegarde complète** | 00:00 – 23:00 (01:00) | Heure de début d’un jour donné où des sauvegardes complètes peuvent être effectuées. |
| **Fenêtre de temps de la sauvegarde complète** | 1 à 23 heures (1 heure) | Durée de la fenêtre de temps d’un jour donné où des sauvegardes complètes peuvent être effectuées. |
| **Fréquence de sauvegarde des journaux** | 5 à 60 minutes (60 minutes) | Fréquence des sauvegardes de journaux. |

## <a name="understanding-full-backup-frequency"></a>Présentation de la fréquence de sauvegarde complète
Il est important de bien comprendre la différence entre les sauvegardes complètes quotidiennes et hebdomadaires. Considérez les deux exemples de scénarios suivants.

### <a name="scenario-1-weekly-backups"></a>Scénario 1 : Sauvegardes hebdomadaires
Vous disposez d’une machine virtuelle SQL Server qui contient plusieurs bases de données volumineuses.

Lundi, vous activez la sauvegarde automatisée version 2 avec les paramètres suivants :

- Planification de la sauvegarde : **Manuel**
- Fréquence de sauvegarde complète : **Hebdomadaire**
- Heure de début de la sauvegarde complète : **01:00**
- Fenêtre de temps de la sauvegarde complète : **1 heure**

Cela signifie que la prochaine fenêtre de sauvegarde disponible est mardi à 1 h 00 pendant 1 heure. À ce stade, la sauvegarde automatisée commence à sauvegarder vos bases de données une par une. Dans ce scénario, vos bases de données sont suffisamment volumineuses pour pouvoir sauvegarder complètement les deux premières bases de données. Mais après une heure, les bases de données n’ont pas toutes été sauvegardées.

Dans ce cas, Sauvegarde automatisée commencera à sauvegarder les bases de données restantes le lendemain, mercredi à 1 h 00 pendant une heure. Si les bases de données n’ont pas toutes été sauvegardées pendant cette période, une nouvelle tentative de sauvegarde est effectuée le lendemain à la même heure. Cette opération se poursuit jusqu'à ce que toutes les bases de données aient été correctement sauvegardées.

Le mardi suivant, la sauvegarde automatisée commence à nouveau à sauvegarder toutes les bases de données.

Ce scénario montre que la sauvegarde automatisée ne fonctionne que dans la fenêtre de temps spécifiée, et que chaque base de données est sauvegardée une fois par semaine. Il montre également que les sauvegardes peuvent s’étaler sur plusieurs jours dans le cas où il n’est pas possible de terminer toutes les sauvegardes le même jour.

### <a name="scenario-2-daily-backups"></a>Scénario 2 : Sauvegardes quotidiennes
Vous disposez d’une machine virtuelle SQL Server qui contient plusieurs bases de données volumineuses.

Lundi, vous activez la sauvegarde automatisée version 2 avec les paramètres suivants :

- Planification de la sauvegarde : Manuel
- Fréquence de sauvegarde complète : Quotidien
- Heure de début de la sauvegarde complète : 22:00
- Fenêtre de temps de la sauvegarde complète : 6 heures

Cela signifie que la prochaine fenêtre de sauvegarde disponible est lundi à 22 h 00 pendant 6 heures. À ce stade, la sauvegarde automatisée commence à sauvegarder vos bases de données une par une.

Puis des sauvegardes complètes de toutes les bases de données seront relancées mardi à 22 h 00 pendant 6 heures.

> [!IMPORTANT]
> Lorsque vous planifiez des sauvegardes quotidiennes, il est recommandé de planifier une fenêtre de temps assez large pour s’assurer que toutes les bases de données puissent être sauvegardées durant cet intervalle. Ceci est particulièrement important dans le cas où vous avez une grande quantité de données à sauvegarder.

## <a name="configure-new-vms"></a>Configurer de nouvelles machines virtuelles

Utilisez le portail Azure pour configurer la sauvegarde automatisée v2 quand vous créez une machine virtuelle SQL Server 2016 ou 2017 dans le modèle de déploiement Resource Manager.

Sous l’onglet **Paramètres SQL Server**, sélectionnez **Activer** sous **Sauvegarde automatisée**. La capture d’écran suivante du portail Azure montre les paramètres de **Sauvegarde automatisée SQL**.

![Configuration d’une sauvegarde automatisée SQL dans le portail Azure](./media/automated-backup/automated-backup-blade.png)

> [!NOTE]
> La sauvegarde automatisée version 2 est désactivée par défaut.

## <a name="configure-existing-vms"></a>Configurer des machines virtuelles existantes

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Pour les machines virtuelles SQL Server existantes, accédez à la [ressource Machines virtuelles SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource), puis sélectionnez **Sauvegardes** pour configurer vos sauvegardes automatisées.

![Sauvegarde automatisée SQL pour les machines virtuelles existantes](./media/automated-backup/sql-server-configuration.png)


Quand vous avez terminé, cliquez sur le bouton **Appliquer** dans le bas de la page des paramètres de **Sauvegarde** pour enregistrer vos modifications.

Si vous activez la sauvegarde automatisée pour la première fois, Azure configure l’agent IaaS de SQL Server en arrière-plan. Pendant ce temps, le portail Azure n’indiquera peut-être pas que la sauvegarde automatisée est configurée. Patientez quelques minutes jusqu’à ce que l’agent soit installé et configuré. Le portail Azure reflète alors les nouveaux paramètres.

## <a name="configure-with-powershell"></a>Configurer avec PowerShell

Vous pouvez utiliser PowerShell pour configurer une sauvegarde automatisée version 2. Avant de commencer, vous devez :

- [Télécharger et installer la version la plus récente d’Azure PowerShell](https://aka.ms/webpi-azps).
- Ouvrez Windows PowerShell et associez-le à votre compte avec la commande **Connect-AzAccount**.

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

### <a name="install-the-sql-server-iaas-extension"></a>Installer l’extension IaaS SQL Server
Si vous avez configuré une machine virtuelle SQL Server à partir du portail Azure, l’extension IaaS SQL Server devrait déjà être installée. Vous pouvez déterminer si elle est installée pour votre machine virtuelle en appelant la commande **Get-AzVM**, puis en examinant la propriété **Extensions**.

```powershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"

(Get-AzVM -Name $vmname -ResourceGroupName $resourcegroupname).Extensions 
```

Si l’extension de l’agent IaaS SQL Server est installée, elle devrait être listée sous la forme « SqlIaaSAgent » ou « SQLIaaSExtension ». La propriété **ProvisioningState** de l’extension devrait également indiquer « Succeeded » (Réussie). 

Si elle n’est pas installée ou n’a pas pu être provisionnée, vous pouvez l’installer avec la commande suivante. Outre le nom de la machine virtuelle et le groupe de ressources, vous devez également spécifier la région ( **$region**) où se trouve votre machine virtuelle.

```powershell
$region = "EASTUS2"
Set-AzVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "2.0" -Location $region 
```

### <a name="verify-current-settings"></a><a id="verifysettings"></a> Vérifier les paramètres actuels
Si vous avez activé la sauvegarde automatisée lors de l’approvisionnement, vous pouvez utiliser PowerShell pour vérifier votre configuration actuelle. Exécutez la commande **Get-AzVMSqlServerExtension** et examinez la propriété **AutoBackupSettings** :

```powershell
(Get-AzVMSqlServerExtension -VMName $vmname -ResourceGroupName $resourcegroupname).AutoBackupSettings
```

La sortie doit ressembler à ce qui suit :

```
Enable                      : True
EnableEncryption            : False
RetentionPeriod             : 30
StorageUrl                  : https://test.blob.core.windows.net/
StorageAccessKey            :  
Password                    : 
BackupSystemDbs             : False
BackupScheduleType          : Manual
FullBackupFrequency         : WEEKLY
FullBackupStartTime         : 2
FullBackupWindowHours       : 2
LogBackupFrequency          : 60
```

Si votre sortie montre que l’option **Activer** est définie sur **False**, vous devez activer la sauvegarde automatisée. La bonne nouvelle est que vous activez et configurez la sauvegarde automatisée de la même façon. Pour en savoir plus, consultez la section suivante.

> [!NOTE] 
> Si vous vérifiez les paramètres immédiatement après une modification, les anciennes valeurs de configuration peuvent s’afficher. Patientez quelques minutes et revérifiez les paramètres pour vous assurer que vos modifications ont bien été appliquées.

### <a name="configure-automated-backup-v2"></a>Configurer une sauvegarde automatisée version 2
Vous pouvez utiliser PowerShell pour activer la sauvegarde automatisée ainsi que pour modifier sa configuration et son comportement à tout moment. 

Pour commencer, sélectionnez ou créez un compte de stockage pour les fichiers de sauvegarde. Le script suivant sélectionne un compte de stockage ou le crée s’il n’existe pas.

```powershell
$storage_accountname = "yourstorageaccount"
$storage_resourcegroupname = $resourcegroupname

$storage = Get-AzStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region } 
```

> [!NOTE]
> La sauvegarde automatisée ne prend pas en charge le stockage des sauvegardes dans un stockage Premium, mais elle peut effectuer des sauvegardes à partir de disques de machines virtuelles qui utilisent le stockage Premium.

Utilisez ensuite la commande **New-AzVMSqlServerAutoBackupConfig** pour activer et configurer les paramètres de sauvegarde automatisée de version 2 afin de stocker les sauvegardes dans le compte de stockage Azure. Dans cet exemple, les sauvegardes sont définies pour être conservées pendant 10 jours. Les sauvegardes de bases de données système sont activées. Les sauvegardes complètes sont planifiées pour être effectuées toutes les semaines, avec une fenêtre de temps commençant à 20 h 00 pendant deux heures. Les sauvegardes de journaux sont planifiées pour être effectuées toutes les 30 minutes. La seconde commande **Set-AzRmVMSqlServerExtension** met à jour la machine virtuelle Azure spécifiée avec ces paramètres.

```powershell
$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType Manual -FullBackupFrequency Weekly `
    -FullBackupStartHour 20 -FullBackupWindowInHours 2 `
    -LogBackupFrequencyInMinutes 30 

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname 
```

L’installation et la configuration de l’agent IaaS de SQL Server peuvent prendre plusieurs minutes. 

Pour activer le chiffrement, modifiez le script précédent pour transmettre le paramètre **EnableEncryption** avec un mot de passe (chaîne sécurisée) pour le paramètre **CertificatePassword**. Le script suivant active les paramètres de sauvegarde automatisée dans l’exemple précédent et ajoute le chiffrement.

```powershell
$password = "P@ssw0rd"
$encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force  

$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -EnableEncryption -CertificatePassword $encryptionpassword `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType Manual -FullBackupFrequency Weekly `
    -FullBackupStartHour 20 -FullBackupWindowInHours 2 `
    -LogBackupFrequencyInMinutes 30 

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

Pour confirmer que vos paramètres ont été appliqués, [vérifiez la configuration de la sauvegarde automatisée](#verifysettings).

### <a name="disable-automated-backup"></a>Désactiver la sauvegarde automatisée
Pour désactiver la sauvegarde automatisée, exécutez le même script sans le paramètre **-Enable** pour la commande **New-AzVMSqlServerAutoBackupConfig**. L’absence du paramètre **-Enable** indique à la commande de désactiver la fonctionnalité. À l’instar de l’installation, la désactivation de la sauvegarde automatisée peut prendre plusieurs minutes.

```powershell
$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -ResourceGroupName $storage_resourcegroupname

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

### <a name="example-script"></a>Exemple de script
Le script suivant fournit un ensemble de variables que vous pouvez personnaliser afin d’activer et de configurer la sauvegarde automatisée pour votre machine virtuelle. Dans votre cas, vous devrez peut-être personnaliser le script selon vos besoins. Par exemple, vous devrez apporter des modifications si vous souhaitez désactiver la sauvegarde des bases de données système ou activer le chiffrement.

```powershell
$vmname = "yourvmname"
$resourcegroupname = "vmresourcegroupname"
$region = "Azure region name such as EASTUS2"
$storage_accountname = "storageaccountname"
$storage_resourcegroupname = $resourcegroupname
$retentionperiod = 10
$backupscheduletype = "Manual"
$fullbackupfrequency = "Weekly"
$fullbackupstarthour = "20"
$fullbackupwindow = "2"
$logbackupfrequency = "30"

# ResourceGroupName is the resource group which is hosting the VM where you are deploying the SQL Server IaaS Extension 

Set-AzVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "2.0" -Location $region

# Creates/use a storage account to store the backups

$storage = Get-AzStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region }

# Configure Automated Backup settings

$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays $retentionperiod -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname -BackupSystemDbs `
    -BackupScheduleType $backupscheduletype -FullBackupFrequency $fullbackupfrequency `
    -FullBackupStartHour $fullbackupstarthour -FullBackupWindowInHours $fullbackupwindow `
    -LogBackupFrequencyInMinutes $logbackupfrequency

# Apply the Automated Backup settings to the VM

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

## <a name="monitoring"></a>Surveillance

Pour surveiller la sauvegarde automatisée sur SQL Server 2016/2017, deux options s’offrent à vous. Étant donné que la sauvegarde automatisée utilise la fonctionnalité de sauvegarde gérée de SQL Server, les mêmes techniques de surveillance s’appliquent.

Vous pouvez d’abord interroger l’état en appelant [msdb.managed_backup.sp_get_backup_diagnostics](/sql/relational-databases/system-stored-procedures/managed-backup-sp-get-backup-diagnostics-transact-sql). Vous pouvez aussi interroger la fonction table [msdb.managed_backup.fn_get_health_status](/sql/relational-databases/system-functions/managed-backup-fn-get-health-status-transact-sql).

Une autre possibilité consiste à tirer parti de la fonctionnalité intégrée de messagerie de base de données pour les notifications.

1. Appelez la procédure stockée [msdb.managed_backup.sp_set_parameter](/sql/relational-databases/system-stored-procedures/managed-backup-sp-set-parameter-transact-sql) pour affecter une adresse e-mail au paramètre **SSMBackup2WANotificationEmailIds**. 
1. Activez [SendGrid](../../../sendgrid-dotnet-how-to-send-email.md) pour envoyer les e-mails à partir de la machine virtuelle Azure.
1. Utilisez le nom d’utilisateur et le serveur SMTP pour configurer la messagerie de base de données. Vous pouvez configurer la messagerie de base de données dans SQL Server Management Studio ou à l’aide de commandes Transact-SQL. Pour plus d’informations, consultez [Messagerie de base de données](/sql/relational-databases/database-mail/database-mail).
1. [Configurez SQL Server Agent pour qu’il utilise la messagerie de base de données](/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail).
1. Vérifiez que le port SMTP est autorisé dans le pare-feu local de la machine virtuelle et le groupe de sécurité réseau pour la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes
La sauvegarde automatisée v2 configure une sauvegarde managée sur les machines virtuelles Azure. Il est donc important de [lire la documentation relative à la sauvegarde gérée](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure) pour comprendre son comportement et ses implications.

Vous trouverez des conseils supplémentaires pour la sauvegarde et la restauration de SQL Server sur les machines virtuelles Azure dans l'article suivant : [Sauvegarde et restauration pour SQL Server sur des machines virtuelles Azure](backup-restore.md).

Pour plus d’informations sur les autres tâches d’automatisation disponibles, voir [Extension de l’agent IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md).

Pour plus d’informations sur l’exécution de SQL Server sur des machines virtuelles Azure, consultez [Vue d’ensemble de SQL Server sur les machines virtuelles Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).