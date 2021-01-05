---
title: Migrer des données vers un compte d’API SQL Azure Cosmos DB à l’aide de Striim
description: Découvrez comment utiliser Striim pour migrer des données d’une base de données Oracle vers un compte d’API SQL Azure Cosmos DB.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 07/22/2019
ms.author: sngun
ms.reviewer: sngun
ms.openlocfilehash: 136853182e353ad5cd71981db5935fc3babe162e
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339594"
---
# <a name="migrate-data-to-azure-cosmos-db-sql-api-account-using-striim"></a>Migrer des données vers un compte d’API SQL Azure Cosmos DB à l’aide de Striim
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
 
L’image Striim dans la place de marché Azure fournit un déplacement continu des données en temps réel à partir d’entrepôts de données et de bases de données vers Azure. Lors du déplacement des données, vous pouvez effectuer la dénormalisation en ligne, la transformation des données, activer les analyses en temps réel et les scénarios de création de rapports de données. La prise en main de Striim pour déplacer continuellement des données d’entreprise vers une API SQL Azure Cosmos DB est simple. Azure fournit une offre de place de marché qui facilite le déploiement de Striim et la migration des données vers Azure Cosmos DB. 

Cet article explique comment utiliser Striim pour migrer des données depuis une **base de données Oracle** vers **un compte d’API SQL Azure Cosmos DB**.

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas [d’abonnement Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.

* Vous devez disposer d’une base de données Oracle qui contient des données et s’exécute localement.

## <a name="deploy-the-striim-marketplace-solution"></a>Déployer la solution de place de marché Striim

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Créer une ressource** , puis recherchez **Striim** dans la place de marché Azure. Sélectionnez la première option, puis **Créer**.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/striim-azure-marketplace.png" alt-text="Rechercher l’élément de la place de marché Striim":::

1. Ensuite, entrez les propriétés de configuration de l’instance Striim. L’environnement Striim est déployé sur un ordinateur virtuel. Dans le volet **De base** , entrez le **nom d’utilisateur de la machine virtuelle** et le **mot de passe de la machine virtuelle** (ce mot de passe est utilisé pour la connexion SSH à la machine virtuelle). Sélectionnez votre **Abonnement** , votre **Groupe de ressources** et les **Détails de l’emplacement** où vous souhaitez déployer Striim. Quand vous avez terminé, sélectionnez **OK**.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/striim-configure-basic-settings.png" alt-text="Configurer les paramètres de base pour Striim":::

1. Dans le volet **Paramètres du cluster Striim** , choisissez le type de déploiement Striim et la taille de la machine virtuelle.

   |Paramètre | Valeur | Description |
   | ---| ---| ---|
   |Type de déploiement Striim |Standalone | Striim peut s’exécuter dans un type de déploiement **Autonome** ou **Cluster**. Le mode Autonome déploie le serveur Striim sur une seule machine virtuelle et vous pouvez sélectionner la taille des machines virtuelles en fonction de votre volume de données. Le mode Cluster déploie le serveur Striim sur deux machines virtuelles ou plus selon la taille sélectionnée. Les environnements de cluster avec plus de 2 nœuds offrent une haute disponibilité et un basculement automatiques.</br></br> Dans ce didacticiel, vous pouvez sélectionner l’option Autonome. Utilisez la taille de machine virtuelle par défaut « Standard_F4s ».  | 
   | Nom du cluster Striim|    <Striim_cluster_Name>|  Nom du cluster Striim.|
   | Mot de passe du cluster Striim|   <Striim_cluster_password>|  Mot de passe pour le cluster.|

   Une fois le formulaire rempli, sélectionnez **OK** pour continuer.

1. Dans le volet **Paramètres d’accès à Striim** , configurez l' **Adresse IP publique** (choisissez les valeurs par défaut), le **Nom de domaine pour Striim** et le **Mot de passe d’administrateur** que vous souhaitez utiliser pour vous connecter à l’interface utilisateur Striim. Configurez un réseau virtuel et un sous-réseau (choisissez les valeurs par défaut). Après avoir renseigné les détails, sélectionnez **OK** pour continuer.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/striim-access-settings.png" alt-text="Paramètres d’accès à Striim":::

1. Azure validera le déploiement et s’assurera que tout fonctionne correctement. Ka validation prend quelques minutes. Une fois la validation terminée, sélectionnez **OK**.
  
1. Enfin, passez en revue les conditions d’utilisation et sélectionnez **Créer** pour créer votre instance Striim. 

## <a name="configure-the-source-database"></a>Configurer la base de données source 

Dans cette section, vous configurez la base de données Oracle en tant que source du déplacement des données.  Vous aurez besoin du [Pilote Oracle JDBC](https://www.oracle.com/technetwork/database/features/jdbc/jdbc-ucp-122-3110062.html) pour vous connecter à Oracle. Pour lire les modifications de votre base de données Oracle source, vous pouvez utiliser l’API [LogMiner](https://www.oracle.com/technetwork/database/features/availability/logmineroverview-088844.html) ou [XStream](https://docs.oracle.com/cd/E11882_01/server.112/e16545/xstrm_intro.htm#XSTRM72647). Le pilote Oracle JDBC doit être présent dans le chemin de classe Java de Striim pour la lecture, l’écriture ou la conservation des données à partir de la base de données Oracle.

Téléchargez le pilote [ojdbc8.jar](https://www.oracle.com/technetwork/database/features/jdbc/jdbc-ucp-122-3110062.html) sur votre ordinateur local. Vous l’installerez ultérieurement dans le cluster Striim.

## <a name="configure-the-target-database"></a>Configurer la base de données cible

Dans cette section, vous allez configurer le compte d’API SQL Azure Cosmos DB en tant que cible du déplacement des données.

1. Créez un [compte d’API SQL Azure Cosmos DB](create-cosmosdb-resources-portal.md) à l’aide du Portail Microsoft Azure.

1. Accédez au volet **Explorateur de données** de votre compte Azure Cosmos. Sélectionnez **Nouveau conteneur** pour créer un conteneur. Nous partons du principe que vous migrez les données *products* (produits) et *orders* (commandes) depuis la base de données Oracle vers Azure Cosmos DB. Créez une nouvelle base de données nommée **StriimDemo** avec un conteneur nommé **Orders**. Approvisionnez le conteneur avec **1 000 unités de requête** (cet exemple utilise 1 000 unités de requête, mais vous devez utiliser le débit estimé pour votre charge de travail) et **/ORDER_ID** comme clé de partition. Ces valeurs varient en fonction de vos données sources. 

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/create-sql-api-account.png" alt-text="Créer un compte d’API SQL":::

## <a name="configure-oracle-to-azure-cosmos-db-data-flow"></a>Configurer Oracle pour le flux de données Azure Cosmos DB

1. À présent, revenons à Striim. Avant d’interagir avec Striim, installez le pilote Oracle JDBC que vous avez téléchargé précédemment.

1. Accédez à l’instance Striim que vous avez déployée dans le Portail Microsoft Azure. Sélectionnez le bouton **Se connecter** dans la barre de menus supérieure, puis sous l’onglet **SSH** , copiez l’URL dans le champ **Login using VM local account** (Connexion à l’aide du compte local de machine virtuelle).

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/get-ssh-url.png" alt-text="Récupérer l’URL SSH":::

1. Ouvrez une nouvelle fenêtre de terminal et exécutez la commande SSH que vous avez copiée à partir du Portail Microsoft Azure. Cet article utilise le terminal dans un MacOS, mais vous pouvez suivre les instructions similaires pour l’utilisation de PuTTy ou d’un autre client SSH sur un ordinateur Windows. Lorsque vous y êtes invité tapez **oui** pour continuer, puis entrez le **mot de passe** que vous avez défini pour la machine virtuelle à l’étape précédente.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/striim-vm-connect.png" alt-text="Se connecter à la machine virtuelle Striim":::

1. À présent, ouvrez un nouvel onglet de terminal pour copier le fichier **ojdbc8.jar** que vous avez téléchargé précédemment. Utilisez la commande SCP suivante pour copier le fichier .jar de votre ordinateur local vers le dossier tmp de l’instance Striim en cours d’exécution dans Azure :

   ```bash
   cd <Directory_path_where_the_Jar_file_exists> 
   scp ojdbc8.jar striimdemo@striimdemo.westus.cloudapp.azure.com:/tmp
   ```

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/copy-jar-file.png" alt-text="Copier le fichier .jar depuis l’emplacement de l’ordinateur vers Striim":::

1. Ensuite, revenez à la fenêtre dans laquelle vous avez effectué le SSH vers l’instance Striim et connectez-vous en tant que sudo. Déplacez le fichier **ojdbc8.jar** depuis le répertoire **/tmp** vers le répertoire **lib** de votre instance Striim à l’aide des commandes suivantes :

   ```bash
   sudo su
   cd /tmp
   mv ojdbc8.jar /opt/striim/lib
   chmod +x ojdbc8.jar
   ```

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/move-jar-file.png" alt-text="Déplacer le fichier .jar vers le dossier lib":::


1. À partir de la même fenêtre de terminal, redémarrez le serveur Striim en exécutant les commandes suivantes :

   ```bash
   Systemctl stop striim-node
   Systemctl stop striim-dbms
   Systemctl start striim-dbms
   Systemctl start striim-node
   ```
 
1. Le démarrage de Striim prend une minute. Si vous souhaitez afficher l’état, exécutez la commande suivante : 

   ```bash
   tail -f /opt/striim/logs/striim-node.log
   ```

1. À présent, revenez à Azure et copiez l’adresse IP publique de votre machine virtuelle Striim. 

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/copy-public-ip-address.png" alt-text="Copier l’adresse IP de l’ordinateur virtuel Striim":::

1. Pour accéder à l’interface utilisateur web de Striim, ouvrez un nouvel onglet dans un navigateur et copiez l’adresse IP publique, puis procédez comme suit : 9080. Connectez-vous en utilisant le nom d’utilisateur **admin** , ainsi que le mot de passe d’administrateur que vous avez spécifié dans le Portail Microsoft Azure.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/striim-login-ui.png" alt-text="Connectez-vous à Striim":::

1. Vous arrivez à présent sur la page d’hébergement de Striim. Il existe trois volets différents : **Dashboards** (Tableaux de bord), **Apps** (Applications) et **SourcePreview** (Aperçu de la source). Le volet Dashboards vous permet de déplacer des données en temps réel et de les visualiser. Le volet Apps contient vos pipelines de données de streaming ou de flux de données. Dans la partie droite de la page se trouve SourcePreview, où vous pouvez afficher un aperçu de vos données avant de les déplacer.

1. Sélectionnez le volet **Apps** , nous nous concentrerons sur ce volet pour l’instant. Vous pouvez utiliser un grand nombre d’exemples d’applications pour en savoir plus sur Striim. Toutefois, dans cet article, vous allez créer votre propre application. Sélectionnez le bouton **Add App** (Ajouter une application) dans le coin supérieur droit.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/add-striim-app.png" alt-text="Ajouter l’application Striim":::

1. Il existe plusieurs façons de créer des applications Striim. Sélectionnez **Start with Template** (Démarrer avec un modèle) pour démarrer avec un modèle existant.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/start-with-template.png" alt-text="Démarrer l’application avec un modèle":::

1. Dans le champ **Search templates** (Rechercher des modèles), tapez « Cosmos », puis sélectionnez **Target: Azure Cosmos DB**. Enfin, sélectionnez **Oracle CDC to Azure Cosmos DB** (CDC Oracle vers Azure Cosmos DB).

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/oracle-cdc-cosmosdb.png" alt-text="Sélection Oracle CDC to Cosmos DB":::

1. Dans la page suivante, nommez votre application. Vous pouvez fournir un nom tel que **oraToCosmosDB** , puis sélectionner **Save** (Enregistrer).

1. Ensuite, entrez la configuration source de votre instance Oracle source. Entrez une valeur pour le champ **Source Name** (Nom de la source). Le nom de la source est simplement une convention d’affectation de noms pour l’application Striim. Vous pouvez utiliser un nom tel que **src_onPremOracle**. Entrez des valeurs pour les autres paramètres source : **URL** , **Username** (Nom d’utilisateur), **Password** (Mot de passe), puis sélectionnez **LogMiner** comme lecteur de données depuis Oracle. Sélectionnez **Suivant** pour continuer.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/configure-source-parameters.png" alt-text="Configurer les paramètres de la source":::

1. Striim vérifie votre environnement et s’assure qu’il peut se connecter à votre instance Oracle source, dispose des privilèges appropriés et que CDC a été configuré correctement. Une fois toutes les valeurs validées, sélectionnez **Next** (Suivant).

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/validate-source-parameters.png" alt-text="Valider les paramètres de la source":::

1. Sélectionnez les tables de la base de données Oracle que vous souhaitez migrer. Par exemple, nous allons choisir la table Orders, puis sélectionner **Next** (Suivant). 

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/select-source-tables.png" alt-text="Sélectionner les tables source":::

1. Après avoir sélectionné la table source, vous pouvez effectuer des opérations plus complexes, telles que le mappage et le filtrage. Dans ce cas, vous allez simplement créer un réplica de votre table source dans Azure Cosmos DB. Sélectionnez **Next** (Suivant) pour configurer la cible.

1. Maintenant, nous allons configurer la cible :

   * **Target Name** (Nom de la cible) : fournissez un nom significatif pour la cible. 
   * **Input From** (Entrée depuis) : dans la liste déroulante, sélectionnez le flux d’entrée que vous avez créé dans la configuration Oracle source. 
   * **Collections**  : entrez les propriétés de configuration Azure Cosmos DB cible. La syntaxe des collections est **SourceSchema.SourceTable, TargetDatabase.TargetContainer**. Dans cet exemple, la valeur serait « SYSTEM. ORDERS, StriimDemo. Orders ». 
   * **AccessKey** (Clé d’accès) : la PrimaryKey (clé primaire) de votre compte Azure Cosmos.
   * **ServiceEndpoint** (Point de terminaison du service) : l’URI de votre compte Azure Cosmos, qui se trouve sous la section **Clés** du Portail Microsoft Azure. 

   Sélectionnez **Save** (Enregistrer) et **Next** (Suivant).

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/configure-target-parameters.png" alt-text="Configurer les paramètres cibles":::


1. Ensuite, vous allez accéder au concepteur de flux, où vous pouvez faire glisser et déposer des connecteurs prêts à l’emploi pour créer vos applications de streaming. À ce stade, vous n’allez pas apporter de modifications au flux. Poursuivez et déployez l’application en sélectionnant le bouton **Deploy App** (Déployer l’application).
 
   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/deploy-app.png" alt-text="Déployer l’application":::

1. Dans la fenêtre de déploiement, vous pouvez spécifier si vous souhaitez exécuter certaines parties de votre application sur des parties spécifiques de votre topologie de déploiement. Étant donné que nous exécutons une topologie de déploiement simple via Azure, nous allons utiliser l’option par défaut.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/deploy-using-default-option.png" alt-text="Utilisation de l’option par défaut":::

1. Après le déploiement, vous pouvez afficher un aperçu du flux pour voir le flux de données. Sélectionnez l'icône en forme de **vague** avec un œil à côté. Sélectionnez le bouton **Deployed** (Déployé) dans la barre de menus supérieure, puis sélectionnez **Start App** (Démarrer l’application).

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/start-app.png" alt-text="Démarrer l’application":::

1. En utilisant un lecteur **CDC (Change Data Capture)** (Capture des changements de données), Striim ne sélectionne que les nouvelles modifications apportées à la base de données. Si des données transitent par vos tables sources, vous les verrez. Toutefois, étant donné qu’il s’agit d’une table de démonstration, la source n’est connectée à aucune application. Si vous utilisez un exemple de générateur de données, vous pouvez insérer une chaîne d’événements dans votre base de données Oracle.

1. Vous verrez des données transitant par la plateforme Striim. Striim récupère également toutes les métadonnées associées à votre table, ce qui est utile pour surveiller les données et vous assurer que les données se trouvent sur la cible appropriée.

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/configure-cdc-pipeline.png" alt-text="Configurer le pipeline CDC":::

1. Enfin, connectez-vous à Azure et accédez à votre compte Azure Cosmos. Actualisez l’Explorateur de données : vous pouvez voir que les données sont arrivées.  

   :::image type="content" source="./media/cosmosdb-sql-api-migrate-data-striim/portal-validate-results.png" alt-text="Valider les données migrées dans Azure":::

En utilisant la solution Striim dans Azure, vous pouvez migrer en continu des données vers des Azure Cosmos DB à partir de diverses sources telles que Oracle, Cassandra, MongoDB, etc. vers Azure Cosmos DB. Pour en savoir plus, visitez le [site web Striim](https://www.striim.com/), [téléchargez une version d’essai gratuite de 30 jours de Striim](https://go2.striim.com/download-free-trial), et pour tout problème de configuration du chemin de migration lié à Striim, effectuez une [demande de support](https://go2.striim.com/request-support-striim).

## <a name="next-steps"></a>Étapes suivantes

* Si vous migrez des données vers une API SQL Azure Cosmos DB, veuillez consulter [Comment migrer des données vers l’API Cassandra compte à l’aide de Striim](cosmosdb-cassandra-api-migrate-data-striim.md)

* [Surveiller et déboguer vos données avec les métriques Azure Cosmos DB](use-metrics.md)