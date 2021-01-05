---
title: Étude de cas d’architecture de solution hautement disponible Azure HDInsight
description: Cet article est une étude de cas fictive d’une architecture de solution à haute disponibilité Azure HDInsight possible.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
keywords: haute disponibilité hadoop
ms.service: hdinsight
ms.topic: conceptual
ms.date: 10/08/2020
ms.openlocfilehash: 4b98b03c2d7eb4a0403b4595c1376656ed42511b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91855036"
---
# <a name="azure-hdinsight-highly-available-solution-architecture-case-study"></a>Étude de cas d’architecture de solution hautement disponible Azure HDInsight

Les mécanismes de réplication d’Azure HDInsight peuvent être intégrés dans une architecture de solution hautement disponible. Dans cet article, une étude de cas fictive pour Contoso Retail est utilisée pour expliquer les approches possibles de la récupération d’urgence à haute disponibilité, les considérations relatives aux coûts et les conceptions correspondantes.

Les recommandations de la récupération d’urgence à haute disponibilité peuvent avoir de nombreuses combinaisons et permutations. Ces solutions doivent être choisies après avoir pesé les avantages et inconvénients de chaque option. Cet article aborde uniquement une solution possible.

## <a name="customer-architecture"></a>Architecture du client

L’illustration suivante représente l’architecture principale de Contoso Retail. L’architecture se compose d’une charge de travail de diffusion en continu, d’une charge de travail par lot, d’une couche de consommation, d’une couche de stockage et d’un contrôle de version.

:::image type="content" source="./media/hdinsight-high-availability-case-study/contoso-architecture.png" alt-text="Architecture Contoso Retail":::

### <a name="streaming-workload"></a>Charge de travail de diffusion en continu

Les appareils et capteurs produisent des données vers HDInsight Kafka, qui constitue l’infrastructure de messagerie. Un consommateur HDInsight Spark lit les rubriques Kafka. Spark transforme les messages entrants et les écrit dans un cluster HDInsight HBase sur la couche de service.

### <a name="batch-workload"></a>Charge de travail par lot

Un cluster HDInsight Hadoop exécutant Hive et MapReduce ingère des données à partir de systèmes transactionnels locaux. Les données brutes transformées par Hive et MapReduce sont stockées dans des tables Hive sur une partition logique du lac de données soutenu par Azure Data Lake Storage Gen2. Les données stockées dans les tables Hive sont également mises à la disposition de Spark SQL, qui effectue des transformations par lot avant de stocker les données organisées dans HBase pour les servir.

### <a name="serving-layer"></a>Couche de service

Un cluster HDInsight HBase avec Apache Phoenix est utilisé pour fournir des données à des applications web et à des tableaux de bord de visualisation. Un cluster HDInsight LLAP est utilisé pour répondre aux besoins de création de rapports internes.

### <a name="consumption-layer"></a>Couche de consommation

Une couche de gestion des API et d’applications d’API Azure soutenant une page web publique. Les exigences de création de rapports internes sont satisfaites par Power BI.

### <a name="storage-layer"></a>Couche de stockage

Une instance Azure Data Lake Storage Gen2 partitionnée logiquement est utilisée comme lac de données d’entreprise. Les metastores HDInsight sont soutenus par Azure SQL DB.

### <a name="version-control-system"></a>Système de gestion de version

Un système de gestion de version intégré à Azure Pipelines et hébergé en dehors d’Azure.

## <a name="customer-business-continuity-requirements"></a>Exigences de continuité de l’activité des clients

Il est important de déterminer les fonctionnalités métier minimales dont vous aurez besoin en cas de sinistre.

### <a name="contoso-retails-business-continuity-requirements"></a>Exigences de continuité des activités de Contoso Retail

* Nous devons être protégés contre les défaillances régionales ou les problèmes d’intégrité du service régional.
* Les clients ne doivent jamais voir d’erreur 404. Le contenu public doit toujours être délivré. (RTO = 0)  
* Pour la majeure partie de l’année, nous pouvons afficher du contenu public qui date de 5 heures. (RPO = 5 heures)
* Pendant les fêtes de fin d’année, notre contenu public doit toujours être à jour. (RPO = 0)
* Nos exigences de création de rapports internes ne sont pas considérées comme critiques pour la continuité de l’activité.
* Optimisez les coûts de continuité de l’activité.

## <a name="proposed-solution"></a>Solution proposée

L’illustration suivante montre l’architecture de récupération d’urgence haute disponibilité de Contoso Retail.

:::image type="content" source="./media/hdinsight-high-availability-case-study/contoso-solution.png" alt-text="Architecture Contoso Retail":::

**Kafka** utilise la réplication [Active – Passive](hdinsight-business-continuity-architecture.md#apache-kafka) pour mettre en miroir les rubriques Kafka de la région primaire vers la région secondaire. Une alternative à la réplication Kafka pourrait consister à produire sur Kafka dans les deux régions.

**Hive et Spark** utilisent des modèles de réplication [Primaire active - Secondaire à la demande ](hdinsight-business-continuity-architecture.md#apache-spark) en temps normal. Le processus de réplication Hive s’exécute régulièrement et accompagne la réplication du metastore Hive Azure SQL et du compte de stockage Hive. Le compte de stockage Spark est périodiquement répliqué à l’aide d’ADF DistCP. La nature temporaire de ces clusters permet d’optimiser les coûts. Les réplications sont planifiées toutes les 4 heures pour atteindre un RPO qui est largement dans les cinq heures.

La réplication **HBase** utilise le modèle [Leader - Suiveur](hdinsight-business-continuity-architecture.md#apache-hbase) en temps normal pour s’assurer que les données sont toujours servies indépendamment de la région et que le RPO est égal à zéro.

En cas de défaillance régionale dans la région primaire, la page web et le contenu du backend sont servis à partir de la région secondaire pendant 5 heures, avec un certain degré d’obsolescence. Si le tableau de bord d’intégrité du service Azure n’indique pas une estimation de temps de récupération dans la fenêtre de cinq heures, Contoso Retail crée la couche de transformation Hive et Spark dans la région secondaire, puis fait pointer toutes les sources de données en amont vers la région secondaire. Rendre la région secondaire accessible en écriture provoquerait un processus de restauration qui implique la réplication vers la région principale.

Pendant un pic d’achats, l’ensemble du pipeline secondaire est toujours actif et en cours d’exécution. Les producteurs Kafka produisent dans les deux régions, et la réplication HBase est changée de Leader -Suiveur à Leader - Leader pour s’assurer que le contenu public est toujours à jour.

Aucune solution de basculement ne doit être conçue pour la création de rapports internes, car elle n’est pas essentielle à la continuité de l’activité.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les fonctionnalités présentées dans cet article, voir :

* [Continuité d’activité Azure HDInsight](./hdinsight-business-continuity.md)
* [Architectures de continuité de l’activité Azure HDInsight](./hdinsight-business-continuity-architecture.md)
* [Présentation d’Apache Hive et HiveQL sur Azure HDInsight](./hadoop/hdinsight-use-hive.md)