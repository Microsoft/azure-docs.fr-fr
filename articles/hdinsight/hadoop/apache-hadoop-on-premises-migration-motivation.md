---
title: 'Avantages : Effectuer la migration d’Apache Hadoop en local vers Azure HDInsight'
description: Découvrez les avantages et raisons pour lesquelles il est recommandé de migrer des clusters Hadoop locaux vers Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: ashishth
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 11/15/2019
ms.openlocfilehash: 595bf6f921265e9e8dbc0e0e065fe835efea14bc
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92331649"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---motivation-and-benefits"></a>Migrer des clusters Apache Hadoop locaux vers Azure HDInsight : raisons et avantages

Le présent document est le premier article d’une série sur les meilleures pratiques en matière de migration de déploiements d’écosystèmes Apache Hadoop vers Azure HDInsight. Ces articles concernent les personnes responsables de la conception, du déploiement et de la migration des solutions Apache Hadoop vers Azure HDInsight. Les rôles qui peuvent tirer parti de ces articles incluent des architectes cloud, les administrateurs Hadoop et les ingénieurs DevOps. Ils peuvent également aider les développeurs de logiciels, les ingénieurs de données et les scientifiques des données à en savoir plus sur le fonctionnement des différents types de cluster dans le cloud.

## <a name="why-to-migrate-to-azure-hdinsight"></a>Pourquoi effectuer une migration vers Azure HDInsight

Azure HDInsight est une distribution cloud des composants Hadoop. Azure HDInsight rend facile, rapide et économique le traitement de volumes importants de données. HDInsight inclut les infrastructures open source les plus populaires, telles que :

- Apache Hadoop
- Apache Spark
- Apache Hive avec LLAP
- Apache Kafka
- Apache Storm
- Apache HBase
- R

## <a name="azure-hdinsight-advantages-over-on-premises-hadoop"></a>Avantages d’Azure HDInsight par rapport à l’instance Hadoop locale

- **Faible coût** : vous pouvez réduire les coûts en [créant des clusters à la demande](../hdinsight-hadoop-create-linux-clusters-adf.md) et en ne payant que ceux que vous utilisez. En découplant les fonctionnalités de traitement et de stockage, vous bénéficiez d’une certaine flexibilité, car le volume de données ne dépend plus de la taille du cluster.

- **Automatisation de la création de clusters** : la création automatisée des clusters nécessite des opérations d’installation et de configuration réduites. L’automatisation peut être utilisée pour les clusters à la demande.

- **Configuration et matériel managés** : grâce au cluster HDInsight, vous n’avez pas besoin de vous soucier de l’infrastructure ou du matériel physique. Il suffit de spécifier la configuration du cluster : Azure le configure pour vous.

- **Évolution facile** : HDInsight vous permet de[faire monter ou descendre](../hdinsight-administer-use-portal-linux.md) vos charges de travail en puissance. Azure s’occupe de la redistribution des données et du rééquilibrage des charges de travail sans interrompre les travaux de traitement des données.

