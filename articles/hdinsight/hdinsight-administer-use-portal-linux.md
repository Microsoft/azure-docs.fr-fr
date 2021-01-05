---
title: Gérer des clusters Apache Hadoop dans HDInsight à l’aide du portail Azure
description: Découvrez comment créer et gérer des clusters Azure HDInsight avec le portail Azure.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 04/24/2020
ms.openlocfilehash: 7c644583e994e51f11bdf7991755a6ea94b01e76
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012803"
---
# <a name="manage-apache-hadoop-clusters-in-hdinsight-by-using-the-azure-portal"></a>Gérer des clusters Apache Hadoop dans HDInsight à l’aide du portail Azure

[!INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

Le [portail Azure](https://portal.azure.com) vous permet de gérer des clusters [Apache Hadoop](https://hadoop.apache.org/) dans Azure HDInsight. Utilisez le sélecteur d’onglets ci-dessus pour obtenir des informations sur la gestion de clusters Hadoop dans HDInsight à l’aide d’autres outils.

## <a name="prerequisites"></a>Prérequis

Un cluster Apache Hadoop existant dans HDInsight.  Consultez [Création de clusters Linux dans HDInsight à l’aide du portail Azure](hdinsight-hadoop-create-linux-clusters-portal.md).

## <a name="getting-started"></a>Mise en route

Connectez-vous à [https://portal.azure.com](https://portal.azure.com).

## <a name="list-and-show-clusters"></a><a name="showClusters"></a> Lister et afficher les clusters

La page **Clusters HDInsight** liste vos clusters existants.  À partir du portail :
1. Dans le menu de gauche, sélectionnez **Tous les services**.
2. Sous **ANALYTICS**, sélectionnez **Clusters HDInsight**.

## <a name="cluster-home-page"></a><a name="homePage"></a> Page d’accueil de cluster

Sélectionnez le nom de votre cluster dans la page [**Clusters HDInsight**](#showClusters).  Cette opération ouvre la **Vue d’ensemble**, qui est similaire à l’image suivante :

![essentials du cluster HDInsight du portail Azure](./media/hdinsight-administer-use-portal-linux/hdinsight-essentials2.png)

**Menu du haut :**  

| Élément| Description |
|---|---|
|Déplacer|déplace le cluster vers un autre groupe de ressources ou un autre abonnement.|
|DELETE|supprime le cluster. |
|Actualiser|Actualise la vue.|

**Menu de gauche :**  

  - **Menu en haut à gauche**

    | Élément| Description |
    |---|---|
    |Vue d’ensemble|fournit des informations générales sur votre cluster.|
    |Journal d’activité|afficher et interroger les journaux d’activité.|
    |Contrôle d’accès (IAM)|utiliser des attributions de rôles.  Consultez [Utiliser les attributions de rôle pour gérer l’accès à vos ressources d’abonnement Azure](../role-based-access-control/role-assignments-portal.md).|
    |Balises|permet de spécifier des paires clé/valeur pour définir une taxonomie personnalisée de vos services cloud. Vous pouvez par exemple créer une clé nommée **projet**, puis utiliser une valeur commune pour tous les services associés à un projet spécifique.|
    |Diagnostiquer et résoudre les problèmes|affiche des informations sur la résolution des problèmes.|
    |Démarrage rapide|affiche des informations qui vous aident à bien démarrer avec HDInsight.|
    |Outils|informations d’aide pour les outils associés à HDInsight.|

  - **Menu Paramètres**  

    | Élément| Description |
    |---|---|
    |Taille du cluster|permet de vérifier, d’augmenter et de réduire le nombre de nœuds Worker du cluster. Voir [Mettre à l’échelle des clusters](hdinsight-administer-use-portal-linux.md#scale-clusters).|
    |Limites de quota|permet d’afficher les cœurs utilisés et disponibles pour votre abonnement.|
    |SSH + Connexion du cluster|affiche les instructions pour se connecter au cluster avec une connexion Secure Shell (SSH). Pour en savoir plus, voir [Utilisation de SSH avec Hadoop Linux sur HDInsight depuis Linux, Unix ou OS X](hdinsight-hadoop-linux-use-ssh-unix.md).|
    |Data Lake Storage Gen 1|permet de configurer l’accès à Data Lake Storage Gen1.  Consultez [Démarrage rapide : Configurer des clusters dans HDInsight](./hdinsight-hadoop-provision-linux-clusters.md).|
    |Comptes de stockage|permet d’afficher les comptes de stockage et les clés. Les comptes de stockage sont configurés pendant le processus de création de cluster.|
    |Applications|permet d’ajouter/supprimer des applications HDInsight.  Consultez [Installer des applications HDInsight personnalisées](hdinsight-apps-install-custom-applications.md).|
    |Actions de script|permet d’exécuter des scripts Bash sur le cluster. Consultez [Personnalisation de clusters HDInsight basés sur Linux à l’aide d’une action de script](hdinsight-hadoop-customize-cluster-linux.md).|
    |Metastores externes|affiche les metastores [Apache Hive](https://hive.apache.org/) et [Apache Oozie](https://oozie.apache.org/). Les metastores ne peuvent être configurés qu’au cours du processus de création de cluster.|
    |Partenaire HDInsight|ajouter/supprimer le partenaire HDInsight actuel.|
    |Propriétés|permet d’afficher les [propriétés du cluster](#properties).|
    |Verrous|permet d’ajouter un verrou pour empêcher la modification ou la suppression du cluster.|
    |Exportation du modèle|permet d’afficher et d’exporter le modèle Azure Resource Manager pour le cluster. Actuellement, vous pouvez exporter uniquement le compte de stockage Azure dépendant. Consultez [Création de clusters Apache Hadoop basés sur Linux dans HDInsight à l’aide de modèles Azure Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md).|

  - **Menu Supervision**

    | Élément| Description |
    |---|---|
    |Alertes|permet de gérer les alertes et les actions.|
    |Mesures|permet de surveiller les mesures du cluster dans les journaux Azure Monitor.|
    |Paramètres de diagnostic|Paramètres de l’emplacement de stockage des métriques de diagnostic.|
    |Azure Monitor|Supervisez votre cluster dans Azure Monitor.|

  - **Menu Support et dépannage**

    | Élément| Description |
    |---|---|
    |Intégrité des ressources|Consultez [Vue d’ensemble d’Azure Resource Health](../service-health/resource-health-overview.md).|
    |Nouvelle demande de support|permet de créer un ticket de support auprès du support Microsoft.|

## <a name="cluster-properties"></a><a name="properties"></a> Propriétés du cluster

Dans la [page d’accueil du cluster](#homePage), sous **Paramètres** sélectionnez **Propriétés**.

|Élément | Description |
|---|---|
|NOM D’HÔTE|Nom du cluster.|
|URL DU CLUSTER|URL de l’interface web Ambari.|
|Point de terminaison privé|point de terminaison privé pour le cluster.|
|Secure Shell (SSH)|nom d’utilisateur et nom d’hôte à utiliser pour accéder au cluster via SSH.|
|STATUT|Valeurs possibles : Abandonné, Accepté, ClusterStorageProvisioned, AzureVMConfiguration, HDInsightConfiguration, En fonctionnement, En cours d’exécution, Erreur, En cours de suppression, Supprimé, TimedOut, DeleteQueued, DeleteTimedOut, DeleteError, PatchQueued, CertRolloverQueued, ResizeQueued ou ClusterCustomization.|
|RÉGION|Emplacement Azure. Pour obtenir la liste des emplacements Azure pris en charge, consultez la zone de liste déroulante **Région** sur [Tarification HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).|
|DATE DE CRÉATION|date à laquelle le cluster a été déployé.|
|SYSTÈME D’EXPLOITATION|**Windows** ou **Linux**.|
|TYPE|Hadoop, HBase, Storm, Spark.|
|Version|Voir [Versions HDInsight](hdinsight-component-versioning.md).|
|Version TLS minimale|Version TLS.|
|ABONNEMENT|Nom de l’abonnement.|
|SOURCE DE DONNÉES PAR DÉFAUT|système de fichiers du cluster par défaut.|
|Tailles des nœuds Worker|taille de machine virtuelle sélectionnée pour les nœuds worker.|
|Taille du nœud principal|taille de machine virtuelle sélectionnée pour les nœuds principaux.|
|Réseau virtuel|nom du réseau virtuel sur lequel le cluster est déployé, si vous en avez sélectionné un au moment du déploiement.|

## <a name="move-clusters"></a>Déplacer des clusters

Vous pouvez déplacer un cluster HDInsight vers un autre groupe de ressources Azure ou un autre abonnement.

Dans la [page d’accueil du cluster](#homePage) :

1. Sélectionnez **Déplacer** dans le menu du haut.
2. Sélectionnez **Déplacer vers un autre groupe de ressources** ou **Déplacer vers un autre abonnement**.
3. Suivez les instructions fournies dans la nouvelle page.

## <a name="delete-clusters"></a>Suppression des clusters

La suppression d’un cluster ne supprime pas le compte de stockage par défaut ni les comptes de stockage liés. Vous pouvez recréer le cluster en utilisant les mêmes comptes de stockage et les mêmes metastores. Il est recommandé d’utiliser un nouveau conteneur d’objets blob par défaut quand vous recréez le cluster.

Dans la [page d’accueil du cluster](#homePage) :

1. Sélectionnez **Supprimer** dans le menu du haut.
2. Suivez les instructions fournies dans la nouvelle page.

Voir aussi [Pause/arrêt de clusters](#pauseshut-down-clusters).

## <a name="add-additional-storage-accounts"></a>Ajouter des comptes de stockage

Vous pouvez ajouter des comptes Stockage Azure et des comptes Azure Data Lake Storage supplémentaires après la création d’un cluster. Pour plus d’informations, consultez [Ajouter des comptes de stockage supplémentaires à HDInsight](./hdinsight-hadoop-add-storage.md).

## <a name="scale-clusters"></a>Mise à l’échelle des clusters

La fonctionnalité de mise à l’échelle d’un cluster vous permet de modifier le nombre de nœuds de travail utilisés par un cluster Azure HDInsight sans avoir à recréer ce cluster.

Pour plus d’informations, consultez la rubrique [Mettre à l’échelle les clusters HDInsight](./hdinsight-scaling-best-practices.md).

## <a name="pauseshut-down-clusters"></a>Pause/arrêt de clusters

La plupart des travaux Hadoop sont les traitements par lots exécutés occasionnellement seulement. Pour la plupart des clusters Hadoop, il existe de longues périodes pendant lequel le cluster n’est pas utilisé pour le traitement. Avec HDInsight, vos données sont stockées dans le stockage Azure. Vous pouvez ainsi supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation.
Vous devez également payer pour un cluster HDInsight, même quand vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, mieux vaut supprimer les clusters quand ils ne sont pas utilisés.

Il existe de nombreuses façons de programmer le processus :

- Utilisateur d’Azure Data Factory. Pour créer des services liés HDInsight à la demande, consultez [Création de clusters Apache Hadoop à la demande basés sur Linux dans HDInsight avec Azure Data Factory](hdinsight-hadoop-create-linux-clusters-adf.md).
- Utilisation d’Azure PowerShell  Voir [Analyse des données sur les retards de vol](./interactive-query/interactive-query-tutorial-analyze-flight-data.md).
- Utiliser l’interface de ligne de commande Microsoft Azure Consultez [Gérer des clusters HDInsight à l’aide d’Azure CLI](hdinsight-administer-use-command-line.md).
- Utilisation du kit de développement logiciel .NET. Voir [Envoyer des tâches Apache Hadoop](hadoop/submit-apache-hadoop-jobs-programmatically.md).

Pour les informations de tarification, consultez [Tarification HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/). Pour supprimer un cluster du portail, consultez [Supprimer les clusters](#delete-clusters)

## <a name="upgrade-clusters"></a>Mettre à niveau des clusters

Consultez [Mettre à niveau le cluster HDInsight](./hdinsight-upgrade-cluster.md).

## <a name="open-the-apache-ambari-web-ui"></a>Ouvrir l’interface utilisateur web d’Apache Ambari

Ambari fournit une interface utilisateur web de gestion Hadoop intuitive et facile à utiliser secondée par ses API RESTful. Ambari permet aux administrateurs système de gérer et de surveiller des clusters Hadoop.

Dans la [page d’accueil du cluster](#homePage) :

1. Sélectionnez **Tableaux de bord du cluster**.

    ![Menu du cluster Apache Hadoop sur HDInsight](./media/hdinsight-administer-use-portal-linux/hdinsight-azure-portal-cluster-menu2.png)

1. Sélectionnez **Accueil Ambari** dans la nouvelle page.
1. Entrez le nom d’utilisateur et le mot de passe du cluster.  Le nom d’utilisateur du cluster par défaut est _admin_.

Pour plus d’informations, consultez [Gérer des clusters HDInsight à l’aide de l’interface utilisateur web d’Apache Ambari](hdinsight-hadoop-manage-ambari.md).

## <a name="change-passwords"></a>Modifier les mots de passe

Un cluster HDInsight peut disposer de deux comptes d'utilisateur. le compte d’utilisateur du cluster HDInsight (compte d’utilisateur HTTP) et le compte d’utilisateur SSH sont créés durant le processus de création. Vous pouvez utiliser le portail pour modifier le mot de passe du compte d’utilisateur du cluster, et des actions de script pour modifier le compte d’utilisateur SSH.

### <a name="change-the-cluster-user-password"></a>Modifier le mot de passe d’utilisateur du cluster

> [!NOTE]  
> La modification du mot de passe d’utilisateur (admin) du cluster peut entraîner l’échec de l’exécution des actions de script sur ce cluster. Si des actions de script persistantes ciblent des nœuds de travail, ces scripts échouent lorsque vous ajoutez des nœuds au cluster par le biais d’opérations de redimensionnement. Pour plus d’informations sur les actions de script, consultez la section [Personnaliser des clusters HDInsight à l’aide d’actions de script](hdinsight-hadoop-customize-cluster-linux.md).

Dans la [page d’accueil du cluster](#homePage) :
1. Sélectionnez **SSH + Connexion du cluster** sous **Paramètres**.
2. Sélectionnez **Réinitialiser les informations d’identification**.
3. Entrez et confirmez le nouveau mot de passe dans les zones de texte.
4. Sélectionnez **OK**.

Le mot de passe est modifié sur tous les nœuds du cluster.

### <a name="change-the-ssh-user-password-or-public-key"></a>Modifier le mot de passe d’utilisateur SSH ou la clé publique

1. À l’aide d’un éditeur de texte, enregistrez le texte suivant dans un fichier nommé **changecredentials.sh**.

    > [!IMPORTANT]  
    > Vous devez utiliser un éditeur qui utilise LF comme caractère de fin de ligne. Si l’éditeur utilise CRLF, le script échoue.

    ```bash
    #! /bin/bash
    USER=$1
    PASS=$2
    usermod --password $(echo $PASS | openssl passwd -1 -stdin) $USER
    ```

2. Chargez le fichier sur un emplacement de stockage accessible à partir de HDInsight à l’aide d’une adresse HTTP ou HTTPS. Par exemple, un magasin de fichiers public tel que le stockage d’objets blob Azure ou OneDrive. Enregistrez l’URI (adresse HTTP ou HTTPS) dans le fichier, car vous en aurez besoin à l’étape suivante.
3. Dans la [page d’accueil du cluster](#homePage), sélectionnez **Actions de script** sous **Paramètres**.
4. En haut de la page **Actions de script**, sélectionnez **Envoyer**.
5. Dans le panneau **Envoyer une action de script**, entrez les informations suivantes :

> [!NOTE]
> Les mots de passe SSH ne peuvent pas contenir les caractères suivants :
> ```
> " ' ` / \ < % ~ | $ & ! 
> ```

   | Champ | Valeur |
   | --- | --- |
   | Type de script | Sélectionnez **- Personnalisé** dans la liste déroulante.|
   | Nom |« Modifier les informations d’identification SSH » |
   | URI de script bash |URI vers le fichier changecredentials.sh |
   | Types de nœud : (Principal, Worker, Nimbus, Superviseur ou Zookeeper) |✓ pour tous les types de nœuds répertoriés |
   | Paramètres |Entrez le nom d’utilisateur SSH, puis le nouveau mot de passe. Il doit y avoir un espace entre le nom d’utilisateur et le mot de passe. |
   | Conservez cette action de script... |Laissez ce champ non coché. |

6. Sélectionnez **Créer** pour appliquer le script. Une fois le script terminé, vous pouvez vous connecter au cluster à l’aide de SSH avec les nouvelles informations d’identification.

## <a name="find-the-subscription-id"></a>Rechercher l’ID d’abonnement

Chaque cluster est lié à un abonnement Azure.  L’ID d’abonnement Azure est visible dans la [page d’accueil du cluster](#homePage).

## <a name="find-the-resource-group"></a>Trouvez le groupe de ressources

En mode Azure Resource Manager, chaque cluster HDInsight est créé avec un groupe de ressources Azure Resource Manager. Le groupe Resource Manager est visible dans la [page d’accueil du cluster](#homePage).

## <a name="find-the-storage-accounts"></a>Rechercher les comptes de stockage

Les clusters HDInsight utilisent un compte Stockage Azure ou Azure Data Lake Storage pour stocker les données. Chaque cluster HDInsight peut avoir un compte de stockage par défaut et une série de comptes de stockage liés. Pour lister les comptes de stockage, dans la [page d’accueil du cluster](#homePage), sous **Paramètres**, sélectionnez **Comptes de stockage**.

## <a name="monitor-jobs"></a>Surveiller des travaux

Consultez [Gérer des clusters HDInsight à l’aide de l’interface utilisateur web d’Apache Ambari](hdinsight-hadoop-manage-ambari.md#monitoring).

## <a name="cluster-size"></a>Taille du cluster

La vignette **Taille du cluster** de la [page d’accueil du cluster](#homePage) affiche le nombre de cœurs alloués à ce cluster et la manière dont ils sont alloués pour les nœuds au sein de ce cluster.

> [!IMPORTANT]  
> Pour surveiller les services fournis par le cluster HDInsight, vous devez utiliser l’interface Ambari Web ou l’API Ambari REST. Pour plus d’informations sur l’utilisation d’Ambari, consultez [Gestion des clusters HDInsight à l’aide d’Apache Ambari](hdinsight-hadoop-manage-ambari.md).

## <a name="connect-to-a-cluster"></a>Se connecter à un cluster

- [Utilisation d’Apache Hive avec HDInsight](hadoop/apache-hadoop-use-hive-ambari-view.md)
- [Utiliser SSH avec HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert certaines fonctions d’administration de base. Pour en savoir plus, consultez les articles suivants :

- [Administration de HDInsight à l’aide d’Azure PowerShell](hdinsight-administer-use-powershell.md)
- [Administration de HDInsight à l’aide de l’interface de ligne de commande Azure](hdinsight-administer-use-command-line.md)
- [Création de clusters HDInsight](hdinsight-hadoop-provision-linux-clusters.md)
- [En savoir plus sur l’utilisation de l’API REST Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md)
- [Utilisation d’Apache Hive dans HDInsight](hadoop/hdinsight-use-hive.md)
- [Utiliser Apache Sqoop dans HDInsight](hadoop/hdinsight-use-sqoop.md)
- [Utiliser des fonctions définies par l’utilisateur (UDF) Python avec Apache Hive et Apache Pig dans HDInsight](hadoop/python-udf-hdinsight.md)
- [Version de Apache Hadoop dans Azure HDInsight](hdinsight-component-versioning.md)