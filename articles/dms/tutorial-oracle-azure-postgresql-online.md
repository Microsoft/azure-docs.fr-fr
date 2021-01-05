---
title: 'Tutoriel : Migrer Oracle en ligne vers Azure Database pour PostgreSQL'
titleSuffix: Azure Database Migration Service
description: Découvrez comment effectuer une migration en ligne d’Oracle en local ou sur des machines virtuelles vers Azure Database pour PostgreSQL à l’aide d’Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/24/2020
ms.openlocfilehash: df789161bb9db8d49f069992600b5fcb4f78dd03
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006540"
---
# <a name="tutorial-migrate-oracle-to-azure-database-for-postgresql-online-using-dms-preview"></a>Tutoriel : Migrer Oracle vers Azure Database pour PostgreSQL en ligne à l’aide de DMS (préversion)

Vous pouvez utiliser Azure Database Migration Service pour migrer les bases de données Oracle hébergées localement ou sur des machines virtuelles vers [Azure Database pour PostgreSQL](../postgresql/index.yml) avec un temps d’arrêt minimal. En d’autres termes, vous pouvez effectuer la migration avec un temps d’arrêt minimal de l’application. Dans ce tutoriel, vous allez migrer l’exemple de base de données **HR** à partir d’une instance locale ou sur machines virtuelles d’Oracle 11g vers Azure Database pour PostgreSQL à l’aide de l’activité de migration en ligne dans Azure Database Migration Service.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> * Évaluer l’effort de migration à l’aide de l’outil ora2pg.
> * Migrer l’exemple de schéma à l’aide de l’outil ora2pg.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration en utilisant Azure Database Migration Service.
> * Exécuter la migration.
> * Surveiller la migration.

> [!NOTE]
> L’utilisation d’Azure Database Migration Service pour effectuer une migration en ligne nécessite la création d’une instance basée sur le niveau tarifaire Premium.

> [!IMPORTANT]
> Pour une expérience de migration optimale, Microsoft recommande de créer une instance Azure Database Migration Service dans la même région Azure que la base de données cible. Le déplacement des données entre les régions ou les zones géographiques peut ralentir le processus de migration et introduire des erreurs.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

