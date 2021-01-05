---
title: Déplacer des ressources Azure SQL entre des régions avec Azure Resource Mover
description: Apprendre à déplacer des ressources Azure SQL dans une autre région à l’aide d’Azure Resource Mover
author: rayne-wiselman
manager: evansma
ms.service: resource-move
ms.topic: tutorial
ms.date: 09/09/2020
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 9fe43125c83436f89bf93cbe975317efec2beb46
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95542811"
---
# <a name="tutorial-move-azure-sql-database-resources-to-another-region"></a>Tutoriel : Déplacer des ressources Azure SQL Database dans une autre région

Dans ce tutoriel, apprenez à déplacer des pools élastiques et des bases de données Azure SQL vers une autre région Azure à l’aide d’[Azure Resource Mover](overview.md).

> [!NOTE]
> Azure Resource Mover est actuellement en préversion.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Vérifier les prérequis et la configuration exigée.
> * Sélectionner les ressources que vous souhaitez déplacer.
> * Résoudre des dépendances de ressources.
> * Préparer et déplacer le serveur SQL Server dans la région cible.
> * Préparer et déplacer des bases de données et des pools élastiques.
> * Décider si vous souhaitez abandonner ou valider le déplacement. 
> * Le cas échéant, supprimer les ressources dans la région source après le déplacement. 

> [!NOTE]
> Les tutoriels enseignent le moyen le plus rapide de tester un scénario et d’utiliser les options par défaut. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer. Puis, connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="prerequisites"></a>Prérequis

-  Veillez à disposer d’un accès *Propriétaire* sur l’abonnement contenant les ressources que vous souhaitez déplacer.
    - La première fois que vous ajoutez une ressource pour une paire source et destination spécifique dans un abonnement Azure, Resource Mover crée une [identité managée affectée par le système](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) (anciennement Managed Service Identify, MSI) qui est approuvée par l’abonnement.
    - Afin de créer l’identité et lui affecter le rôle demandé (Contributeur ou Administrateur de l’accès utilisateur dans l’abonnement source), le compte que vous utilisez pour ajouter des ressources a besoin des autorisations *Propriétaire* sur l’abonnement. [Explorez en détail](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) les rôles Azure.
- L’abonnement a besoin d’un quota suffisant pour créer les ressources que vous déplacez dans la région cible. S’il n’a pas le quota, [demandez des limites supplémentaires](../azure-resource-manager/management/azure-subscription-service-limits.md).
- Vérifiez le tarif et les frais associés à la région cible vers laquelle vous déplacez des ressources. Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour vous aider.
    

## <a name="check-sql-requirements"></a>Vérifier la configuration SQL exigée

