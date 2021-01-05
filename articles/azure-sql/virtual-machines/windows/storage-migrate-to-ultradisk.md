---
title: Migrer le disque du journal vers le disque Ultra
description: Découvrez comment migrer votre disque du journal SQL Server sur des machines virtuelles Azure vers un disque Ultra Azure pour tirer parti des performances élevées et de la faible latence.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 07/09/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 42d7760d25f6ab591c19889eb2159711d6de1b07
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356750"
---
# <a name="migrate-log-disk-to-ultra-disk"></a>Migrer le disque du journal vers le disque Ultra
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Les disques Ultra Azure offrent un haut débit, un nombre élevé d’E/S par seconde et un stockage sur disque à faible latence cohérent pour SQL Server sur des machines virtuelles Azure. 

Cet article vous apprend à migrer votre disque du journal vers un disque SSD Ultra pour tirer parti des avantages en termes de performances offerts par les disques Ultra. 

## <a name="back-up-database"></a>Sauvegarder la base de données

Effectuez une [sauvegarde complète](backup-restore.md) de votre base de données. 

## <a name="attach-disk"></a>Attacher un disque

Attachez le disque SSD Ultra à votre machine virtuelle une fois que vous avez activé la compatibilité avec les disques Ultra sur la machine virtuelle. 