Cet article décrit comment effectuer une migration en ligne d’Oracle vers Azure Database pour PostgreSQL.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Téléchargez et installez [Oracle 11g version 2 (Standard Edition, Standard Edition One ou Enterprise Edition)](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html).
* Téléchargez l’exemple de base de données **HR** en cliquant [ici](https://docs.oracle.com/database/121/COMSC/installation.htm#COMSC00002).
* Téléchargez et [installez ora2pg sur Windows ou sur Linux](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows%20and%20Linux.pdf).
* [Créer une instance dans Azure Database pour PostgreSQL](../postgresql/quickstart-create-server-database-portal.md).
* Connectez-vous à l’instance et créez une base de données à l’aide des instructions figurant dans ce [document](../postgresql/tutorial-design-database-using-azure-portal.md).
* Créez un Réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.

  > [!NOTE]
  > Pendant la configuration du réseau virtuel, si vous utilisez ExpressRoute avec le peering réseau à Microsoft, ajoutez ces [points de terminaison](../virtual-network/virtual-network-service-endpoints-overview.md) au sous-réseau où doit être provisionné le service :
  >
  > * Point de terminaison de base de données cible (un point de terminaison SQL ou Cosmos DB, par exemple)
  > * Point de terminaison de stockage
  > * Point de terminaison Service Bus
  >
  > Cette configuration est nécessaire car Azure Database Migration Service ne dispose pas d’une connectivité Internet.

* Vérifiez que les règles du groupe de sécurité réseau de votre réseau virtuel ne bloquent pas les ports suivants pour les communications entrantes vers Azure Database Migration Service : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurez votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Ouvrez votre pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur Oracle source. Par défaut, il s’agit du port TCP 1521.
* Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.
* Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) de niveau serveur pour Azure Database pour PostgreSQL afin de permettre à Azure Database Migration Service d’accéder aux bases de données cibles. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
* Activez l’accès aux bases de données Oracle sources.

  > [!NOTE]
  > Un utilisateur doit avoir le rôle d’administrateur de base de données pour se connecter à la source Oracle.

  * Les journaux de phase de restauration par progression d'archivage sont requis pour que la synchronisation incrémentielle dans Azure Database Migration Service puisse capturer les modifications des données. Pour configurer la source Oracle, suivez ces étapes :
    * Connectez-vous à l’aide du privilège SYSDBA en exécutant la commande suivante :

      ```
      sqlplus (user)/(password) as sysdba
      ```

    * Arrêtez l’instance de base de données en exécutant la commande suivante.

      ```
      SHUTDOWN IMMEDIATE;
      ```

      Attendez la confirmation `'ORACLE instance shut down'`.

    * Démarrez la nouvelle instance et montez (mais n’ouvrez pas) la base de données pour activer ou désactiver l’archivage en exécutant la commande suivante :

      ```
      STARTUP MOUNT;
      ```

      La base de données doit être montée. Attendez la confirmation « Instance Oracle démarrée ».

    * Modifiez le mode d’archivage de la base de données en exécutant la commande suivante :

      ```
      ALTER DATABASE ARCHIVELOG;
      ```

    * Ouvrez la base de données pour les opérations normales en exécutant la commande suivante :

      ```
      ALTER DATABASE OPEN;
      ```

      Vous devrez peut-être redémarrer pour que le fichier ARC apparaisse.

    * Pour cela, exécutez la commande suivante :

      ```
      SELECT log_mode FROM v$database;
      ```

      Vous devez recevoir une réponse `'ARCHIVELOG'`. Si la réponse est `'NOARCHIVELOG'`, la condition requise n’est pas remplie.

  * Activez la journalisation supplémentaire pour la réplication à l’aide d’une des options suivantes.

    * **Option 1**.
      Modifiez la journalisation supplémentaire au niveau de la base de données pour couvrir toutes les tables avec des clés primaires et un index unique. La requête de détection retournera `'IMPLICIT'`.

      ```
      ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY, UNIQUE) COLUMNS;
      ```

      Modifiez la journalisation supplémentaire de niveau table. Exécutez uniquement pour les tables dotées de la manipulation des données qui n’ont pas de clés primaires ou d’index uniques.

      ```
      ALTER TABLE [TABLENAME] ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
      ```

    * **Option 2**.
      Modifiez la journalisation supplémentaire au niveau de la base de données pour couvrir toutes les tables. La requête de détection retourne `'YES'`.

      ```
      ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
      ```

      Modifiez la journalisation supplémentaire de niveau table. Suivez la logique ci-dessous pour exécuter une seule instruction pour chaque table.

      Si la table possède une clé primaire :

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
      ```

      Si la table possède un index unique :

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG GROUP (first unique index columns) ALWAYS;
      ```

      Dans le cas contraire, exécutez la commande suivante :

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
      ```

    Pour cela, exécutez la commande suivante :

      ```
      SELECT supplemental_log_data_min FROM v$database;
      ```

    Vous devez recevoir une réponse `'YES'`.

## <a name="assess-the-effort-for-an-oracle-to-azure-database-for-postgresql-migration"></a>Évaluer l’effort requis pour une migration d’Oracle vers Azure Database pour PostgreSQL

Nous vous recommandons d’utiliser ora2pg pour évaluer l’effort requis pour migrer d’Oracle vers Azure Database pour PostgreSQL. Utilisez la directive `ora2pg -t SHOW_REPORT` pour créer un rapport listant tous les objets Oracle, le coût de migration estimé (en jours de développeur) et certains objets de base de données qui peuvent nécessiter une attention particulière dans le cadre de la conversion.

La plupart des clients passent un temps considérable à consulter le rapport d’évaluation et à prendre en considération l’effort de conversion automatique et manuelle.

Afin de configurer et exécuter ora2pg pour créer un rapport d’évaluation, consultez la section **Premigration: Assessment** du document [Oracle to Azure Database for PostgreSQL Cookbook](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf). Un exemple de rapport d’évaluation ora2pg est disponible comme référence [ici](https://ora2pg.darold.net/report.html).

## <a name="export-the-oracle-schema"></a>Exporter le schéma Oracle

Nous vous recommandons d’utiliser ora2pg pour convertir le schéma Oracle et les autres objets Oracle (types, procédures, fonctions, etc.) en un schéma compatible avec Azure Database pour PostgreSQL. ora2pg inclut de nombreuses directives pour vous aider à prédéfinir certains types de données. Par exemple, vous pouvez utiliser la directive `DATA_TYPE` pour remplacer toutes les occurrences de NUMBER(*,0) par bigint au lieu de NUMERIC(38).

Vous pouvez exécuter ora2pg pour exporter tous les objets de base de données vers des fichiers .sql. Vous pouvez ensuite passer en revue les fichiers .sql avant de les importer dans Azure Database pour PostgreSQL à l’aide de psql, ou vous pouvez exécuter le script .SQL dans PgAdmin.

```
psql -f [FILENAME] -h [AzurePostgreConnection] -p 5432 -U [AzurePostgreUser] -d database 
```

Par exemple :

```
psql -f %namespace%\schema\sequences\sequence.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database
```

Pour configurer et exécuter ora2pg pour la conversion de schéma, consultez la section **Migration: Schema and data** du document [Oracle to Azure Database for PostgreSQL Cookbook](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf).

## <a name="set-up-the-schema-in-azure-database-for-postgresql"></a>Configurer le schéma dans Azure Database pour PostgreSQL

Vous pouvez choisir de convertir des schémas de table, des procédures stockées, des packages et d’autres objets de base de données Oracle pour les rendre compatibles avec Postgres en utilisant ora2pg avant de démarrer un pipeline de migration dans Azure Database Migration Service. Consultez les liens ci-dessous pour savoir comment utiliser ora2pg :

* [Installer ora2pg sur Windows](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows%20and%20Linux.pdf)
* [Livre de recettes sur la migration d’Oracle vers Azure PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf)

Azure Database Migration Service peut également créer le schéma de table PostgreSQL. Le service accède au schéma de table dans la source Oracle connectée et crée un schéma de table compatible dans Azure Database pour PostgreSQL. Veillez à valider et vérifier le format de schéma dans Azure Database pour PostgreSQL une fois qu’Azure Database Migration Service a terminé la création du schéma et le déplacement des données.

> [!IMPORTANT]
> Azure Database Migration Service crée uniquement le schéma de table ; les autres objets de base de données, tels que les procédures stockées, les packages, les index, etc., ne sont pas créés.

Veillez aussi à supprimer la clé étrangère dans la base de données cible pour que la charge complète s’exécute. Reportez-vous à la section **Migrer l’exemple de schéma** de [cet article](./tutorial-postgresql-azure-postgresql-online.md) afin d’obtenir un script que vous pouvez utiliser pour supprimer la clé étrangère. Utilisez Azure Database Migration Service pour exécuter la synchronisation et la charge complète.

### <a name="when-the-postgresql-table-schema-already-exists"></a>Quand le schéma de table PostgreSQL existe déjà

Si vous créez un schéma PostgreSQL à l’aide d’outils comme ora2pg avant de commencer le déplacement des données avec Azure Database Migration Service, mappez les tables sources aux tables cibles dans Azure Database Migration Service.

1. Lorsque vous créez un projet de migration Oracle vers Azure Database pour PostgreSQL, vous êtes invité à sélectionner la base de données cible et le schéma cible dans l’étape de sélection des schémas. Renseignez la base de données cible et le schéma cible.

   ![Capture d’écran montrant Mapper aux bases de données cibles.](media/tutorial-oracle-azure-postgresql-online/dms-map-to-target-databases.png)

2. L’écran **Paramètres de migration** présente une liste de tables dans la source Oracle. Azure Database Migration Service tente de faire correspondre des tables dans les tables sources et cibles en fonction du nom de la table. S’il existe plusieurs tables cibles correspondantes avec une casse différente, vous pouvez sélectionner la table cible à mapper.

    ![Capture d’écran montrant les paramètres de migration.](media/tutorial-oracle-azure-postgresql-online/dms-migration-settings.png)

> [!NOTE]
> Si vous avez besoin de mapper des noms de tables sources à des tables avec des noms différents, envoyez un e-mail à [dmsfeedback@microsoft.com](mailto:dmsfeedbac@microsoft.com) et nous pouvons vous fournir un script pour automatiser le processus.

### <a name="when-the-postgresql-table-schema-doesnt-exist"></a>Quand le schéma de table PostgreSQL n’existe pas

Si la base de données PostgreSQL cible ne contient aucune information de schéma de table, Azure Database Migration Service convertit le schéma source et le recrée dans la base de données cible. Gardez à l’esprit qu’Azure Database Migration Service crée uniquement le schéma de table et non les autres objets de base de données, tels que les procédures stockées, les packages et les index.
Pour qu’Azure Database Migration Service crée le schéma pour vous, assurez-vous que votre environnement cible comprend un schéma sans tables existantes. Si Azure Database Migration Service découvre une table, le service suppose que le schéma a été créé par un outil externe comme ora2pg.

> [!IMPORTANT]
> Azure Database Migration Service exige que toutes les tables soient créées de la même façon, en utilisant Azure Database Migration Service ou un outil comme ora2pg, mais pas les deux.

Pour commencer :

1. Créez un schéma dans la base de données cible en fonction des exigences de votre application. Par défaut, les noms des colonnes et du schéma de table PostgreSQL sont en minuscules. En revanche, les noms des colonnes et du schéma de table Oracle sont par défaut en majuscules.
2. Dans l’étape de sélection des schémas, spécifiez la base de données cible et le schéma cible.
3. En fonction du schéma que vous créez dans Azure Database pour PostgreSQL, Azure Database Migration Service utilise les règles de transformation suivantes :

    Si le nom du schéma dans la source Oracle est identique à celui dans Azure Database pour PostgreSQL, Azure Database Migration Service *crée le schéma de table en utilisant la même casse que dans la cible*.

    Par exemple :

    | Schéma Oracle source | Schéma Database.Schema PostgreSQL cible | Schéma schema.table.column créé par DMS |
    | ------------- | ------------- | ------------- |
    | HR | targetHR.public | public.countries.country_id |
    | HR | targetHR.trgthr | trgthr.countries.country_id |
    | HR | targetHR.TARGETHR | "TARGETHR"."COUNTRIES"."COUNTRY_ID" |
    | HR | targetHR.HR | "HR"."COUNTRIES"."COUNTRY_ID" |
    | HR | targetHR.Hr | *Impossible de mapper les casses mixtes |

    *Pour créer des noms de tables et de schémas à casse mixte dans la base de données PostgreSQL cible, contactez [dmsfeedback@microsoft.com](mailto:dmsfeedback@microsoft.com). Nous pouvons fournir un script pour configurer le schéma de table à casse mixte dans la base de données PostgreSQL cible.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

   ![Afficher les abonnements au portail](media/tutorial-oracle-azure-postgresql-online/portal-select-subscriptions.png)

2. Sélectionnez l’abonnement dans lequel vous souhaitez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-oracle-azure-postgresql-online/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-oracle-azure-postgresql-online/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Créer une instance de service DMS

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-oracle-azure-postgresql-online/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-create2.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom pour le service, l’abonnement, et un réseau virtuel nouveau ou existant.

4. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès à l’instance Oracle source et à l’instance Azure Database pour PostgreSQL cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

5. Sélectionnez un niveau tarifaire.

    Pour plus d’informations sur les coûts et les niveaux de tarification, consultez la [page de tarification](https://aka.ms/dms-pricing).

    ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-settings5.png)

6. Sélectionnez **Créer** pour créer le service.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

    ![Rechercher toutes les instances Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée, puis sélectionnez l’instance.

    ![Rechercher votre instance Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-instance-search.png)

3. Sélectionnez **+ Nouveau projet de migration**.
4. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet, dans la zone de texte **Type de serveur source**, sélectionnez **Oracle** et, dans la zone de texte **Type de serveur cible**, sélectionnez **Azure Database pour PostgreSQL**.
5. Dans la section **Choisir un type d’activité**, sélectionnez **Migration de données en ligne**.

   ![Créer un projet Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-create-project5.png)

   > [!NOTE]
   > Une autre possibilité consiste à choisir **Créer un projet uniquement** pour créer le projet de migration à ce stade et exécuter la migration ultérieurement.

6. Sélectionnez **Enregistrer**, notez les conditions requises pour pouvoir utiliser Azure Database Migration Service pour effectuer une migration en ligne, puis sélectionnez **Créer et exécuter une activité**.

## <a name="specify-source-details"></a>Spécifier les détails de la source

* Dans l’écran **Ajouter les détails de la source**, spécifiez les détails de connexion de l’instance Oracle source.

  ![Écran Ajouter les détails de la source](media/tutorial-oracle-azure-postgresql-online/dms-add-source-details.png)

## <a name="upload-oracle-oci-driver"></a>Télécharger le pilote Oracle OCI

1. Sélectionnez **Enregistrer**, puis, dans l’écran **Installer le pilote OCI**, connectez-vous à votre compte Oracle et téléchargez le pilote **instantclient-basiclite-windows.x64-12.2.0.1.0.zip**(37 128 586 octet(s)) (Somme de contrôle SHA1 : 865082268) en cliquant [ici](https://www.oracle.com/technetwork/topics/winx64soft-089540.html#ic_winx64_inst).
2. Téléchargez le pilote dans un dossier partagé.

   Assurez-vous que ce dossier est partagé avec le nom d’utilisateur que vous avez spécifié avec au minimum un accès en lecture seule. Azure Database Migration Service accède au partage et lit à partir de ce partage pour charger le pilote OCI sur Azure en empruntant l’identité du nom d’utilisateur que vous spécifiez.

   Le nom d’utilisateur que vous spécifiez doit être un compte d’utilisateur Windows.

   ![Installation du pilote OCI](media/tutorial-oracle-azure-postgresql-online/dms-oci-driver-install.png)

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Sélectionnez **Enregistrer**, puis, dans l’écran **Détails de la cible**, spécifiez les détails de connexion du serveur Azure Database pour PostgreSQL cible, à savoir l’instance Azure Database pour PostgreSQL préconfigurée sur laquelle le schéma **HR** a été déployé.

    ![Écran Détails de la cible](media/tutorial-oracle-azure-postgresql-online/dms-add-target-details1.png)

2. Sélectionnez **Enregistrer**, puis dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    ![Mapper aux bases de données cibles](media/tutorial-oracle-azure-postgresql-online/dms-map-target-details.png)

3. Sélectionnez **Enregistrer**, dans l’écran **Récapitulatif de la migration** pour la zone de texte **Nom de l’activité**, spécifiez un nom pour l’activité de migration, puis examinez le récapitulatif pour vous assurer que les détails de la source et de la cible correspondent à ceux que vous avez spécifiés précédemment.

    ![Résumé de la migration](media/tutorial-oracle-azure-postgresql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

  La fenêtre d’activité de migration s’affiche, et le champ **État** de l’activité affiche **Initialisation en cours**.

## <a name="monitor-the-migration"></a>Surveiller la migration

1. Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à ce que le champ **État** de la migration prenne la valeur **En cours d’exécution**.

     ![État de l’activité – En cours d’exécution](media/tutorial-oracle-azure-postgresql-online/dms-activity-running.png)

2. Sous **Nom de la base de données**, sélectionnez une base de données spécifique pour obtenir l’état de migration des opérations **Full data load** (Charge complète des données) et **Incremental data sync** (Synchronisation incrémentielle des données).

    L’opération Charge complète des données affichera l’état de migration de la charge initiale, tandis que l’opération Synchronisation des données incrémentielles indiquera l’état de capture des changements de données (CDC).

     ![État de l’activité : Charge complète terminée](media/tutorial-oracle-azure-postgresql-online/dms-activity-full-load-completed.png)

     ![État de l’activité : Synchronisation des données incrémentielles](media/tutorial-oracle-azure-postgresql-online/dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>Effectuer le basculement de la migration

Une fois le chargement complet initial effectué, les bases de données identifiées par le libellé **Prêt pour le basculement**.

1. Lorsque vous êtes prêt à effectuer la migration de base de données, sélectionnez **Démarrer le basculement**.

2. Veillez à arrêter toutes les transactions entrantes pour la base de données source ; attendez que le compteur **Changements en attente** indique la valeur **0**.

   ![Démarrer le basculement](media/tutorial-oracle-azure-postgresql-online/dms-start-cutover.png)

3. Sélectionnez **Confirmer**, puis **Appliquer**.
4. Lorsque l’état de migration de la base de données est **Terminé**, connectez vos applications à la nouvelle instance Azure Database pour PostgreSQL cible.

 > [!NOTE]
 > Comme PostgreSQL a par défaut schéma.table.colonne en minuscules, vous pouvez rétablir les minuscules à partir des majuscules en utilisant le script indiqué dans la section **Configurer le schéma dans Azure Database pour PostgreSQL**, plus haut dans cet article.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les problèmes connus et les limitations lors de l’exécution de migrations en ligne vers Azure Database pour PostgreSQL, consultez l’article [Known issues and workarounds with Azure Database for PostgreSQL online migrations](known-issues-azure-postgresql-online.md) (Problèmes connus et solutions de contournement pour les migrations Azure Database pour PostgreSQL en ligne).
* Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).
* Pour obtenir des informations sur Azure Database pour PostgreSQL, consultez l’article [Qu’est-ce qu’Azure Database pour PostgreSQL ?](../postgresql/overview.md).