- **Disponibilité générale**  : HDInsight est disponible dans un plus grand nombre de [régions](https://azure.microsoft.com/regions/services/) que les autres offres d’analytique du Big Data. Azure HDInsight est également disponible dans Azure Government, en Chine, et en Allemagne, ce qui vous permet de répondre aux besoins de votre entreprise dans les principaux domaines souverains.

- **Sécurité et conformité** : HDInsight vous permet de protéger les données de votre entreprise à l’aide du[réseau virtuel Azure](../hdinsight-plan-virtual-network-deployment.md), du[chiffrement](../hdinsight-hadoop-create-linux-clusters-with-secure-transfer-storage.md), et de l’intégration avec [Azure Active Directory](../domain-joined/hdinsight-security-overview.md). HDInsight répond également aux [normes de conformité](https://azure.microsoft.com/overview/trusted-cloud) du gouvernement et du secteur les plus populaires.

- **Gestion simplifiée des versions** : Azure HDInsight gère la version des composants d’écosystème Hadoop et les maintient à jour. Les mises à jour logicielles sont généralement un processus complexe pour les déploiements locaux.

- **Clusters plus petits, optimisés pour des charges de travail spécifiques, avec moins de dépendances entre les composants**  : une installation Hadoop locale classique utilise un seul cluster à de nombreuses fins. Avec Azure HDInsight, vous pouvez créer des clusters spécifiques aux charges de travail. En effet, en créant des clusters pour certaines charges de travail, vous simplifiez la gestion d’un cluster unique toujours plus complexe.

- **Productivité** : vous pouvez utiliser différents outils pour gérer Hadoop et Spark dans votre environnement de développement préféré.

- **Extensibilité grâce à des outils personnalisés ou des applications tierces** : vous pouvez étendre vos clusters HDInsight avec des composants installés, et les intégrer dans d’autres solutions de Big Data, en utilisant des déploiements [en un clic](https://azure.microsoft.com/services/hdinsight/partner-ecosystem/) disponibles sur la Place de marché Microsoft Azure.

- **Gestion, administration et supervision simples** : Azure HDInsight s’intègre aux [journaux Azure Monitor](../hdinsight-hadoop-oms-log-analytics-tutorial.md) pour fournir une interface unique permettant de superviser l’ensemble des clusters.

- **Intégration avec d’autres services Azure** : HDInsight s’intègre facilement avec d’autres services Azure populaires, à savoir :

    - Azure Data Factory (ADF)
    - Stockage Blob Azure
    - Azure Data Lake Storage Gen2
    - Azure Cosmos DB
    - Azure SQL Database
    - Azure Analysis Services

- **Réparation automatique des processus et des composants** : HDInsight vérifie en permanence les composants d’infrastructure et open source à l’aide de sa propre infrastructure de supervision. Il aide automatiquement les systèmes à récupérer après des défaillances critiques, telles que de l’indisponibilité des nœuds et des composants open source. En cas d’échec d’un composant OSS, des alertes sont déclenchées dans Ambari.

Pour plus d’informations, consultez [Présentation de HDInsight et de la pile de technologies Apache Hadoop](../hadoop/apache-hadoop-introduction.md).

## <a name="migration-planning-process"></a>Processus de planification de la migration

Les étapes suivantes sont recommandées pour la planification de la migration de clusters Hadoop locaux vers Azure HDInsight :

1. Comprendre le fonctionnement des topologies et des déploiements locaux actuels
2. Déterminer la portée actuelle du projet actuel, ainsi que les chronologies associées et le savoir-faire de l’équipe
3. Comprendre les exigences Azure
4. Créer un plan détaillé, basé sur les meilleures pratiques

## <a name="gathering-details-to-prepare-for-a-migration"></a>Collecte des détails pour la préparation à la migration

Cette section fournit des modèles de questionnaires, afin de vous aider à collecter des informations importantes sur les éléments suivants :

- Déploiement local
- Détails du projet
- Conditions requises pour Azure

### <a name="on-premises-deployment-questionnaire"></a>Questionnaire relatif au déploiement local

| **Question** | **Exemple** | **Réponse** |
|---|---|---|
|**Rubrique**  : **Environment**|||
|Version de la distribution du cluster|HDP 2.6.5, CDH 5.7|
|Composants de l’écosystème Big Data|HDFS, Yarn, Hive, LLAP, Impala, Kudu, HBase, Spark, MapReduce, Kafka, Zookeeper, Solr, Sqoop, Oozie, Ranger, Atlas, Falcon, Zeppelin, R|
|Types de cluster|Hadoop, Spark, Confluent Kafka, Storm, Solr|
|Nombre de clusters|4|
|Nombre de nœuds principaux|2|
|Nombre de nœuds Worker|100|
|Nombre de nœuds de périphérie| 5|
|Espace disque total|100 To|
|Configuration des nœuds principaux|m/y, UC, disque, etc.|
|Configuration des nœuds de données|m/y, UC, disque, etc.|
|Configuration des nœuds de périphérie|m/y, UC, disque, etc.|
|Chiffrement HDFS ?|Oui|
|Haute disponibilité|HDFS, metastore|
|Récupération d'urgence / sauvegarde|Cluster de sauvegarde ?|  
|Systèmes dépendants du cluster|SQL Server, Teradata, Power BI, MongoDB|
|Intégrations tierces|Tableau, GridGain, Qubole, Informatica, Splunk|
|**Rubrique**  : **Sécurité**|||
|Sécurité du périmètre|Pare-feux|
|Authentification et autorisation du cluster|Active Directory, Ambari, Cloudera Manager, aucune authentification|
|Contrôle d’accès HDFS|  Manuel, utilisateurs de SSH|
|Authentification et autorisation Hive|Sentry, LDAP, Active Directory avec Kerberos, Ranger|
|Audit|Ambari, Cloudera Navigator, Ranger|
|Surveillance|Graphite, collectd, statsd, Telegraf, InfluxDB|
|Génération d’alertes|Kapacitor, Prometheus, Datadog|
|Durée de la période de rétention des données| 3 ans, 5 ans|
|Administrateurs des clusters|Administrateur unique, plusieurs administrateurs|

### <a name="project-details-questionnaire"></a>Questionnaire sur les détails du projet

|**Question**|**Exemple**|**Réponse**|
|---|---|---|
|**Rubrique**  : **Charges de travail et fréquence**|||
|Tâches MapReduce|10 travaux (deux fois par jour)||
|Tâches Hive|100 travaux (toutes les heures)||
|Programmes de traitement par lots Spark|50 travaux (toutes les 15 minutes)||
|Travaux de diffusion en continu Spark|5 travaux (toutes les 3 minutes)||
|Travaux structurés de diffusion en continu|5 travaux (chaque minute)||
|Travaux de formation aux modèles AML|2 travaux (une fois par semaine)||
|Langages de programmation|Python, Scala, Java||
|Création de scripts|Shell, Python||
|**Rubrique**  : **Données**|||
|Sources de données|Fichiers plats, Json, Kafka, SGBDR||
|Orchestration de données|Flux de travail Oozie, Airflow||
|Recherches en mémoire|Apache Ignite, Redis||
|Destination des données|HDFS, SGBDR, Kafka, MPP ||
|**Rubrique**  : **Métadonnées**|||
|Type de base de données Hive|MySQL, Postgres||
|Nombre de metastores Hive|2||
|Nombre de tables Hive|100||
|Nombre de stratégies Ranger|20||
|Nombre de workflows Oozie|100||
|**Rubrique**  : **Mettre à l'échelle**|||
|Volume de données (réplication comprise)|100 To||
|Volume d’ingestion quotidien|50 Go||
|Taux de croissance des données|10 % par an||
|Taux de croissance des nœuds du cluster|5 % par an
|**Rubrique**  : **Utilisation du cluster**|||
|Pourcentage moyen d’utilisation de l’UC|60%||
|Pourcentage moyen d’utilisation de la mémoire|75 %||
|Espace disque utilisé|75 %||
|Pourcentage moyen d’utilisation du réseau|25 %
|**Rubrique**  : **Personnel**|||
|Nombre d’administrateurs|2||
|Nombre de développeurs|10||
|Nombre d’utilisateurs finaux|100||
|Compétences|Hadoop, Spark||
|Nombre de ressources disponibles pour les efforts de migration|2||
|**Rubrique**  : **Limitations**|||
|Limites actuelles|Latence élevée||
|Défis actuels|Problèmes liés à la concurrence||

### <a name="azure-requirements-questionnaire"></a>Questionnaire relatif aux exigences d’Azure

|**Question**|**Exemple**|**Réponse**|
|---|---|---|
|**Rubrique**  : **Infrastructure** |||
| Région recommandée|USA Est||
|Réseau virtuel recommandé ?|Oui||
|HA/récupération d’urgence requise(s) ?|Oui||
|Intégration dans d’autres services cloud ?|ADF, CosmosDB||
|**Rubrique**  :   **Déplacement des données**  |||
|Préférence en matière de charge initiale|DistCp, Data Box, ADF, WANDisco||
|Delta associé au transfert de données|DistCp, AzCopy||
|Transfert continu de données incrémentielles|DistCp, Sqoop||
|**Rubrique**  :   **Supervision et alertes** |||
|Utilisation de fonctions de supervision et de création d’alertes Azure ou intégration d’une fonction de supervision tierce ?|Utiliser la supervision et les alertes Azure||
|**Rubrique**  :   **Préférences de sécurité** |||
|Pipeline de données privé et protégé ?|Oui||
|Cluster joint au domaine (ESP) ?|     Oui||
|Synchronisation Active Directory locale vers le cloud ?|     Oui||
|Nombre d’utilisateurs AD à synchroniser ?|          100||
|Synchroniser les mots de passe sur le cloud ?|    Oui||
|Utilisateurs du cloud uniquement ?|                 Oui||
|MFA requise ?|                       Non|| 
|Exigences relatives aux autorisations associées aux données ?|  Oui||
|Contrôle d’accès en fonction du rôle ?|        Oui||
|Audit nécessaire ?|                  Oui||
|Chiffrement des données au repos ?|          Oui||
|Chiffrement des données en transit ?|       Oui||
|**Rubrique**  :   **Préférences de récréation d’architecture** |||
|Cluster unique ou types de cluster spécifiques ?|Types de cluster spécifiques||
|Stockage à distance ou stockage colocalisé ?|Stockage à distance||
|Clusters plus petits si les données sont stockées à distance ?|Clusters plus petits||
|Utiliser plusieurs clusters plus petits plutôt qu’un seul cluster volumineux ?|Utiliser plusieurs clusters plus petits||
|Utiliser un metastore à distance ?|Oui||
|Partager des metastores entre plusieurs clusters différents ?|Oui||
|Décomposer les charges de travail ?|Remplacer les travaux Hive par des travaux Spark||
|Utiliser ADF pour l’orchestration des données ?|Non||

## <a name="next-steps"></a>Étapes suivantes

Lisez l’article suivant de cette série :

- [Meilleures pratiques en matière d’architecture pour une migration vers Azure HDInsight Hadoop](apache-hadoop-on-premises-migration-best-practices-architecture.md)