1. [Vérifiez](support-matrix-move-region-sql.md) quelles fonctionnalités de pool élastique et de base de données sont prises en charge pour le déplacement dans une autre région.
2. Dans la région cible, créez un serveur cible pour chaque serveur source. [Plus d’informations](../azure-sql/database/active-geo-replication-security-configure.md#how-to-configure-logins-and-users)
4. Si les bases de données sont chiffrées avec Transparent Data Encryption (TDE), et que vous utilisez votre propre clé de chiffrement dans Azure Key Vault, [apprenez à](../key-vault/general/move-region.md) déplacer des coffres de clés dans une autre région.
5. Si la fonctionnalité SQL Data Sync est activée, le déplacement des bases de données membres est pris en charge. Après le déplacement, vous devez configurer SQL Data Sync dans la nouvelle base de données cible.
6. Supprimez les paramètres Advanced Data Security avant le déplacement. Après le déplacement, [configurez les paramètres](../azure-sql/database/azure-defender-for-sql.md) au niveau du serveur SQL Server dans la région cible.
7. Si l’option d’audit est activée, les stratégies sont réinitialisées à la valeur par défaut après le déplacement. [Configurez l’audit](../azure-sql/database/auditing-overview.md) de nouveau, après le déplacement.
7. Les stratégies de rétention des sauvegardes de la base de données source sont transférées à la base de données cible. [Apprenez-en davantage](../azure-sql/database/long-term-backup-retention-configure.md) sur la modification des paramètres après le déplacement.
8. Supprimez les règles de pare-feu au niveau du serveur avant le déplacement. Les règles de pare-feu au niveau de la base de données sont copiées du serveur source vers le serveur cible, pendant le déplacement. Après le déplacement, [configurez des règles de pare-feu](../azure-sql/database/firewall-create-server-level-portal-quickstart.md) pour SQL Server dans la région cible.
9. Supprimez les paramètres de réglage automatique avant le déplacement. [Configurez le réglage automatique](../azure-sql/database/automatic-tuning-enable.md) de nouveau après le déplacement.
10. Supprimez les paramètres d’alerte de base de données avant le déplacement. [Réinitialisez](../azure-sql/database/alerts-insights-configure-portal.md) après le déplacement.
    
## <a name="select-resources"></a>Sélection des ressources

Sélectionnez les ressources que vous souhaitez déplacer.

- Vous pouvez sélectionner n’importe quel type de ressource pris en charge, dans n’importe quel groupe de ressources de la région source sélectionnée.
- Vous déplacez des ressources dans une région cible du même abonnement que la région source. Si vous voulez changer d’abonnement, vous pouvez le faire après le déplacement des ressources.

1. Dans le portail Azure, recherchez *resource mover*. Ensuite, sous **Services**, sélectionnez **Azure Resource Mover**.

     ![Résultats de la recherche de Resource Mover dans le portail Azure](./media/tutorial-move-region-sql/search.png)

2. Dans **Vue d’ensemble**, cliquez sur **Démarrer**.

    ![Bouton pour ajouter des ressources à déplacer dans une autre région](./media/tutorial-move-region-sql/get-started.png)

3. Dans **Déplacer des ressources** > **Source + destination**, sélectionnez l’abonnement et la région sources.
4. Dans **Destination**, sélectionnez la région vers laquelle vous souhaitez déplacer les ressources. Cliquez ensuite sur **Suivant**.

    ![Page de sélection de la région source et de la région de destination](./media/tutorial-move-region-sql/source-target.png)

6. Dans **Ressources à déplacer**, cliquez sur **Sélectionner des ressources**.
7. Dans **Sélectionner des ressources**, sélectionnez les ressources. Vous pouvez uniquement ajouter des ressources prises en charge pour le déplacement. Cliquez ensuite sur **Terminé**.

    ![Page de sélection des ressources SQL à déplacer](./media/tutorial-move-region-sql/select-resources.png)

8. Dans **Ressources à déplacer**, cliquez sur **Suivant**.

9. Dans **Vérifier + ajouter**, contrôlez les paramètres de source et de destination. Confirmez que vous comprenez que les métadonnées relatives au déplacement seront stockées dans un groupe de ressources créé à cette fin dans la région des métadonnées.


    ![Page de vérification des paramètres et de la poursuite du déplacement](./media/tutorial-move-region-sql/review.png)

10. Cliquez sur **Continuer** pour commencer à ajouter les ressources.
11. Une fois le processus d’ajout terminé, cliquez sur **Ajout de ressources pour le déplacement** dans l’icône de notification.
12. Après avoir cliqué sur la notification, passez en revue les ressources sur la page **Entre régions**.


> [!NOTE]
> 
> - Le serveur SQL Server présente désormais l’état *Affectation manuelle en attente*.
> - D’autres ressources ajoutées présentent l’état *Préparation en attente*.
> - Si vous souhaitez supprimer une ressource d’une collection de déplacement, la méthode permettant d’effectuer cette opération dépend de l’étape où vous vous trouvez dans la procédure de déplacement. [Plus d’informations](remove-move-resources.md)

## <a name="resolve-dependencies"></a>Résoudre les erreurs de dépendance


1. Dans **Entre régions**, si des ressources affichent un message *Valider les dépendances* dans la colonne **Problèmes**, cliquez sur le bouton **Valider les dépendances**. Le processus de validation démarre.
2. Si des dépendances sont trouvées, cliquez sur **Ajouter des dépendances**.

    ![Bouton pour ajouter des dépendances](./media/tutorial-move-region-sql/add-dependencies.png)
   
3. Dans **Ajouter des dépendances**, sélectionnez les ressources dépendantes > **Ajouter des dépendances**. Supervisez la progression dans les notifications.

4. Ajoutez des dépendances supplémentaires si nécessaire, puis validez de nouveau les dépendances. 

5. Dans la page **Entre régions**, vérifiez que les ressources présentent maintenant l’état *Préparation en attente*, sans aucun problème répertorié.

    ![Page montrant les ressources avec l’état de préparation en attente](./media/tutorial-move-region-sql/prepare-pending.png)



## <a name="move-the-sql-server"></a>Déplacer le serveur SQL

Affectez un serveur SQL Server cible dans la région cible et validez le déplacement.

### <a name="assign-a-target-sql-server"></a>Affecter un serveur SQL Server cible

1. Dans **Entre régions**, pour la ressource SQL Server, dans la colonne **Configuration de la destination**, cliquez sur **Ressource non affectée**.
2. Sélectionnez une ressource SQL Server existante dans la région cible. 
    
    ![Entrée indiquant l’état du serveur SQL défini sur Validation du déplacement en attente](./media/tutorial-move-region-sql/sql-server-commit-move-pending.png) 

    
> [!NOTE]
> L’état du serveur SQL Server source passe à *Validation du déplacement en attente*. 

### <a name="commit-the-sql-server-move"></a>Valider le déplacement du serveur SQL Server

1. Dans **Entre régions**, sélectionnez le serveur SQL Server, puis cliquez sur **Valider le déplacement**.
2. Dans **Valider les ressources**, cliquez sur **Valider**.

    ![Page de validation du déplacement du serveur SQL Server](./media/tutorial-move-region-sql/commit-sql-server.png)

3. Suivez la progression du déplacement dans la barre de notification.

> [!NOTE]
> Après la validation, le serveur SQL Server présente désormais l’état *Suppression de la source en attente*.


## <a name="prepare-resources-to-move"></a>Préparer les ressources à déplacer

Le serveur SQL Server étant déplacé, vous pouvez préparer le déplacement des autres ressources.

## <a name="prepare-an-elastic-pool"></a>Préparer un pool élastique

1. Dans **Entre régions**, sélectionnez le pool élastique source (demo-test1-elasticpool dans notre procédure pas à pas), puis cliquez sur **Préparer**.

    ![Bouton pour préparer les ressources](./media/tutorial-move-region-sql/prepare-elastic.png)

2. Dans **Préparer les ressources**, cliquez sur **Préparer**.
3. Lorsque les notifications indiquent que le processus de préparation a réussi, cliquez sur **Actualiser**.

> [!NOTE]
> Le pool élastique présente maintenant l’état *Lancement du déplacement en attente*.

## <a name="prepare-a-single-database"></a>Préparer une base de données seule

1. Dans **Entre régions**, sélectionnez la base de données unique (pas dans un pool élastique), puis cliquez sur **Préparer**.

    ![Bouton pour préparer les ressources sélectionnées](./media/tutorial-move-region-sql/prepare-db.png)

2. Dans **Préparer les ressources**, cliquez sur **Préparer**.
3. Lorsque les notifications indiquent que le processus de préparation a réussi, cliquez sur **Actualiser**.

> [!NOTE]
> La base de données présente maintenant l’état *Lancement du déplacement en attente*, elle a été créée dans la région cible.


## <a name="move-the-pool-and-prepare-pool-databases"></a>Déplacer le pool et préparer les bases de données du pool

Pour préparer les bases de données dans un pool élastique, ce pool élastique doit présenter l’état *Validation du déplacement en attente*. Pour passer à cet état, lancez le déplacement du pool.

#### <a name="initiate-move---elastic-pool"></a>Lancer le déplacement du pool élastique

1. Dans **Entre régions**, sélectionnez le pool élastique source (demo-test1-elasticpool dans notre procédure pas à pas), puis cliquez sur **Lancer le déplacement**.
2. Dans **Déplacer des ressources**, cliquez sur **Lancer le déplacement**.

    
    ![Bouton pour initier le déplacement d’un pool élastique](./media/tutorial-move-region-sql/initiate-elastic.png)

1. Suivez la progression du déplacement dans la barre de notification.
1. Lorsque les notifications indiquent que le processus de déplacement a réussi, cliquez sur **Actualiser**.

> [!NOTE]
> Le pool élastique présente maintenant l’état *Validation du déplacement en attente*.

#### <a name="prepare-database"></a>Préparer une base de données

1. Dans **Entre régions**, sélectionnez la base de données (demo-test2-sqldb dans notre procédure pas à pas), puis cliquez sur **Préparer**.
2. Dans **Préparer les ressources**, cliquez sur **Préparer**.

    ![Bouton pour préparer une base de données dans un pool élastique](./media/tutorial-move-region-sql/prepare-database-elastic.png) 

Pendant la préparation, la base de données cible est créée dans la région cible, et la réplication des données démarre. Après la préparation, la base de données présente l’état *Lancement du déplacement en attente*. 

![Bouton pour préparer la base de données sélectionnée dans le pool élastique](./media/tutorial-move-region-sql/initiate-move-pending.png) 

## <a name="move-databases"></a>Déplacer des bases de données

Commencez à déplacer les bases de données.
1. Dans **Entre régions**, sélectionnez les ressources avec l’état **Lancement du déplacement en attente**. Cliquez ensuite sur **Lancer le déplacement**.
2. Dans **Déplacer des ressources**, cliquez sur **Lancer le déplacement**.

    ![Page de lancement du déplacement](./media/tutorial-move-region-sql/initiate-move.png)

3. Suivez la progression du déplacement dans la barre de notification.

> [!NOTE]
> Les bases de données présentent maintenant l’état *Validation du déplacement en attente*.


## <a name="discard-or-commit"></a>Abandonner ou valider ?

Après le déplacement initial, vous pouvez décider si vous souhaitez valider le déplacement ou l’abandonner. 

- **Abandonner** : vous pouvez abandonner un déplacement si vous effectuez un test et que vous ne souhaitez pas déplacer réellement la ressource source. Si vous abandonnez le déplacement, la ressource reviendra à l’état **Lancement du déplacement en attente**.
- **Valider** : La validation termine le déplacement vers la région cible. Après sa validation, une ressource source se présente avec l’état de **Suppression de la source en attente**, et vous pouvez décider si vous voulez la supprimer.


## <a name="discard-the-move"></a>Abandonner le déplacement 

Vous pouvez abandonner le déplacement de la manière suivante :

1. Dans **Entre régions**, sélectionnez les ressources avec l’état **Validation du déplacement en attente**, puis cliquez sur **Abandonner le déplacement**.
2. Dans **Abandonner le déplacement**, cliquez sur **Abandonner**.
3. Suivez la progression du déplacement dans la barre de notification.


> [!NOTE]
> - Après l’abandon des ressources, celles-ci présentent l’état *Lancement du déplacement en attente*.
> - S’il n’y a qu’un pool élastique, abandonnez les progressions pour que le pool élastique créé dans la région cible soit supprimé.
> - S’il existe un pool élastique avec des bases de données associées dans l’état *Validation du déplacement en attente*, vous ne pouvez pas abandonner le pool élastique.
> - Si vous abandonnez une base de données SQL, les ressources de la région cible ne sont pas supprimées. 

Si vous voulez recommencer le déplacement après l’avoir abandonné, sélectionnez la base de données SQL ou le pool élastique, puis relancez le déplacement.


## <a name="commit-the-move"></a>Valider le déplacement

Achevez le déplacement des bases de données et des pools élastiques de la manière suivante :


1. Vérifiez que le serveur SQL Server présente l’état *Suppression de la source en attente*.
2. Mettez à jour les chaînes de connexion de la base de données vers la région cible avant de valider.
3. Dans **Entre régions**, sélectionnez les ressources SQL, puis cliquez sur **Valider le déplacement**.
4. Dans **Valider les ressources**, cliquez sur **Valider**.

    ![Valider le déplacement](./media/tutorial-move-region-sql/commit-sql-resources.png)

5. Suivez la progression de la validation dans la barre de notification.


> [!NOTE]
> Un temps d’arrêt se produit pour les bases de données SQL pendant le processus de validation.
> Les bases de données et les pools élastiques validés présentent désormais l’état *Suppression de la source en attente*.
> Après la validation, mettez à jour les paramètres liés à la base de données, y compris les règles de pare-feu, les stratégies et les alertes, sur la base de données cible.


## <a name="delete-source-resources-after-commit"></a>Supprimer les ressources sources après la validation

Après le déplacement, vous pouvez, si vous le souhaitez, supprimer les ressources dans la région source. 

1. Dans **Entre régions**, cliquez sur le nom de chaque ressource source que vous décidez de supprimer.
2. Dans la page des propriétés de chaque ressource, sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Avez déplacé des bases de données Azure SQL dans une autre région Azure.
> * Avez déplacé des pools élastiques Azure SQL dans une autre région.

À présent, vous allez vous intéresser au déplacement de machines virtuelles dans une autre région.

> [!div class="nextstepaction"]
> [Déplacer des machines virtuelles Azure](./tutorial-move-region-virtual-machines.md)