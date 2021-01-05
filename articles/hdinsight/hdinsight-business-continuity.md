---
title: Continuité de l’activité Azure HDInsight
description: Cet article offre une vue d’ensemble des meilleures pratiques, de la disponibilité dans une seule région et des options d’optimisation pour la planification de la continuité de l’activité Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
keywords: haute disponibilité hadoop
ms.service: hdinsight
ms.topic: conceptual
ms.date: 10/08/2020
ms.openlocfilehash: f83f5aec264aeae1a729e81932843825a0ce6673
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546940"
---
# <a name="azure-hdinsight-business-continuity"></a>Continuité de l’activité Azure HDInsight

Les clusters Azure HDInsight dépendent de nombreux services Azure tels que le stockage, les bases de données, Active Directory, Active Directory Domain Services, la mise en réseau et Key Vault. Une application d’analytique bien conçue, hautement disponible et tolérante aux pannes doit être conçue avec suffisamment de redondance pour résister à des interruptions régionales ou locales dans un ou plusieurs de ces services. Cet article offre une vue d’ensemble des meilleures pratiques, de la disponibilité dans une seule région et des options d’optimisation pour la planification de la continuité de l’activité.

## <a name="general-best-practices"></a>Bonnes pratiques générales

Cette section décrit quelques-unes des meilleures pratiques à prendre en compte lors de la planification de la continuité de l’activité.

* Déterminez les fonctionnalités métier minimales dont vous aurez besoin en cas d’urgence et les raisons. Par exemple, évaluez si vous avez besoin de capacités de basculement pour la couche de transformation des données (en jaune) *et* la couche de service des données (en bleu), ou si vous avez uniquement besoin d’un basculement pour la couche de service de données.

   :::image type="content" source="media/hdinsight-business-continuity/data-layers.png" alt-text="Couches de transformation des données et de service des données":::

* Segmentez vos clusters en fonction de la charge de travail, du cycle de vie de développement et des départements. Le fait de disposer d’un plus grand nombre de clusters réduit les risques qu’une seule grande défaillance nuise à plusieurs processus métier différents.

* Rendez vos régions secondaires accessibles en lecture seule. Les régions de basculement disposant à la fois de capacités de lecture et d’écriture peuvent conduire à des architectures complexes.

* Les clusters temporaires sont plus faciles à gérer en cas d’urgence. Concevez vos charges de travail de façon à ce que les clusters puissent être cycliques et qu’aucun état ne soit conservé dans les clusters.

* Souvent, les charges de travail ne sont pas terminées en cas d’urgence et doivent être redémarrées dans la nouvelle région. Concevez vos charges de travail de manière à ce qu’elles soient de nature idempotente.

* Utilisez l’automatisation lors des déploiements de cluster et assurez-vous que les paramètres de configuration de cluster sont scriptés autant que possible pour garantir un déploiement rapide et entièrement automatisé en cas d’urgence.

* Utilisez les outils d’analyse Azure sur HDInsight pour détecter un comportement anormal dans le cluster et définir les notifications d’alerte correspondantes. Vous pouvez déployer les solutions de gestion de cluster HDInsight préconfigurées qui collectent des métriques de performances importantes du type de cluster spécifique. Pour plus d’informations, consultez [Azure Monitor pour HDInsight](./hdinsight-hadoop-oms-log-analytics-tutorial.md).  

* Abonnez-vous aux alertes d’intégrité Azure pour être informé des problèmes de service, de la maintenance planifiée et des conseils en matière d’intégrité et de sécurité pour un abonnement, un service ou une région. Les notifications d’intégrité qui incluent la cause du problème et une estimation de l’heure de résolution vous aident à mieux exécuter le basculement et les restaurations. Pour plus d’informations, consultez la [documentation Azure Service Health](../service-health/index.yml).

## <a name="single-region-availability"></a>Disponibilité dans une seule région

Un système HDInsight de base dispose des composants suivants. Tous les composants ont leurs propres mécanismes de tolérance de panne dans une seule région.

