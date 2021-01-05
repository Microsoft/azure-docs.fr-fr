---
title: Résoudre les problèmes de SQL Data Sync
description: Découvrez comment identifier et résoudre les problèmes courants liés à SQL Data Sync dans Azure.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: data sync, sqldbrb=1
ms.devlang: ''
ms.topic: troubleshooting
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/20/2018
ms.openlocfilehash: 02eaec4c86c934e8d2638de1b60aa9267babf7a8
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790166"
---
# <a name="troubleshoot-issues-with-sql-data-sync"></a>Résoudre les problèmes liés à SQL Data Sync
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article explique comment résoudre les problèmes connus liés à SQL Data Sync dans Azure. S’il existe un moyen de résoudre un problème, celui-ci est décrit ici.

Pour obtenir une vue d’ensemble de SQL Data Sync, consultez [Synchroniser des données entre plusieurs bases de données cloud et locales avec SQL Data Sync dans Azure](sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> Pour le moment, SQL Data Sync **ne prend pas en charge** Azure SQL Managed Instance.

## <a name="sync-issues"></a>Problèmes de synchronisation

- [La synchronisation de bases de données locales associées à l’agent client échoue dans l’interface utilisateur du portail](#sync-fails)

- [Mon groupe de synchronisation est bloqué à l’état de traitement](#sync-stuck)

- [Mes tables contiennent des données erronées](#sync-baddata)

- [Des données de clé primaire incohérentes s’affichent après une synchronisation réussie](#sync-pkdata)

- [Je constate une dégradation significative des performances](#sync-perf)

- [Je vois ce message : « Impossible d’insérer la valeur NULL dans la colonne \<column>. Cette colonne n’accepte pas les valeurs NULL. » Que signifie cette erreur et comment puis-je la corriger ?](#sync-nulls)

- [Comment Data Sync traite-t-il les références circulaires ? Autrement dit, lorsque les mêmes données sont synchronisées dans plusieurs groupes de synchronisation et changent constamment en conséquence.](#sync-circ)

### <a name="sync-fails-in-the-portal-ui-for-on-premises-databases-that-are-associated-with-the-client-agent"></a><a name="sync-fails"></a> La synchronisation des bases de données locales associées à l’agent client échoue dans l’interface utilisateur du portail.

La synchronisation des bases de données locales associées à l’agent client échoue dans l’interface utilisateur du portail SQL Data Sync. Sur l’ordinateur local qui exécute l’agent, le journal des événements affiche des erreurs System.IO.IOException. Ces erreurs indiquent que l’espace disponible sur le disque est insuffisant.

- **Cause** . L’espace dont dispose le lecteur est insuffisant.

- **Résolution** . Libérez de l’espace sur le lecteur où se trouve le répertoire %TEMP%.

### <a name="my-sync-group-is-stuck-in-the-processing-state"></a><a name="sync-stuck"></a> Mon groupe de synchronisation est bloqué à l’état de traitement

Un groupe de synchronisation de SQL Data Sync est en cours de traitement depuis un certain temps. Il ne répond pas à la commande d’ **arrêt** et aucune nouvelle entrée n’apparaît dans les journaux d’activité.

L’une des conditions suivantes peut provoquer le blocage d’un groupe de synchronisation sur l’état de traitement :

- **Cause** . L’agent client est hors connexion.

- **Résolution** . Vérifiez que l’agent client est en ligne, puis réessayez.

- **Cause** . L’agent client est désinstallé ou manquant

- **Résolution** . Si l’agent client est désinstallé ou manquant :

    1. Accédez au dossier d’installation de SQL Data Sync et supprimez le fichier XML de l’agent, si ce fichier existe.
    1. Installez l’agent sur un ordinateur local (il peut s’agir du même ordinateur ou d’un autre ordinateur). Envoyez ensuite la clé générée dans le portail pour l’agent qui apparaît comme étant hors connexion.

- **Cause** . Le service SQL Data Sync est arrêté.

- **Résolution** . Redémarrez le service SQL Data Sync.

    1. Dans le menu **Démarrer** , recherchez **Services** .
    1. Dans les résultats de la recherche, sélectionnez **Services** .
    1. Trouvez le service **SQL Data Sync** .
    1. Si l’état du service est **Arrêté** , cliquez avec le bouton droit sur le nom du service, puis sélectionnez **Démarrer** .

> [!NOTE]
> Si les informations précédentes ne permettent pas à votre groupe de synchronisation de quitter l’état de traitement, le Support Microsoft peut réinitialiser son état. Pour réinitialiser l’état de votre groupe de synchronisation, dans la [page de questions Microsoft Q&A sur Azure SQL Database](/answers/topics/azure-sql-database.html), créez un post. Dans le billet, indiquez votre ID d’abonnement et l’ID du groupe de synchronisation à réinitialiser. Un ingénieur du Support Microsoft répondra à votre billet et vous préviendra dès que l’état aura été réinitialisé.

### <a name="i-see-erroneous-data-in-my-tables"></a><a name="sync-baddata"></a> Mes tables contiennent des données erronées.

Si des tables portant le même nom mais provenant de schémas de base de données différents sont incluses dans une synchronisation, des données erronées apparaissent dans les tables à l’issue de la synchronisation.

- **Cause** . Le processus de déploiement de SQL Data Sync utilise les mêmes tables de suivi pour les tables qui portent le même nom mais proviennent de schémas différents. Par conséquent, les modifications des deux tables sont répercutées dans la même table de suivi. Et cela génère des modifications de données erronées pendant la synchronisation.

- **Résolution** . Assurez-vous que les noms des tables impliquées dans une synchronisation sont différents, même si elles appartiennent à des schémas de base de données différents.

### <a name="i-see-inconsistent-primary-key-data-after-a-successful-sync"></a><a name="sync-pkdata"></a> Des données de clé primaire incohérentes s’affichent après une synchronisation réussie

Une synchronisation est signalée comme réussie, et le journal n’affiche aucune ligne Échec ou Ignoré, mais vous constatez la présence de données de clé primaire incohérentes dans les bases de données du groupe de synchronisation.

- **Cause** . Ce résultat est lié à la conception. Toute modification d’une colonne de clé primaire génère des données incohérentes sur les lignes où la clé primaire a été modifiée.

- **Résolution** . Pour éviter ce problème, vérifiez qu’aucune donnée dans une colonne de clé primaire n’est modifiée. Pour résoudre ce problème, supprimez la ligne contenant les données incohérentes sur tous les points de terminaison du groupe de synchronisation. Puis réinsérez la ligne.

### <a name="i-see-a-significant-degradation-in-performance"></a><a name="sync-perf"></a> Je constate une dégradation significative des performances.

Les performances se dégradent considérablement, jusqu’à ce que vous ne soyez même plus en mesure d’ouvrir l’interface utilisateur de Data Sync.

- **Cause** . Le problème est probablement lié à une boucle de synchronisation. Une boucle de synchronisation se produit lorsqu’une synchronisation initiée par un groupe de synchronisation A déclenche une synchronisation par un groupe de synchronisation B, qui déclenche à son tour une synchronisation par le groupe de synchronisation A. La réalité est parfois plus complexe, puisque plus de deux groupes de synchronisation peuvent être impliqués dans la boucle. Le problème est lié au déclenchement circulaire de la synchronisation, lui-même provoqué par le chevauchement des groupes de synchronisation.

- **Résolution** . Le meilleur correctif est la prévention. Vérifiez l’absence de références circulaires dans vos groupes de synchronisation. Lorsqu’une ligne est synchronisée par un groupe de synchronisation, elle ne peut pas l’être par un autre groupe de synchronisation.

### <a name="i-see-this-message-cannot-insert-the-value-null-into-the-column-column-column-does-not-allow-nulls-what-does-this-mean-and-how-can-i-fix-it"></a><a name="sync-nulls"></a> Je vois ce message : « Impossible d’insérer la valeur NULL dans la colonne \<column>. Cette colonne n’accepte pas les valeurs NULL. » Que signifie cette erreur et comment puis-je la corriger ? 
Ce message d’erreur indique que l’un des deux problèmes suivants est survenu :
-  Une table ne possède pas de clé primaire. Pour résoudre ce problème, ajoutez une clé primaire à toutes les tables que vous synchronisez.
-  Votre instruction CREATE INDEX contient une clause WHERE. Data Sync ne traite pas cette condition. Pour résoudre ce problème, supprimez la clause WHERE ou apportez manuellement les modifications à toutes les bases de données. 
 
### <a name="how-does-data-sync-handle-circular-references-that-is-when-the-same-data-is-synced-in-multiple-sync-groups-and-keeps-changing-as-a-result"></a><a name="sync-circ"></a> Comment Data Sync traite-t-il les références circulaires ? Autrement dit, lorsque les mêmes données sont synchronisées dans plusieurs groupes de synchronisation et changent constamment en conséquence ?
Data Sync ne traite pas les références circulaires. Veillez à les éviter. 

## <a name="client-agent-issues"></a>Problèmes liés à l’agent client

Pour résoudre les problèmes liés à l'agent client, consultez [Résoudre les problèmes liés à l'agent de synchronisation des données](sql-data-sync-agent-overview.md#agent-tshoot).

## <a name="setup-and-maintenance-issues"></a>Problèmes d’installation et de maintenance

- [Je reçois un message « Espace disque insuffisant ».](#setup-space)

- [Je ne parviens pas à supprimer mon groupe de synchronisation.](#setup-delete)

- [Je ne peux pas annuler l’inscription d’une base de données SQL Server](#setup-unreg).

- [Je ne dispose pas des privilèges suffisants pour démarrer les services système.](#setup-perms)

- [Une base de données a un état « Obsolète ».](#setup-date)

- [Un groupe de synchronisation a un état « Obsolète ».](#setup-date2)

- [Impossible de supprimer un groupe de synchronisation dans les trois minutes qui suivent la désinstallation ou l’arrêt de l’agent.](#setup-delete2)

- [Que se passe-t-il quand je restaure une base de données perdue ou endommagée ?](#setup-restore)

### <a name="i-get-a-disk-out-of-space-message"></a><a name="setup-space"></a>Je reçois un message « Espace disque insuffisant ».

- **Cause** . Le message « Espace disque insuffisant » peut s’afficher s’il reste des fichiers à supprimer. Ce problème peut être dû à un logiciel antivirus. Il peut également survenir lorsque des fichiers sont ouverts pendant une tentative de suppression.

- **Résolution** . Supprimez manuellement les fichiers de synchronisation situés dans le dossier %Temp% (`del \*sync\* /s`). Puis supprimez les sous-répertoires du dossier %Temp%.

> [!IMPORTANT]
> Ne supprimez aucun fichier tant que la synchronisation est en cours.

### <a name="i-cant-delete-my-sync-group"></a><a name="setup-delete"></a> Je ne parviens pas à supprimer mon groupe de synchronisation.

Votre tentative de suppression d’un groupe de synchronisation échoue. La suppression d’un groupe de synchronisation peut échouer dans les situations suivantes :

- **Cause** . L’agent client est hors connexion

- **Résolution** . Vérifiez que l’agent client est en ligne, puis réessayez.

- **Cause** . L’agent client est désinstallé ou manquant

- **Résolution** . Si l’agent client est désinstallé ou manquant :  
    a. Accédez au dossier d’installation de SQL Data Sync et supprimez le fichier XML de l’agent, si ce fichier existe.  
    b. Installez l’agent sur un ordinateur local (il peut s’agir du même ordinateur ou d’un autre ordinateur). Envoyez ensuite la clé générée dans le portail pour l’agent qui apparaît comme étant hors connexion.

- **Cause** . Une base de données est hors connexion

- **Résolution** . Vérifiez que vos bases de données sont toutes en ligne.

- **Cause** . Le groupe de synchronisation est en cours de déploiement ou de synchronisation.

- **Résolution** . Attendez que le processus de déploiement ou de synchronisation se termine, puis essayez à nouveau de supprimer le groupe de synchronisation.

### <a name="i-cant-unregister-a-sql-server-database"></a><a name="setup-unreg"></a> Je ne peux pas annuler l’inscription d’une base de données SQL Server.

- **Cause** . Vous essayez probablement d’annuler l’inscription d’une base de données qui a déjà été supprimée.

- **Résolution** . Pour annuler l’inscription d’une base de données SQL Server, sélectionnez la base de données, puis sélectionnez **Forcer la suppression** .

  Si cette opération ne permet pas de supprimer la base de données du groupe de synchronisation :

  1. Arrêtez le service hôte de l’agent client, puis redémarrez-le :  
    a. Sélectionnez le menu **Démarrer** .  
    b. Dans la zone de recherche, entrez **services.msc** .  
    c. Dans la section **Programmes** du volet des résultats de la recherche, double-cliquez sur **Services** .  
    d. Faites un clic droit sur le service **SQL Data Sync** .  
    e. Si le service est en cours d’exécution, arrêtez-le.  
    f. Cliquez avec le bouton droit sur le service, puis sélectionnez **Démarrer** .  
    g. Vérifiez si la base de données est toujours inscrite. Si elle n’est plus inscrite, vous avez terminé. Sinon, passez à l’étape suivante.
  1. Ouvrez l’application de l’agent client (SqlAzureDataSyncAgent).
  1. Sélectionnez **Modifier les informations d’identification** , puis entrez les informations d’identification de la base de données.
  1. Effectuez l’annulation de l’inscription.

### <a name="i-dont-have-sufficient-privileges-to-start-system-services"></a><a name="setup-perms"></a> Je ne dispose pas des privilèges suffisants pour démarrer les services système.

- **Cause** . Cette erreur se produit dans deux situations :
  -   Le nom d’utilisateur et/ou le mot de passe sont incorrects.
  -   Le compte d’utilisateur spécifié ne dispose pas des privilèges suffisants pour se connecter en tant que service.

- **Résolution** . Accordez des informations d’identification « Ouvrir une session en tant que service » au compte d’utilisateur :

  1. Accédez à **Démarrer** > **Panneau de configuration** > **Outils d’administration** > **Stratégie de sécurité locale** > **Stratégie locale** > **Gestion des droits de l’utilisateur** .
  1. Sélectionnez **Ouvrir une session en tant que service** .
  1. Dans la boîte de dialogue **Propriétés** , ajoutez le compte d’utilisateur.
  1. Sélectionnez **Apply** (Appliquer), puis **OK** .
  1. Fermez toutes les fenêtres.

### <a name="a-database-has-an-out-of-date-status"></a><a name="setup-date"></a> Une base de données a un état « Obsolète ».

- **Cause** . SQL Data Sync supprime du service les bases de données qui sont hors connexion depuis 45 jours ou plus (délai calculé à partir du moment où la base de données a basculé hors connexion). Si une base de données est hors connexion pendant au moins 45 jours, puis qu’elle rebascule en ligne, son état est défini sur **Obsolète** .

- **Résolution** . Pour éviter l’état **Obsolète** , veillez à ce qu’aucune de vos bases de données ne reste hors ligne pendant 45 jours ou plus.

  Si l’état d’une base de données est **Obsolète**  :

  1. Supprimez la base de données dont l’état est **Obsolète** du groupe de synchronisation.
  1. Réajouter la base de données dans le groupe de synchronisation.

  > [!WARNING]
  > Vous perdez toutes les modifications apportées à cette base de données pendant qu’elle était hors connexion.

### <a name="a-sync-group-has-an-out-of-date-status"></a><a name="setup-date2"></a> Un groupe de synchronisation a un état « Obsolète ».

- **Cause** . Si une ou plusieurs modifications ne sont pas appliquées pendant la période de conservation de 45 jours, un groupe de synchronisation peut devenir obsolète.

- **Résolution** . Pour éviter que l’état d’un groupe de synchronisation ne devienne **Obsolète** , examinez régulièrement les résultats de vos travaux de synchronisation dans la visionneuse de l’historique. Recherchez et résolvez les problèmes liés aux modifications qui ne sont pas appliquées.

  Si l’état d’un groupe de synchronisation est **Obsolète** , supprimez le groupe et recréez-le.

### <a name="a-sync-group-cant-be-deleted-within-three-minutes-of-uninstalling-or-stopping-the-agent"></a><a name="setup-delete2"></a> Impossible de supprimer un groupe de synchronisation dans les trois minutes qui suivent la désinstallation ou l’arrêt de l’agent.

Vous ne pouvez pas supprimer un groupe de synchronisation dans les trois minutes qui suivent la désinstallation ou l’arrêt de l’agent client SQL Data Sync associé.

- **Résolution** .

  1. Supprimez un groupe de synchronisation pendant que les agents de synchronisation associés sont en ligne (recommandé).
  1. Si l’agent est hors connexion mais installé, mettez-le en ligne sur l’ordinateur local. Attendez que l’état de l’agent apparaisse comme **En ligne** sur le portail SQL Data Sync. Puis supprimez le groupe de synchronisation.
  1. Si l’agent est hors connexion parce qu’il a été désinstallé :  
    a.  Accédez au dossier d’installation de SQL Data Sync et supprimez le fichier XML de l’agent, si ce fichier existe.  
    b.  Installez l’agent sur un ordinateur local (il peut s’agir du même ordinateur ou d’un autre ordinateur). Envoyez ensuite la clé générée dans le portail pour l’agent qui apparaît comme étant hors connexion.  
    c. Essayez de supprimer le groupe de synchronisation.

### <a name="what-happens-when-i-restore-a-lost-or-corrupted-database"></a><a name="setup-restore"></a> Que se passe-t-il quand je restaure une base de données perdue ou endommagée ?

Si vous restaurez une base de données perdue ou endommagée à partir d’une sauvegarde, un problème de non-convergence des données peut survenir dans les groupes de synchronisation auxquels la base de données appartient.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur SQL Data Sync, consultez :

-   Vue d’ensemble - [Synchroniser des données entre plusieurs bases de données cloud et locales avec SQL Data Sync dans Azure](sql-data-sync-data-sql-server-sql-database.md)
-   Configurer Data Sync
    - Sur le portail - [Tutoriel : Configurer SQL Data Sync pour synchroniser les données entre Azure SQL Database et SQL Server](sql-data-sync-sql-server-configure.md)
    - Avec PowerShell
        -  [Utiliser PowerShell pour synchroniser des données entre plusieurs bases de données dans Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Utiliser PowerShell pour synchroniser des données entre une base de données dans Azure SQL Database et une base de données dans une instance SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Data Sync Agent - [Data Sync Agent pour SQL Data Sync dans Azure](sql-data-sync-agent-overview.md)
-   Bonnes pratiques - [Bonnes pratiques pour SQL Data Sync dans Azure](sql-data-sync-best-practices.md)
-   Supervision – [Superviser SQL Data Sync avec les journaux d’activité Azure Monitor](./monitor-tune-overview.md)
-   Mettre à jour le schéma de synchronisation
    -   Avec Transact-SQL - [Automatiser la réplication des modifications de schéma dans SQL Data Sync dans Azure](sql-data-sync-update-sync-schema.md)
    -   Avec PowerShell - [Utiliser PowerShell pour mettre à jour le schéma de synchronisation dans un groupe de synchronisation existant](scripts/update-sync-schema-in-sync-group.md)

Pour plus d’informations sur SQL Database, consultez :

-   [Vue d’ensemble des bases de données SQL](sql-database-paas-overview.md)
-   [Gestion du cycle de vie des bases de données](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))