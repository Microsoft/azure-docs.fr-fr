---
title: Configurer SQL Data Sync
description: Ce tutoriel vous montre comment configurer SQL Data Sync pour Azure
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/14/2019
ms.openlocfilehash: f5192176a6a0e174d5878c51defce70d949c1eb1
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96922331"
---
# <a name="tutorial-set-up-sql-data-sync-between-databases-in-azure-sql-database-and-sql-server"></a>Tutoriel : configurer SQL Data Sync entre des bases de données d'Azure SQL Database et SQL Server
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Dans ce tutoriel, vous allez apprendre à configurer SQL Data Sync en créant un groupe de synchronisation contenant à la fois des instances d'Azure SQL Database et de SQL Server. Le groupe de synchronisation est configuré de manière personnalisée et se synchronise selon la planification définie.

Ce tutoriel part du principe que vous avez déjà utilisé SQL Database et SQL Server.

Pour une vue d'ensemble de SQL Data Sync, consultez [Synchroniser des données entre des bases de données locales et cloud avec SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md).

Pour obtenir des exemples PowerShell sur la façon de configurer SQL Data Sync, consultez les guides pratiques consacrés à la synchronisation des données [entre des bases de données de SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md) ou [entre des bases de données d'Azure SQL Database et de SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md).

> [!IMPORTANT]
> Pour le moment, SQL Data Sync ne prend **pas** en charge Azure SQL Managed Instance.

## <a name="create-sync-group"></a>Créer un groupe de synchronisation

1. Accédez au [portail Azure](https://portal.azure.com) pour rechercher votre base de données dans SQL Database. Recherchez et sélectionnez **Base de données SQL**.

    ![Rechercher des bases de données, portail Microsoft Azure](./media/sql-data-sync-sql-server-configure/search-for-sql-databases.png)

1. Sélectionnez la base de données que vous souhaitez utiliser comme base de données Hub pour la synchronisation des données.

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/select-sql-database.png" alt-text = "Select from the database list, Microsoft Azure portal":::

    > [!NOTE]
    > La base de données Hub est le point de terminaison central d’une topologie de synchronisation, où un groupe de synchronisation a plusieurs points de terminaison de base de données. Toutes les autres bases de données membres avec des points de terminaison dans le groupe de synchronisation sont synchronisées avec la base de données Hub.

1. Dans le menu **Base de données SQL** de la base de données choisie, sélectionnez **Synchroniser avec les autres bases de données**.

    :::image type="content" source="./media/sql-data-sync-sql-server-configure/sync-to-other-databases.png" alt-text = "Sync to other databases, Microsoft Azure portal":::

1. Dans la page **Synchroniser avec les autres bases de données**, sélectionnez **Nouveau groupe de synchronisation**. Dans la page **Nouveau groupe de synchronisation** qui s’ouvre, l’étape 1 **Créer un groupe de synchronisation** apparaît.

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/new-sync-group-private-link.png" alt-text = "Set up new sync group with private link":::

   Dans la page **Créer un groupe de synchronisation de données**, changez les paramètres suivants :

   | Paramètre                        | Description |
   | ------------------------------ | ------------------------------------------------- |
   | **Nom du groupe de synchronisation** | Entrez le nom du nouveau groupe de synchronisation. Ce nom est différent du nom de la base de données. |
   | **Synchroniser la base de données de métadonnées** | Choisissez entre créer une base de données (recommandé) ou utiliser une base de données existante.<br/><br/>Si vous choisissez **Nouvelle base de données**, sélectionnez **Créer une base de données**. Ensuite, dans la page **Base de données SQL**, nommez et configurez la nouvelle base de données, puis sélectionnez **OK**.<br/><br/>Si vous choisissez **Utiliser une base de données existante**, sélectionnez la base de données dans la liste. |
   | **Synchronisation automatique** | Sélectionnez **Activé** ou **Désactivé**.<br/><br/>Si vous choisissez **Activé**, entrez un nombre et sélectionnez **Secondes**, **Minutes**, **Heures** ou **Jours** dans la section **Fréquence de synchronisation**.<br/> La première synchronisation commence après que la période d’intervalle sélectionnée se soit écoulée à partir du moment où la configuration est enregistrée.|
   | **Résolution des conflits** | Sélectionnez **Gain du hub** ou **Gain du membre**.<br/><br/>**Gain du hub** signifie qu’en cas de conflit, les données de la base de données Hub remplacent les données en conflit de la base de données membre.<br/><br/>**Gain du membre** signifie qu’en cas de conflit, les données de la base de données membre remplacent les données en conflit de la base de données Hub. |
   | **Utiliser une liaison privée** | Choisissez un point de terminaison privé managé par le service pour établir une connexion sécurisée entre le service de synchronisation et la base de données Hub. |

   > [!NOTE]
   > Microsoft recommande de créer une base de données vide à utiliser comme **base de données des métadonnées de synchronisation**. SQL Data Sync crée les tables dans cette base de données et exécute une charge de travail fréquente. Cette base de données est partagée comme **base des métadonnées de synchronisation** pour l’ensemble des groupes de synchronisation dans une région et un abonnement sélectionnés. Vous ne pouvez pas modifier la base de données ou son nom sans supprimer tous les groupes de synchronisation et les agents de synchronisation dans la région.

   Sélectionnez **OK** et attendez que le groupe de synchronisation soit créé et déployé.
   
1. Dans la page **Nouveau groupe de synchronisation**, si vous avez sélectionné **Utiliser une liaison privée**, vous devez approuver la connexion au point de terminaison privé. Le lien dans le message d’informations vous permet d’accéder à l’expérience des connexions au point de terminaison privé, où vous pouvez approuver la connexion. 

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/approve-private-link.png" alt-text = "Approve private link":::

## <a name="add-sync-members"></a>Ajouter des membres de synchronisation

Une fois le groupe de synchronisation créé et déployé, l’étape 2 **Ajouter des membres de synchronisation** apparaît en surbrillance dans la page **Nouveau groupe de synchronisation**.

Dans la section **Base de données Hub**, entrez les informations d'identification du serveur où réside la base de données Hub. N’entrez pas de *nouvelles* informations d’identification dans cette section.

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/steptwo.png" alt-text = "Enter existing credentials for the hub database server":::

### <a name="to-add-a-database-in-azure-sql-database"></a>Pour ajouter une base de données dans Azure SQL Database

Dans la section **Base de données membre**, ajoutez éventuellement une base de données Azure SQL Database au groupe de synchronisation en sélectionnant **Add an Azure SQL Database** (Ajouter une instance Azure SQL Database). La page **Configurer Azure SQL Database** s’ouvre.
  
   :::image type="content" source="./media/sql-data-sync-sql-server-configure/step-two-configure.png" alt-text = "Add a database to the sync group":::
   
  Dans la page **Configurer Azure SQL Database**, changez les paramètres suivants :

  | Paramètre                       | Description |
  | ----------------------------- | ------------------------------------------------- |
  | **Nom du membre de synchronisation** | Entrez le nom du nouveau membre de synchronisation. Ce nom est différent de celui de la base de données. |
  | **Abonnement** | Sélectionnez l’abonnement Azure associé en vue de la facturation. |
  | **Azure SQL Server** | Sélectionnez le serveur existant. |
  | **Azure SQL Database** | Sélectionnez la base de données existante dans SQL Database. |
  | **Sens de la synchronisation** | Sélectionnez **Synchronisation bidirectionnelle**, **Vers le hub** ou **À partir du hub**. |
  | **Nom d’utilisateur** et **Mot de passe** | Entrez les informations d'identification du serveur qui héberge la base de données membre. N’entrez pas de *nouvelles* informations d’identification dans cette section. |
  | **Utiliser une liaison privée** | Choisissez un point de terminaison privé managé par le service pour établir une connexion sécurisée entre le service de synchronisation et la base de données membre. |

  Sélectionnez **OK** et attendez que le nouveau membre de synchronisation soit créé et déployé.

<a name="add-on-prem"></a>

### <a name="to-add-a-sql-server-database"></a>Pour ajouter une base de données SQL Server

Dans la section **Base de données membre**, ajoutez éventuellement une base de données SQL Server au groupe de synchronisation en sélectionnant **Ajouter une base de données locale**. Dans la page **Configurer localement** qui s’ouvre, vous pouvez effectuer les tâches suivantes :

1. Sélectionnez **Choisir la passerelle de l’agent de synchronisation**. La page **Sélectionner l’agent de synchronisation** s’ouvre.

   :::image type="content" source="./media/sql-data-sync-sql-server-configure/steptwo-agent.png" alt-text = "Creating a sync agent":::

1. Dans la page **Choisir l’agent de synchronisation**, choisissez entre utiliser un agent existant ou en créer un.

   Si vous choisissez **Agents existants**, sélectionnez un agent dans la liste.

   Si vous choisissez **Créer un agent**, effectuez les étapes suivantes :

   1. Téléchargez l’agent de synchronisation de données à l’aide du lien fourni et installez-le sur l’ordinateur qui héberge le serveur SQL Server. Vous pouvez également télécharger directement l’agent à partir de [Azure SQL Data Sync Agent](https://www.microsoft.com/download/details.aspx?id=27693).

      > [!IMPORTANT]
      > Vous devez ouvrir le port TCP sortant 1433 dans le pare-feu pour permettre à l’agent du client de communiquer avec le serveur.

   1. Entrez le nom de l’agent.

   1. Sélectionnez **Créer et générer une clé** et copiez la clé de l’agent dans le Presse-papiers.

   1. Sélectionnez **OK** pour fermer la page **Sélectionner l’agent de synchronisation**.

1. Sur l’ordinateur du serveur SQL Server, recherchez et exécutez l’application de l’agent de synchronisation du client.

   ![Application de l’agent de synchronisation du client](./media/sql-data-sync-sql-server-configure/datasync-preview-clientagent.png)

    1. Dans l’application de l’agent de synchronisation, sélectionnez **Submit Agent Key (Envoyer la clé de l’agent)** . La boîte de dialogue **Sync Metadata Database Configuration (Configuration de la base des métadonnées de synchronisation)** s’ouvre.

    1. Dans la boîte de dialogue **Sync Metadata Database Configuration (Configuration de la base des métadonnées de synchronisation)** , collez la clé de l’agent copiée à partir du portail Azure. Indiquez également les informations d'identification du serveur qui héberge la base de données des métadonnées. (Si vous avez créé une base de données de métadonnées, celle-ci réside sur le même serveur que la base de données Hub.) Sélectionnez **OK** et attendez la fin de la configuration.

        ![Entrer la clé de l’agent et les informations d’identification du serveur](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-enterkey.png)

        > [!NOTE]
        > Si vous recevez une erreur de pare-feu, créez une règle de pare-feu sur Azure pour autoriser le trafic entrant en provenance de l’ordinateur qui héberge le serveur SQL Server. Vous pouvez créer cette règle manuellement dans le portail ou dans SSMS (SQL Server Management Studio). Dans SSMS, connectez-vous à la base de données Hub sur Azure en entrant son nom au format <nom_base_données_hub>.database.windows.net.

    1. Sélectionnez **Inscrire** pour inscrire une base de données SQL Server auprès de l’agent. La boîte de dialogue **Configuration de SQL Server** s’ouvre.

        ![Ajouter et configurer une base de données SQL Server](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-adddb.png)

    1. Dans la boîte de dialogue **Configuration de SQL Server**, choisissez une connexion avec authentification SQL Server ou authentification Windows. Si vous choisissez l’authentification SQL Server, entrez les informations d’identification existantes. Indiquez le nom du serveur SQL Server et le nom de la base de données à synchroniser, puis sélectionnez **Tester la connexion** pour tester vos paramètres. Sélectionnez ensuite **Enregistrer**. La base de données inscrite apparaît alors dans la liste.

        ![La base de données SQL Server est maintenant inscrite.](./media/sql-data-sync-sql-server-configure/datasync-preview-agent-dbadded.png)

    1. Fermez l’application Agent de synchronisation du client.

1. Dans le portail, dans la page **Configurer localement**, choisissez **Sélectionner la base de données**.

1. Dans la page **Sélectionner la base de données**, indiquez le nom du nouveau membre de synchronisation dans le champ **Nom du membre de synchronisation**. Ce nom est différent du nom de la base de données. Sélectionnez la base de données dans la liste. Dans le champ **Sens de la synchronisation**, sélectionnez **Synchronisation bidirectionnelle**, **Vers le hub** ou **À partir du hub**.

    ![Sélectionner la base de données locale](./media/sql-data-sync-sql-server-configure/datasync-preview-selectdb.png)

1. Sélectionnez **OK** pour fermer la page **Sélectionner la base de données**. Ensuite, sélectionnez **OK** pour fermer la page **Configurer localement** et attendez que le nouveau membre de synchronisation soit créé et déployé. Pour finir, sélectionnez **OK** pour fermer la page **Sélectionner les membres de synchronisation**.

> [!NOTE]
> Pour vous connecter à SQL Data Sync et à l’agent local, ajoutez votre nom d’utilisateur au rôle *DataSync_Executor*. Data Sync crée ce rôle sur l’instance SQL Server.

## <a name="configure-sync-group"></a>Configurer le groupe de synchronisation

Une fois les nouveaux membres du groupe de synchronisation créés et déployés, l’étape 3 **Configurer le groupe de synchronisation** apparaît en surbrillance dans la page **Nouveau groupe de synchronisation**.

![Paramètres de l’étape 3](./media/sql-data-sync-sql-server-configure/stepthree.png)

1. Dans la page **Tables**, sélectionnez une base de données dans la liste des membres du groupe de synchronisation, puis sélectionnez **Actualiser le schéma**.

1. Dans la liste, sélectionnez les tables à synchroniser. Toutes les colonnes étant sélectionnées par défaut, décochez celles que vous ne souhaitez pas synchroniser. Vérifiez que la colonne de la clé primaire est sélectionnée.

1. Sélectionnez **Enregistrer**.

1. Par défaut, les bases de données ne sont synchronisées que si l’opération de synchronisation est planifiée ou exécutée manuellement. Pour exécuter une synchronisation manuelle, accédez à votre base de données dans SQL Database dans le portail Azure, sélectionnez **Synchroniser avec d’autres bases de données**, puis sélectionnez le groupe de synchronisation. La page **Synchronisation des données** s’ouvre. Sélectionnez **Synchroniser**.

    ![Synchronisation manuelle](./media/sql-data-sync-sql-server-configure/datasync-sync.png)

## <a name="faq"></a>Questions fréquentes (FAQ)

**À quelle fréquence Data Sync synchronise-t-il mes données ?**

La durée minimale entre les synchronisations est de cinq minutes.

**SQL Data Sync gère-t-il intégralement la création des tables ?**

Si des tables de schéma de synchronisation ne figurent pas dans la base de données de destination, SQL Data Sync les crée avec les colonnes que vous avez sélectionnées. Cela ne permet toutefois pas la création d’un schéma haute fidélité, et ce pour les raisons suivantes :

- Seules les colonnes que vous sélectionnez sont créées dans la table de destination. Les colonnes non sélectionnées sont ignorées.
- Seuls les index des colonnes sélectionnées sont créés dans la table de destination. Pour les colonnes non sélectionnées, ces index sont ignorés.
- Les index sur les colonnes de type XML ne sont pas créés.
- Les contraintes CHECK ne sont pas créées.
- Les déclencheurs sur les tables sources ne sont pas créés.
- Les vues et procédures stockées ne sont pas créées.

En raison de ces limitations, nous vous recommandons les opérations suivantes :

- Pour les environnements de production, créez vous-même le schéma haute fidélité.
- Quand vous testez le service, utilisez la fonctionnalité de provisionnement automatique.

**Comment se fait-il que je vois des tables que je n’ai pas créées ?**

Data Sync crée des tables supplémentaires dans la base de données pour le suivi des changements. Ne les supprimez pas, sinon Data Sync cessera de fonctionner.

**Mes données sont-elles convergentes après une synchronisation ?**

Pas nécessairement. Prenez un groupe de synchronisation comprenant un hub et trois rayons (A, B et C), avec les synchronisations suivantes : Hub vers A, Hub vers B et Hub vers C. Si un changement est apporté à la base de données A *après* la synchronisation Hub vers A, ce changement n’est écrit dans la base de données B ou C qu’au cours de la prochaine tâche de synchronisation.

**Comment faire pour répercuter les changements de schéma dans un groupe de synchronisation ?**

Apportez et propagez tous les changements de schéma manuellement.

1. Répliquez les modifications du schéma manuellement vers le hub et tous les membres de synchronisation.
1. Mettez à jour le schéma de synchronisation.

Ajout de nouvelles tables et colonnes :

Les nouvelles tables et colonnes n’ont aucun impact sur la synchronisation actuelle. Data Sync les ignore tant qu’elles ne sont pas ajoutées au schéma de synchronisation. Quand vous ajoutez des objets de base de données, suivez cette séquence :

1. Ajoutez les nouvelles tables ou colonnes au hub et à tous les membres de synchronisation.
1. Ajoutez les nouvelles tables ou colonnes au schéma de synchronisation.
1. Commencez par insérer des valeurs dans les nouvelles tables et colonnes.

Changement du type de données d’une colonne :

Lorsque vous modifiez le type de données d’une colonne existante, Data Sync continue à fonctionner tant que les nouvelles valeurs correspondent au type de données d’origine défini dans le schéma de synchronisation. Par exemple, si vous remplacez le type **int** dans la base de données source par **bigint**, Data Sync continue de fonctionner jusqu’à ce que vous insériez une valeur trop grande pour le type de données **int**. Pour terminer le changement, répliquez le changement de schéma manuellement sur le hub et sur tous les membres de synchronisation, puis mettez à jour le schéma de synchronisation.

**Comment faire pour exporter et importer une base de données avec Data Sync ?**

Après avoir exporté une base de données sous forme de fichier *.bacpac* et avoir importé le fichier pour créer une base de données, effectuez les tâches suivantes pour utiliser Data Sync dans la nouvelle base de données :

1. Nettoyer les objets Data Sync et les tables annexes de la nouvelle base de données à l’aide de [ce script](https://github.com/vitomaz-msft/DataSyncMetadataCleanup/blob/master/Data%20Sync%20complete%20cleanup.sql). Le script supprime tous les objets Data Sync requis de la base de données.
1. Recréer le groupe de synchronisation avec la nouvelle base de données. Si vous n’avez plus besoin de l’ancien groupe de synchronisation, supprimez-le.

**Où puis-je trouver des informations sur l’agent client ?**

Pour les questions fréquemment posées sur l’agent client, consultez [FAQ sur l'agent](sql-data-sync-agent-overview.md#agent-faq).

**Est-il nécessaire d’approuver manuellement la liaison privée avant de pouvoir commencer à l’utiliser ?**

Oui, vous devez approuver manuellement le point de terminaison privé managé par le service dans la page Connexions de point de terminaison privé du portail Azure pendant le déploiement du groupe de synchronisation ou à l’aide de PowerShell.

## <a name="next-steps"></a>Étapes suivantes

Félicitations ! Vous avez créé un groupe de synchronisation comprenant une instance SQL Database et une base de données SQL Server.

Pour plus d’informations sur SQL Data Sync, consultez :

- [Data Sync Agent pour Azure SQL Data Sync](sql-data-sync-agent-overview.md)
- [Bonnes pratiques](sql-data-sync-best-practices.md) et [Comment résoudre les problèmes liés à Azure SQL Data Sync](sql-data-sync-troubleshoot.md)
- [Superviser SQL Data Sync avec des journaux Azure Monitor](./monitor-tune-overview.md)
- [Mettre à jour le schéma de synchronisation avec Transact-SQL](sql-data-sync-update-sync-schema.md) ou [PowerShell](scripts/update-sync-schema-in-sync-group.md)

Pour plus d’informations sur SQL Database, consultez :

- [Vue d’ensemble des bases de données SQL](sql-database-paas-overview.md)
- [Gestion du cycle de vie des bases de données](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))