* Calcul (machines virtuelles) : Cluster Azure HDInsight
* Metastore(s) : Azure SQL Database
* Stockage : Azure Data Lake Gen2 ou Stockage Blob
* Authentification : Azure Active Directory, Azure Active Directory Domain Services, Pack Sécurité Entreprise
* Résolution de noms de domaine : Azure DNS

Il existe d’autres services facultatifs qui peuvent être utilisés, comme Azure Key Vault et Azure Data Factory.

:::image type="content" source="media/hdinsight-business-continuity/hdinsight-components.png" alt-text="Couches de transformation des données et de service des données":::

### <a name="azure-hdinsight-cluster-compute"></a>Cluster Azure HDInsight (calcul)

HDInsight propose un contrat SLA de disponibilité de 99,9 %. Pour assurer la haute disponibilité dans un seul déploiement, HDInsight est accompagné de nombreux services qui sont en mode haute disponibilité par défaut. Les mécanismes de tolérance de panne dans HDInsight sont fournis par les services de haute disponibilité de Microsoft et de l’écosystème OSS Apache.

Les services suivants sont conçus pour être hautement disponibles :

#### <a name="infrastructure"></a>Infrastructure

* Nœuds principaux actif et de secours
* Plusieurs nœuds de passerelle
* Trois nœuds de quorum Zookeeper
* Nœuds Worker distribués par des domaines d’erreur et de mise à jour

#### <a name="service"></a>Service

* Serveur Apache Ambari
* Serveurs de chronologie d’application pour YARN
* Serveur d’historique des travaux pour Hadoop MapReduce
* Apache Livy
* HDFS
* Gestionnaire de ressources YARN
* HBase Master

Pour en savoir plus, reportez-vous à la documentation relative aux [services de haute disponibilité pris en charge par Azure HDInsight](./hdinsight-high-availability-components.md).

Il n’y a pas toujours besoin d’un événement catastrophique pour nuire aux fonctionnalités métier. Les incidents de service dans un ou plusieurs des services suivants dans une même région peuvent également entraîner une perte des fonctionnalités métier attendues.

### <a name="hdinsight-metastore"></a>Metastore HDInsight

