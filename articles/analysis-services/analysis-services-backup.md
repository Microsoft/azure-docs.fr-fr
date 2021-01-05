---
title: Sauvegarde et restauration de la base de données Azure Analysis Services | Microsoft Docs
description: Cet article explique comment sauvegarder et restaurer les métadonnées et les données de modèle à partir d’une base de données Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: af1850f77c1d13c761bfc2a143074b5067b349b4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013001"
---
# <a name="analysis-services-database-backup-and-restore"></a>Sauvegarde et restauration de bases de données Analysis Services

La sauvegarde de bases de données de modèles tabulaires dans Azure Analysis Services est relativement semblable à celle des Analysis Services locaux. La principale différence réside dans le lieu où vous enregistrez vos fichiers de sauvegarde. Les fichiers de sauvegarde doivent être enregistrés dans un conteneur dans un [compte de stockage Azure](../storage/common/storage-account-create.md). Vous pouvez utiliser un compte de stockage et un conteneur que vous avez déjà, ou bien en créer de nouveaux lors de la configuration des paramètres de stockage de votre serveur.

> [!NOTE]
> La création d’un compte de stockage peut entraîner un nouveau service facturable. Pour en savoir plus, consultez [Tarification Azure Storage](https://azure.microsoft.com/pricing/details/storage/blobs/).
> 
> 

> [!NOTE]
> Si le compte de stockage se trouve dans une autre région, les paramètres du pare-feu du compte de stockage doivent être configurés de façon à autoriser l’accès à partir des **Réseaux sélectionnés**. Dans la **plage d’adresses** du pare-feu, spécifiez la plage d’adresses IP pour la région dans laquelle se trouve le serveur Analysis Services. La configuration des paramètres de pare-feu de compte de stockage pour autoriser l’accès à partir de tous les réseaux est prise en charge, mais il est préférable de choisir des réseaux sélectionnés et de spécifier une plage d’adresses IP. Pour en savoir plus, consultez [FAQ sur la connectivité réseau](analysis-services-network-faq.md#backup-and-restore).

Les sauvegardes sont enregistrées avec l’extension .abf. Pour les modèles tabulaires en mémoire, les données et les métadonnées de modèle sont enregistrées. Pour les modèles tabulaires DirectQuery, seules les métadonnées de modèle sont stockées. Les sauvegardes peuvent être compressées et chiffrées, en fonction des options que vous avez choisies.


## <a name="configure-storage-settings"></a>Configurer les paramètres de stockage
Avant de procéder à la sauvegarde, vous devez configurer les paramètres de stockage de votre serveur.


### <a name="to-configure-storage-settings"></a>Pour configurer les paramètres de stockage
1.  Dans le portail Azure > **Paramètres**, cliquez sur **Sauvegarde**.

    ![Sauvegardes dans Paramètres](./media/analysis-services-backup/aas-backup-backups.png)

2.  Cliquez sur **Activé**, puis cliquez sur **Paramètres de stockage**.

    ![Activer](./media/analysis-services-backup/aas-backup-enable.png)

3. Sélectionnez un compte de stockage ou créez-en un nouveau.

4. Sélectionnez un conteneur ou créez-en un nouveau.

    ![Sélectionner un conteneur](./media/analysis-services-backup/aas-backup-container.png)

5. Enregistrez vos paramètres de sauvegarde.

    ![Enregistrer les paramètres de sauvegarde](./media/analysis-services-backup/aas-backup-save.png)

## <a name="backup"></a>Backup

### <a name="to-backup-by-using-ssms"></a>Pour effectuer une sauvegarde à l’aide de SSMS

1. Dans SSMS, effectuez un clic droit sur une base de données > **Sauvegarder**.

2. Dans **Sauvegarder une base de données** > **Fichier de sauvegarde**, cliquez sur **Parcourir**.

3. Dans la boîte de dialogue **Enregistrer le fichier sous**, vérifiez le chemin d’accès au dossier, puis saisissez le nom du fichier de sauvegarde. 

4. Dans la boîte de dialogue **Sauvegarder une base de données**, sélectionnez les options.

    **Autoriser l’écrasement des fichiers** - Sélectionnez cette option pour remplacer les fichiers de sauvegarde portant le même nom. Si cette option n’est pas sélectionnée, le fichier que vous tentez d’enregistrer ne peut pas porter le même nom qu’un fichier se trouvant déjà au même emplacement.

    **Appliquer la compression** - Sélectionnez cette option pour compresser le fichier de sauvegarde. Les fichiers de sauvegarde compressés permettent d’économiser de l’espace disque, mais nécessite une utilisation un peu plus intensive du processeur. 

    **Chiffrer le fichier de sauvegarde** - Sélectionnez cette option pour chiffrer le fichier de sauvegarde. Cette option nécessite un mot de passe fourni par l’utilisateur pour sécuriser le fichier de sauvegarde. Le mot de passe empêche la lecture des données de sauvegarde à des fins autres que les opérations de restauration. Si vous choisissez de chiffrer les sauvegardes, conservez le mot de passe en lieu sûr.

5. Cliquez sur **OK** pour créer et enregistrer le fichier de sauvegarde.


### <a name="powershell"></a>PowerShell
Utilisez l’applet de commande [Backup-ASDatabase](/powershell/module/sqlserver/backup-asdatabase).

## <a name="restore"></a>Restaurer
Lors de la restauration, votre fichier de sauvegarde doit être dans le compte de stockage que vous avez configuré pour votre serveur. Si vous devez déplacer un fichier de sauvegarde d’un emplacement local vers votre compte de stockage, utilisez [Microsoft Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md) ou l’utilitaire de ligne de commande [AzCopy](../storage/common/storage-use-azcopy-v10.md). 



> [!NOTE]
> Si vous procédez à une restauration à partir d’un serveur local, vous devez supprimer tous les utilisateurs de domaine des rôles du modèle, puis les rajouter aux rôles en tant qu’utilisateurs Azure Active Directory.
> 
> 

### <a name="to-restore-by-using-ssms"></a>Pour effectuer une restauration à l’aide de SSMS

1. Dans SSMS, effectuez un clic droit sur une base de données > **Restaurer**.

2. Dans la boîte de dialogue **Sauvegarder une base de données**, dans **Fichier de sauvegarde**, cliquez sur **Parcourir**.

3. Dans la boîte de dialogue **Rechercher les fichiers de base de données**, sélectionnez le fichier à restaurer.

4. Dans la boîte de dialogue **Restaurer la base de données**, sélectionnez la base de données.

5. Spécifiez les options. Les options de sécurité doivent correspondre aux options de sauvegarde utilisées lors de la sauvegarde.


### <a name="powershell"></a>PowerShell

Utilisez l’applet de commande [Restore-ASDatabase](/powershell/module/sqlserver/restore-asdatabase).


## <a name="related-information"></a>Informations connexes

[Des comptes de stockage Azure](../storage/common/storage-account-create.md)  
[Haute disponibilité](analysis-services-bcdr.md)      
[FAQ sur la connectivité réseau d’Analysis Services](analysis-services-network-faq.md)