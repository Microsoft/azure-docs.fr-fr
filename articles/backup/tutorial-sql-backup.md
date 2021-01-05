---
title: Tutoriel - Sauvegarder des bases de données SQL Server sur Azure
description: Dans ce tutoriel, découvrez comment sauvegarder, dans un coffre Recovery Services de la sauvegarde Azure, une base de données SQL Server s’exécutant sur une machine virtuelle Azure.
ms.topic: tutorial
ms.date: 06/18/2019
ms.openlocfilehash: 17a8472da2595c08cb198baaf853faf110a619fa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88612450"
---
# <a name="back-up-a-sql-server-database-in-an-azure-vm"></a>Sauvegarder une base de données SQL Server dans une machine virtuelle Azure

Ce tutoriel vous explique comment sauvegarder, dans un coffre Recovery Services de la sauvegarde Azure, une base de données SQL Server s’exécutant sur une machine virtuelle Azure. Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Créer et configurer un coffre
> * Détecter des bases de données et configurer des sauvegardes
> * Configurer la protection automatique de bases de données
> * Exécutez une sauvegarde à la demande.

## <a name="prerequisites"></a>Prérequis

Pour pouvoir sauvegarder votre base de données SQL Server, vérifiez les conditions suivantes :

1. Identifiez ou [créez](backup-sql-server-database-azure-vms.md#create-a-recovery-services-vault) un coffre Recovery Services dans la même région ou avec les mêmes paramètres régionaux que la machine virtuelle qui héberge l’instance SQL Server.
2. [Vérifiez les autorisations de la machine virtuelle](backup-azure-sql-database.md#set-vm-permissions) qui sont nécessaires pour sauvegarder les bases de données SQL.
3. Vérifiez que la machine virtuelle dispose d’une [connectivité réseau](backup-sql-server-database-azure-vms.md#establish-network-connectivity).
4. Vérifiez que les bases de données SQL Server sont nommées conformément aux [instructions de nommage](#verify-database-naming-guidelines-for-azure-backup) pour la sauvegarde Azure.
5. Vérifiez que vous n’avez aucune autre solution de sauvegarde activée pour la base de données. Désactivez tous les autres sauvegardes SQL Server avant de configurer ce scénario. Vous pouvez activer la Sauvegarde Azure pour une machine virtuelle Azure en même temps que la Sauvegarde Azure pour une base de données SQL Server s’exécutant sur la machine virtuelle, sans aucun conflit.

### <a name="establish-network-connectivity"></a>Établir la connectivité réseau

Pour toute opération, la machine virtuelle SQL Server a besoin d’une connectivité pour accéder aux adresses IP publiques Azure. Les opérations de machine virtuelle (détection de bases de données, configuration de sauvegardes, sauvegardes planifiées, restauration des points de récupération, etc.) échouent en cas d’absence de connexion aux adresses IP publiques. Établissez une connexion à l’aide d’une des options suivantes :

* **Autoriser les plages d’adresses IP du centre de données Azure** : autorisez les [plages d’adresses IP](https://www.microsoft.com/download/details.aspx?id=41653) dans le téléchargement. Pour accéder à un groupe de sécurité réseau, utilisez l’applet de commande **Set-AzureNetworkSecurityRule**.
* **Déployer un serveur proxy HTTP pour le routage du trafic** : lorsque vous sauvegardez une base de données SQL Server sur une machine virtuelle Azure, l’extension de sauvegarde sur la machine virtuelle utilise les API HTTPS pour envoyer des commandes de gestion à la sauvegarde Azure, et des données au stockage Azure. L’extension de sauvegarde utilise également Azure Active Directory (Azure AD) pour l’authentification. Acheminez le trafic de l’extension de sauvegarde pour ces trois services via le proxy HTTP. Les extensions sont le seul composant configuré pour l’accès à l’internet public.

Chaque option présente des avantages et des inconvénients

**Option** | **Avantages** | **Inconvénients**
--- | --- | ---
Autoriser les plages d’adresses IP | Aucun coût supplémentaire | Difficile à gérer, car les plages d’adresses IP changent au fil du temps. <br/><br/> Fournit un accès à l’ensemble d’Azure et pas seulement au stockage Azure.
Utiliser un proxy HTTP   | Le contrôle granulaire dans le proxy sur les URL de stockage est autorisé. <br/><br/> Un seul point d’accès Internet aux machines virtuelles. <br/><br/> Non soumis aux modifications d’adresse IP Azure. | Frais supplémentaires d’exécution de machine virtuelle avec le logiciel de serveur proxy.

### <a name="set-vm-permissions"></a>Définir des autorisations de machine virtuelle

La sauvegarde Azure effectue un certain nombre de choses lorsque vous configurez la sauvegarde pour une base de données SQL Server :

* Elle ajoute l’extension **AzureBackupWindowsWorkload**.
* Lors de la découverte des bases de données sur la machine virtuelle, la Sauvegarde Azure crée le compte **NT SERVICE\AzureWLBackupPluginSvc**. Ce compte est utilisé pour la sauvegarde et la restauration, il doit disposer d’autorisations sysadmin SQL.
* La sauvegarde Azure utilise le compte **NT AUTHORITY\SYSTEM** pour la détection et l’interrogation des bases de données. Ce compte doit donc être une connexion publique sur SQL.

Si vous n’avez pas créé la machine virtuelle SQL Server à partir de la Place de marché Azure, vous pouvez recevoir une erreur **UserErrorSQLNoSysadminMembership**. Si cela se produit, [suivez ces instructions](backup-azure-sql-database.md#set-vm-permissions).

### <a name="verify-database-naming-guidelines-for-azure-backup"></a>Vérifier les instructions de nommage des bases de données pour la sauvegarde Azure

Évitez les éléments suivants pour les noms de base de données :

* Espaces au début ou à la fin
* « ! » à la fin
* Crochets de fermeture « ] »
* Noms de bases de données commençant par « F:\ »

Il existe bien des alias pour les caractères non pris en charge dans la table Azure, mais nous vous recommandons de les éviter. [Plus d’informations](/rest/api/storageservices/understanding-the-table-service-data-model)

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-sql-server-databases"></a>Détecter les bases de données SQL Server

Détectez les bases de données en cours d’exécution sur la machine virtuelle.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez le coffre Recovery Services que vous utilisez pour sauvegarder la base de données.

2. Dans le tableau de bord du **coffre Recovery Services**, cliquez sur **Sauvegarde**.

   ![Cliquer sur Sauvegarder pour ouvrir le menu Objectif de sauvegarde](./media/backup-azure-sql-database/open-backup-menu.png)

3. Dans **Objectif de sauvegarde**, définissez **Où s’exécute votre charge de travail ?** sur **Azure** (la valeur par défaut).

4. Dans **Que souhaitez-vous sauvegarder ?** , sélectionnez **SQL Server dans une machine virtuelle Azure**.

    ![Sélectionnez SQL Server dans une machine virtuelle Azure pour la sauvegarde](./media/backup-azure-sql-database/choose-sql-database-backup-goal.png)

5. Dans **Objectif de sauvegarde** > **Découvrir les bases de données dans les machines virtuelles**, sélectionnez **Démarrer la détection** pour rechercher des machines virtuelles non protégées dans l’abonnement. La durée de cette opération varie selon le nombre de machines virtuelles non protégées de l’abonnement.

   * Après la détection, les machines virtuelles non protégées doivent apparaître dans la liste, répertoriées par nom et groupe de ressources.
   * Si une machine virtuelle n’est pas répertoriée contrairement à ce que vous attendez, vérifiez si elle n’est pas déjà sauvegardée dans un coffre.
   * Plusieurs machines virtuelles peuvent avoir le même nom, mais appartenir à différents groupes de ressources.

     ![La sauvegarde est en attente au cours de la recherche pour des bases de données dans des machines virtuelles](./media/backup-azure-sql-database/discovering-sql-databases.png)

6. Dans la liste des machines virtuelles, sélectionnez la machine virtuelle exécutant la base de données SQL Server > **Découvrir les bases de données**.

7. Suivez la découverte des bases de données dans la zone **Notifications**. L’exécution du travail peut prendre un certain temps, et varie en fonction du nombre de bases de données résidant sur la machine virtuelle. Lorsque les bases de données sélectionnées ont été détectées, un message de réussite s’affiche.

    ![Message Déploiement réussi](./media/backup-azure-sql-database/notifications-db-discovered.png)

8. La sauvegarde Azure détecte toutes les bases de données SQL Server résidant sur la machine virtuelle. Lors de la découverte, les événements suivants se produisent en arrière-plan :

    * La sauvegarde Azure inscrit la machine virtuelle auprès du coffre pour la sauvegarde de la charge de travail. Les bases de données présentes sur la machine virtuelle inscrite ne peuvent être sauvegardées que sur ce coffre.
    * La sauvegarde Azure installe l’extension **AzureBackupWindowsWorkload** sur la machine virtuelle. Aucun agent n’est installé sur la base de données SQL.
    * La sauvegarde Azure crée le compte de service **NT Service\AzureWLBackupPluginSvc** sur la machine virtuelle.
      * Toutes les opérations de sauvegarde et de restauration utilisent le compte de service.
      * **NT Service\AzureWLBackupPluginSvc** nécessite des autorisations d’administrateur système SQL. **SqlIaaSExtension** est installé sur toutes les machines virtuelles SQL Server créées dans la Place de marché Azure. L’extension **AzureBackupWindowsWorkload** utilise l’extension **SQLIaaSExtension** pour obtenir automatiquement les autorisations requises.
    * Si vous n’avez pas créé la machine virtuelle à partir de la Place de marché, **SqlIaaSExtension** n’est pas installé sur la machine virtuelle, et l’opération de découverte échoue avec le message d’erreur **UserErrorSQLNoSysAdminMembership**. Suivez les [instructions](backup-azure-sql-database.md#set-vm-permissions) pour résoudre ce problème.

        ![Sélectionner la machine virtuelle et la base de données](./media/backup-azure-sql-database/registration-errors.png)

## <a name="configure-backup"></a>Configurer une sauvegarde  

Configurez la sauvegarde de la façon suivante :

1. Dans le menu **Objectif de sauvegarde**, sélectionnez **Configurer la sauvegarde**.

   ![Sélectionner Configurer la sauvegarde](./media/backup-azure-sql-database/backup-goal-configure-backup.png)

2. Sélectionnez **Configurer la sauvegarde** : le volet **Sélectionner les éléments à sauvegarder** apparaît. Cette liste répertorie tous les groupes de disponibilité enregistrés et serveurs SQL autonomes. Développez le chevron situé à gauche de la ligne pour voir toutes les bases de données non protégées de cette instance ou Groupe de disponibilité Always On.  

    ![Affichage de toutes les instances SQL Server avec des bases de données autonomes](./media/backup-azure-sql-database/list-of-sql-databases.png)

3. Sélectionnez toutes les bases de données que vous souhaitez protéger > **OK**.

   ![Protection de la base de données](./media/backup-azure-sql-database/select-database-to-protect.png)

   Pour optimiser les charges de sauvegarde, la sauvegarde Azure définit le nombre maximal de bases de données à 50 dans une tâche de sauvegarde.

     * Vous pouvez aussi activer la protection automatique sur l’ensemble de l’instance ou du groupe de disponibilité Always On en sélectionnant l’option **ACTIVÉ** dans la liste déroulante correspondant à la colonne **PROTECTION AUTOMATIQUE**. La fonctionnalité de protection automatique permet non seulement de protéger toutes les bases de données existantes en une seule étape, mais aussi de protéger automatiquement toutes les nouvelles bases de données qui seront ajoutées à cette instance ou ce groupe de disponibilité par la suite.  

4. Sélectionnez **OK** pour créer le volet **Stratégie de sauvegarde**.

    ![Activer la protection automatique sur le groupe de disponibilité Always On](./media/backup-azure-sql-database/enable-auto-protection.png)

5. Dans **Choisir une stratégie de sauvegarde**, sélectionnez une stratégie, puis **OK**.

   * Sélectionner la stratégie par défaut : HourlyLogBackup.
   * Choisir une stratégie de sauvegarde existante créée précédemment pour SQL.
   * Définir une nouvelle stratégie selon votre RPO et la durée de rétention.

     ![Sélectionner la stratégie de sauvegarde](./media/backup-azure-sql-database/select-backup-policy.png)

6. Dans le menu **Sauvegarder**, sélectionnez **Activer la sauvegarde**.

    ![Activer la stratégie de sauvegarde choisie](./media/backup-azure-sql-database/enable-backup-button.png)

7. Vous pouvez suivre la progression de la configuration dans la zone **Notifications** du portail.

    ![Zone Notifications](./media/backup-azure-sql-database/notifications-area.png)

### <a name="create-a-backup-policy"></a>Créer une stratégie de sauvegarde

Une stratégie de sauvegarde définit le moment auquel les sauvegardes sont effectuées ainsi que leur durée de rétention.

* Une stratégie est créée au niveau du coffre.
* Plusieurs coffres peuvent utiliser la même stratégie de sauvegarde, mais vous devez appliquer la stratégie de sauvegarde à chaque coffre.
* Lorsque vous créez une stratégie de sauvegarde, une sauvegarde complète quotidienne est la valeur par défaut.
* Vous pouvez ajouter une sauvegarde différentielle, mais uniquement si vous configurez les sauvegardes complètes pour qu’elles aient lieu toutes les semaines.
* [Apprenez-en davantage](backup-architecture.md#sql-server-backup-types) sur les différents types de stratégies de sauvegarde.

Pour créer une stratégie de sauvegarde :

1. Dans le coffre, sélectionnez **Stratégies de sauvegarde** > **Ajouter**.
2. Dans le menu **Ajouter**, sélectionnez **SQL Server dans une machine virtuelle Azure** pour définir le type de stratégie.

   ![Choisissez un type de stratégie pour la nouvelle stratégie de sauvegarde](./media/backup-azure-sql-database/policy-type-details.png)

3. Dans **Nom de la stratégie**, entrez le nom de la nouvelle stratégie.
4. Dans **Stratégie de sauvegarde complète**, comme **Fréquence de sauvegarde**, sélectionnez **Tous les jours** ou **Toutes les semaines**.

   * Si vous sélectionnez **quotidienne**, sélectionnez l’heure et le fuseau horaire de début du travail de sauvegarde.
   * Vous devez exécuter une sauvegarde complète, car vous ne pouvez pas désactiver l’option **Sauvegarde complète**.
   * Sélectionnez **Sauvegarde complète** pour afficher la stratégie.
   * Si vous choisissez des sauvegardes complètes quotidiennes, vous ne pouvez pas créer de sauvegardes différentielles.
   * Si vous choisissez la fréquence **hebdomadaire**, sélectionnez le jour de la semaine, l’heure et le fuseau horaire indiquant le début du travail de sauvegarde.

     ![Champs de la nouvelle stratégie de sauvegarde](./media/backup-azure-sql-database/full-backup-policy.png)  

5. Pour la **Durée de rétention**, par défaut toutes les options sont sélectionnées. Désactivez les limites des plages de rétention dont vous ne souhaitez pas vous servir, puis définissez les intervalles à utiliser.

    * La période de rétention minimale est de sept jours pour tous les types de sauvegardes (complète/différentielle/fichier journal).
    * Des points de récupération sont marqués pour la rétention et varient selon la durée de rétention. Par exemple, si vous sélectionnez une sauvegarde complète quotidienne, seule une sauvegarde complète est déclenchée chaque jour.
    * La sauvegarde d’un jour spécifique est marquée et conservée conformément à la durée et aux paramètres de rétention hebdomadaire.
    * Les durées de rétention mensuelle et annuelle ont le même comportement.

   ![Paramètres d’intervalle de la durée de rétention](./media/backup-azure-sql-database/retention-range-interval.png)

6. Dans le menu de **stratégie Sauvegarde complète**, cliquez sur **OK** pour accepter les paramètres.
7. Pour ajouter une stratégie de sauvegarde différentielle, sélectionnez **Sauvegarde différentielle**.

   ![Paramètres d’intervalle des durées de rétention](./media/backup-azure-sql-database/retention-range-interval.png)
   ![Ouvrir le menu de la stratégie de sauvegarde différentielle](./media/backup-azure-sql-database/backup-policy-menu-choices.png)

8. Dans la stratégie **Sauvegarde différentielle**, sélectionnez **Activer** pour ouvrir les contrôles de fréquence et de rétention.

    * Vous pouvez déclencher au plus une sauvegarde différentielle par jour.
    * Les sauvegardes différentielles peuvent être conservées jusqu’à 180 jours. Si vous avez besoin d’une durée de rétention supérieure, vous devez utiliser des sauvegardes complètes.

9. Sélectionnez **OK** pour enregistrer la stratégie et revenir au menu principal **Stratégie de sauvegarde**.

10. Pour ajouter une stratégie de sauvegarde de fichier journal, cliquez sur **Sauvegarde de fichier journal**.
11. Dans **Sauvegarde de fichier journal**, sélectionnez **Activer** et définissez les contrôles de fréquence et de rétention. Les sauvegardes de fichiers journaux peuvent se produire toutes les 15 minutes et être conservées jusqu’à 35 jours.
12. Sélectionnez **OK** pour enregistrer la stratégie et revenir au menu principal **Stratégie de sauvegarde**.

    ![Modifier la stratégie de sauvegarde de fichier journal](./media/backup-azure-sql-database/log-backup-policy-editor.png)

13. Dans le menu **Stratégie de sauvegarde**, choisissez s’il convient d’activer l’option **Compression de la sauvegarde SQL**,
    * qui est désactivée par défaut.
    * Sur le serveur principal, Sauvegarde Azure utilise la compression de sauvegarde native SQL.

14. Après avoir terminé les modifications apportées à la stratégie de sauvegarde, sélectionnez **OK**.

## <a name="run-an-on-demand-backup"></a>Exécuter une sauvegarde à la demande

1. Dans le coffre Recovery Services, choisissez Éléments de la sauvegarde.
2. Sélectionnez « SQL dans la machine virtuelle Azure ».
3. Cliquez avec le bouton droit sur une base de données et choisissez « Sauvegarder maintenant ».
4. Choisissez le Type de sauvegarde (Complète/Différentielle/Journal/Copie complète uniquement) et la Compression (Activer/Désactiver)
5. Sélectionnez OK pour commencer la sauvegarde.
6. Surveillez le travail de sauvegarde en vous rendant dans votre coffre Recovery Services et en choisissant « Travaux de sauvegarde ».

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé le portail Azure pour effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Créer et configurer un coffre
> * Détecter des bases de données et configurer des sauvegardes
> * Configurer la protection automatique de bases de données
> * Exécutez une sauvegarde à la demande.

Passez au didacticiel suivant pour restaurer une machine virtuelle Azure à partir d’un disque.

> [!div class="nextstepaction"]
> [Restaurer des bases de données SQL Server sur des machines virtuelles Azure](./restore-sql-database-azure-vm.md)