Le disque Ultra est pris en charge sur un sous-ensemble de tailles et de régions de machine virtuelle. Avant de continuer, vérifiez que votre machine virtuelle se présente dans une région, une zone et une taille qui prennent en charge le disque Ultra. Vous pouvez [déterminer et valider la taille et la région de machine virtuelle](../../../virtual-machines/disks-enable-ultra-ssd.md#determine-vm-size-and-region-availability) avec Azure CLI ou PowerShell. 

### <a name="enable-compatibility"></a>Activer la compatibilité

Pour activer la compatibilité, procédez comme suit :

1. Accédez à votre machine virtuelle dans le [portail Azure](https://portal.azure.com/). 
1. Arrêtez/libérez la machine virtuelle. 
1. Sélectionnez **Disques** sous **Paramètres**, puis **Paramètres supplémentaires**. 

   :::image type="content" source="media/storage-migrate-to-ultradisk/additional-disks-settings-azure-portal.png" alt-text="Sélectionner des paramètres supplémentaires pour les disques sous Paramètres dans le portail Azure":::

1. Sélectionnez **Oui** pour **Activer la compatibilité avec les disques Ultra**. 

   :::image type="content" source="../../../virtual-machines/media/virtual-machines-disks-getting-started-ultra-ssd/ultra-options-yes-enable.png" alt-text="Capture d’écran montrant l’option Oui.":::

1. Sélectionnez **Enregistrer**. 



### <a name="attach-disk"></a>Attacher un disque

Utilisez le portail Azure pour attacher un disque Ultra à votre machine virtuelle. Pour plus d’informations, consultez [Attachement d’un Disque Ultra](../../../virtual-machines/disks-enable-ultra-ssd.md#attach-an-ultra-disk).

Une fois le disque attaché, redémarrez votre machine virtuelle à l’aide du portail Azure. 



## <a name="format-disk"></a>Formater le disque

Connectez-vous à votre machine virtuelle et formatez votre disque Ultra.  

Pour formater votre disque Ultra, procédez comme suit :

1. Connectez-vous à votre machine virtuelle à l’aide du protocole RDP (Remote Desktop Protocol).
1. Utilisez [Gestion des disques](/windows-server/storage/disk-management/overview-of-disk-management) pour formater et partitionner votre disque Ultra récemment attaché. 


## <a name="use-disk-for-log"></a>Utiliser le disque pour le journal

Configurez SQL Server pour utiliser le nouveau lecteur de journal. Vous pouvez le faire avec Transact-SQL (T-SQL) ou SQL Server Management Studio (SSMS). Le compte utilisé pour le compte de service SQL Server doit avoir le contrôle total du nouvel emplacement du fichier journal. 

### <a name="configure-permissions"></a>Configurer les autorisations

1. Vérifiez le compte de service utilisé par SQL Server. Vous pouvez le faire en utilisant le Gestionnaire de configuration SQL Server ou Services.msc.
1. Accédez à votre nouveau disque. 
1. Créez un dossier (ou plusieurs dossiers) à utiliser pour votre fichier journal. 
1. Cliquez avec le bouton droit sur le dossier et sélectionnez **Propriétés**.
1. Sous l’onglet **Sécurité**, octroyez un accès Contrôle total au compte de service SQL Server. 
1. Sélectionnez **OK** pour enregistrer vos paramètres. 
1. Répétez cette procédure pour chaque dossier de niveau racine dans lequel vous prévoyez d’avoir des données SQL. 

### <a name="use-new-log-drive"></a>Utiliser le nouveau lecteur de journal 

Une fois l’autorisation accordée, utilisez Transact-SQL (T-SQL) ou SQL Server Management Studio (SSMS) pour détacher la base de données et déplacer les fichiers journaux existants vers le nouvel emplacement.

   > [!CAUTION]
   > Le détachement de la base de données permet de la mettre hors connexion, de fermer les connexions et de restaurer les transactions en cours. Agissez avec précaution et pendant une fenêtre de maintenance de temps d’arrêt. 



# <a name="transact-sql-t-sql"></a>[Transact-SQL (T-SQL)](#tab/tsql)

Utilisez T-SQL pour déplacer les fichiers existants vers un nouvel emplacement :

1. Connectez-vous à votre base de données dans SQL Server Management Studio et ouvrez une fenêtre **Nouvelle requête**. 
1. Récupérez les fichiers et emplacements existants :

   ```sql
   USE AdventureWorks
   GO

   sp_helpfile
   GO
   ```

1. Détachez la base de données : 

   ```sql
   USE master
   GO

   sp_detach_db 'AdventureWorks'
   GO
   ```

1. Utilisez l’Explorateur de fichiers pour déplacer le fichier journal vers le nouvel emplacement sur le disque Ultra. 

1. Attachez la base de données en spécifiant les nouveaux emplacements de fichiers : 

   ```sql
    sp_attach_db 'AdventureWorks'
   'E:\Fixed_FG\AdventureWorks.mdf',
   'E:\Fixed_FG\AdventureWorks_2.ndf',
   'F:\New_Log\AdventureWorks_log.ldf'
   GO
   ```

À ce stade, la base de données est mise en ligne avec le journal au nouvel emplacement. 



# <a name="sql-server-management-studio-ssms"></a>[SQL Server Management Studio (SSMS)](#tab/ssms)

Utilisez SSMS pour déplacer les fichiers existants vers un nouvel emplacement :

1. Connectez-vous à votre base de données dans SQL Server Management Studio (SSMS). 
1. Cliquez avec le bouton droit sur la base de données, sélectionnez **Propriétés**, puis **Fichiers**. 
1. Notez le chemin des fichiers existants. 
1. puis cliquez sur **OK** pour fermer la boîte de dialogue. 
1. Cliquez avec le bouton droit sur la base de données et sélectionnez **Tâches** > **Détacher**. 
1. Suivez les instructions de l’Assistant pour détacher la base de données. 
1. Utilisez l’Explorateur de fichiers pour déplacer manuellement le fichier journal vers le nouvel emplacement.
1. Attachez la base de données dans SQL Server Management Studio
   1. Cliquez avec le bouton droit sur **Bases de données** dans l’**Explorateur d’objets**, puis sélectionnez **Attacher la base de données**. 
   1. À l’aide de la boîte de dialogue, ajoutez chaque fichier, dont le fichier journal dans son nouvel emplacement. 
   1. Sélectionnez **OK** pour attacher la base de données. 

À ce stade, la base de données est mise en ligne avec le journal au nouvel emplacement.

---


## <a name="next-steps"></a>Étapes suivantes

Passez en revue les [bonnes pratiques en matière de performances](performance-guidelines-best-practices.md) pour obtenir des paramètres supplémentaires afin d’améliorer les performances. 

Pour obtenir une vue d’ensemble de SQL Server sur des machines virtuelles Azure, consultez les articles suivants :

- [Vue d’ensemble de SQL Server sur des machines virtuelles Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
- [Vue d’ensemble de SQL Server sur des machines virtuelles Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)