HDInsight utilise [Azure SQL Database](https://azure.microsoft.com/support/legal/sla/sql-database/v1_4/) comme metastore, qui propose un contrat SLA de 99,99 %. Trois réplicas de données persistent dans un centre de données à l’aide de la réplication synchrone. En cas de perte d’un réplica, un autre réplica prend le relais en toute transparence. La [géoréplication active](../azure-sql/database/active-geo-replication-overview.md) est prise en charge dès le départ avec un maximum de quatre centres de données. En cas de basculement, manuel ou initié par le centre de données, le premier réplica de la hiérarchie devient automatiquement accessible en lecture-écriture. Pour plus d’informations, consultez [Continuité de l’activité Azure SQL Database](../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md).

### <a name="hdinsight-storage"></a>Stockage HDInsight

HDInsight recommande Azure Data Lake Storage Gen2 comme couche de stockage sous-jacente. [Stockage Azure](https://azure.microsoft.com/support/legal/sla/storage/v1_5/), notamment Azure Data Lake Storage Gen2, propose un contrat SLA de 99,9 %. HDInsight utilise le service LRS dans lequel trois réplicas de données sont conservés dans un centre de données et où la réplication est synchrone. En cas de perte d’un réplica, un autre réplica prend le relais en toute transparence.

### <a name="azure-active-directory"></a>Azure Active Directory

[Azure Active Directory](https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/) propose un contrat SLA de 99,9 %. Active Directory est un service mondial avec plusieurs niveaux de redondance interne et de récupération automatique. Pour plus d’informations, consultez la façon dont [Microsoft améliore continuellement la fiabilité d’Azure Active Directory](https://azure.microsoft.com/blog/advancing-azure-active-directory-availability/).

### <a name="azure-active-directory-domain-services-ad-ds"></a>Azure Active Directory Domain Services (AD DS)

[Azure Active Directory Domain Services](https://azure.microsoft.com/support/legal/sla/active-directory-ds/v1_0/) propose un contrat SLA de 99,9 %. Azure AD DS est un service hautement disponible hébergé dans des centres de données répartis dans le monde entier. Les jeux de réplicas sont une fonctionnalité d’évaluation dans Azure AD DS qui permet la récupération d’urgence géographique si une région Azure est mise hors connexion. Pour en savoir plus, consultez [Concepts et fonctionnalités des jeux de réplicas pour Azure Active Directory Domain Services](../active-directory-domain-services/concepts-replica-sets.md).  

### <a name="azure-dns"></a>Azure DNS

[Azure DNS](https://azure.microsoft.com/support/legal/sla/dns/v1_1/) propose un contrat SLA de 100 %. HDInsight utilise Azure DNS à différents emplacements pour la résolution de noms de domaine.

## <a name="multi-region-cost-and-complexity-optimizations"></a>Optimisation des coûts et de la complexité dans plusieurs régions

L’amélioration de la continuité de l’activité à l’aide de la récupération d’urgence à haute disponibilité entre les régions nécessite des conceptions architecturales plus complexes et plus coûteuses. Les tableaux suivants décrivent certains domaines techniques qui peuvent augmenter le coût total de possession.

### <a name="cost-optimizations"></a>Optimisation des coûts

|Domaine|Cause de la hausse des coûts|Stratégies d’optimisation|
|----|------------------------|-----------------------|
|Stockage des données|Duplication des données/tables primaires dans une région secondaire|Répliquer uniquement les données organisées|
|Sortie de données|Les transferts de données interrégions sortants ont un prix. Voir les directives relatives à la tarification de la bande passante|Répliquer uniquement les données organisées pour réduire l’empreinte de sortie de la région|
|Calcul de cluster|Clusters HDInsight supplémentaires dans la région secondaire|Utiliser des scripts automatisés pour déployer le calcul secondaire après l’échec du calcul primaire. Utiliser la mise à l’échelle automatique pour réduire au minimum la taille des clusters secondaires. Utiliser des SKU de machine virtuelle moins chers. Créer des clusters secondaires dans les régions où les SKU de machine virtuelle peuvent faire l’objet d’une remise|
|Authentification |Les scénarios multi-utilisateurs dans la région secondaire nécessitent des installations supplémentaires d’Azure AD DS|Éviter les installations multi-utilisateurs dans la région secondaire|

### <a name="complexity-optimizations"></a>Optimisations de la complexité

|Domaine|Cause de la hausse de la complexité|Stratégies d’optimisation|
|----|------------------------|-----------------------|
|Modèles Lecture/Écriture |Obligation de permettre la lecture et l’écriture dans les régions primaires et secondaires |Concevoir la région secondaire pour qu’elle soit en lecture seule|
|RPO et RTO zéro |Obligation de zéro perte de données (RPO = 0) et zéro temps d’arrêt (RTO = 0) |Concevoir RPO et RTO de manière à réduire le nombre de composants qui doivent basculer|
|Fonctionnalités métier |Obligation d’avoir toutes les fonctionnalités métier de la région primaire dans la région secondaire |Évaluer s’il est possible de fonctionner avec un sous-ensemble critique minimal des fonctionnalités métier dans la région secondaire|
|Connectivité |Obligation pour tous les systèmes en amont et en aval de la région primaire de se connecter également à la région secondaire|Limiter la connectivité de la région secondaire à un sous-ensemble critique minimal|

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les fonctionnalités présentées dans cet article, voir :

* [Architectures de continuité de l’activité Azure HDInsight](./hdinsight-business-continuity-architecture.md)
* [Étude de cas d’architecture de solution hautement disponible Azure HDInsight](./hdinsight-high-availability-case-study.md)
* [Présentation d’Apache Hive et HiveQL sur Azure HDInsight](./hadoop/hdinsight-use-hive.md)