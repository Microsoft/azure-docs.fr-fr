---
title: Composants et versions Apache Hadoop – Azure HDInsight
description: Découvrez les composants et versions d’Apache Hadoop dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 08/13/2020
ms.openlocfilehash: 82297b81ebe99301574e99ef438cc81c711dbeff
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95533682"
---
# <a name="apache-components-and-versions-available-for-azure-hdinsight"></a>Composants et versions d’Apache disponibles pour Azure HDInsight

Dans cet article, vous allez découvrir les composants et versions de l’environnement [Apache Hadoop](https://hadoop.apache.org/) inclus dans Azure HDInsight et le Pack Sécurité Entreprise. Vous allez également apprendre à vérifier les versions des composants Hadoop dans HDInsight.

## <a name="apache-components-available-with-different-hdinsight-versions"></a>Composants d’Apache disponibles avec différentes versions de HDInsight

Azure HDInsight prend en charge plusieurs versions de cluster Hadoop qui peuvent être déployées à tout moment. Depuis le 4 avril 2017, la version de cluster par défaut utilisée par Azure HDInsight est la version 3.6.

Les versions de composants associées aux versions de cluster HDInsight sont répertoriées dans le tableau suivant.

> [!NOTE]
> La version par défaut associée au service HDInsight peut changer sans préavis. Si vous dépendez d’une version HDInsight, spécifiez cette dernière lorsque vous créez vos clusters à l’aide du Kit de développement logiciel (SDK) .NET avec Azure PowerShell et l’interface CLI classique d’Azure.

| Composant              | HDInsight 4.0 | HDInsight 3.6 (par défaut)     |
|------------------------|---------------|-----------------------------|
| Apache Hadoop et YARN | 3.1.1         | 2.7.3                       |
| Apache Tez             | 0.9.1         | 0.7.0                       |
| Apache Pig             | 0.16.1        | 0.16.0                      |
| Apache Hive            | 3.1.0         | 1.2.1 (2.1.0 sur ESP Interactive Query) |
| Apache Tez/Hive2       | -             | 0.8.4                       |
| Apache Ranger          | 1.1.0         | 0.7.0                       |
| Apache HBase           | 2.1.6         | 1.1.2                       |
| Apache Sqoop           | 1.4.7         | 1.4.6                       |
| Apache Oozie           | 4.3.1         | 4.2.0                       |
| Apache Zookeeper       | 3.4.6         | 3.4.6                       |
| Apache Storm           | -             | 1.1.0                       |
| Apache Mahout          | -             | 0.9.0+                      |
| Apache Phoenix         | 5             | 4.7.0                       |
| Apache Spark           | 2.4.4         | 2.3.2.                      |
| Apache Livy            | 0.5           | 0.4.                        |
| Apache Kafka           | 2.1.1         | 1.1                         |
| Apache Ambari          | 2.7.0         | 2.6.0                       |
| Apache Zeppelin        | 0.8.0         | 0.7.3                       |
| Mono                   | 4.2.1         | 4.2.1                       |

> [!NOTE]
> HDInsight 4.0 : Spark 2.4 et Kafka 2.1 sont entièrement pris en charge. Toutefois, les types de cluster Spark 2.3 et Kafka 1.1 ne sont pas pris en charge. HDInsight 3.6 : Spark 2.3 et Kafka 1.1 sont entièrement pris en charge.  

## <a name="check-for-current-apache-component-version-information"></a>Rechercher des informations sur les versions actuelles des composants Apache

Les versions de composant d’environnement Hadoop associées aux versions de cluster HDInsight peuvent changer avec les mises à jour de HDInsight. Pour vérifier les composants Hadoop ainsi que les versions utilisées pour un cluster, utilisez l’API REST Ambari. La commande **GetComponentInformation** extrait des informations sur les composants du service. Pour plus d’informations, consultez la [documentation d’Apache Ambari](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md).

### <a name="release-notes"></a>Notes de publication

Pour d’autres notes de publication sur les dernières versions de HDInsight, consultez la page [Notes de publication de HDInsight](hdinsight-release-notes.md) .

## <a name="supported-hdinsight-versions"></a>Versions de HDInsight prises en charge

### <a name="support-expiration-and-retirement-for-hdinsight-versions"></a>Expiration du support et mise hors service des versions de HDInsight

L’**expiration du support** signifie que Microsoft ne prend plus en charge la version spécifique de HDInsight. Celle-ci ne sera plus disponible par le biais du portail Azure pour la création de cluster. Ces versions peuvent toujours être créées à l’aide d’Azure CLI ou des différents Kits de développement logiciel (SDK).

La **mise hors service** signifie que les clusters existants d’une version de HDInsight continuent à s’exécuter tels quels. Il n’est pas possible de créer des clusters de cette version par quelque moyen que ce soit (CLI et SDK notamment). D’autres fonctionnalités du plan de contrôle, comme la mise à l’échelle manuelle ou automatique, peuvent également ne plus fonctionner après la mise hors service d’une version. Aucun support n’est disponible pour les versions mises hors service.

Les tableaux suivants listent les versions HDInsight. Les dates d’expiration du support et de mise hors service sont également fournies lorsqu’elles sont connues.

### <a name="available-versions"></a>Versions disponibles

Ce tableau répertorie les versions de HDInsight qui sont disponibles dans le portail Azure ainsi que par d’autres méthodes de déploiement telles que PowerShell et le Kit de développement logiciel (SDK) .NET.

| Version de HDInsight | SYSTÈME D’EXPLOITATION DE LA MACHINE VIRTUELLE | Date de publication | Date d’expiration du support | Date de mise hors service | Haute disponibilité |  Disponibilité dans le portail Azure |
| --- | --- | --- | --- | --- | --- | --- |
| HDInsight 4.0 |Ubuntu 16.0.4 LTS |24 septembre 2018 | | |Oui |Oui |
| HDInsight 3.6 |Ubuntu 16.0.4 LTS |4 avril 2017      | *30 juin 2021 |30 juin 2021 |Oui |Oui |

*Nous étendons le délai de prise en charge de certains types de cluster HDInsight 3.6

| Type du cluster                    | Version du framework | Date d’expiration actuelle de la prise en charge        | Nouvelle date d’expiration de la prise en charge |
|---------------------------------|-------------------|-----------------------------------|-----------------------------|
| HDInsight 3.6 Hadoop            | 2.7.3             | 31 décembre 2020                      | 30 juin 2021               |
| HDInsight 3.6 Spark             | 2.3               | 31 décembre 2020                      | 30 juin 2021               |
| HDInsight 3.6 Spark             | 2,2               | Supprimé le 30 juin 2020          |                             |
| HDInsight 3.6 Spark             | 2.1               | Supprimé le 30 juin 2020          |                             |
| HDInsight 3.6 Kafka             | 1,1               | 31 décembre 2020                      | 30 juin 2021               |
| HDInsight 3.6 Kafka             | 1.0               | Supprimé le 30 juin 2020         |                             |
| HDInsight 3.6 HBase             | 1,1               | 31 décembre 2020                      | 30 juin 2021               |
| HDInsight 3.6 Interactive Query | 2.1               | 31 décembre 2020                      | 30 juin 2021               |
| HDInsight 3.6 Storm             | 1,1               | 31 décembre 2020                      | 30 juin 2021               |
| HDInsight 3.6  ML Services      | 9.3               | 31 décembre 2020                      | 31 décembre 2020                |

> [!NOTE]
> Une fois que la prise en charge d’une version de cluster a expiré, il arrive que celle-ci ne soit plus disponible via le portail Azure. Dans certains cas, les versions de clusters restent disponibles à l’aide du paramètre **Version** de la commande Windows PowerShell [New-AzHDInsightCluster](/powershell/module/az.hdinsight/new-azhdinsightcluster) et du kit SDK .NET jusqu’à leur date de mise hors service.

### <a name="retired-versions"></a>Versions supprimées

Ce tableau répertorie les versions de HDInsight qui ne sont pas disponibles dans le portail Azure.

| Version de HDInsight | Version de la plateforme HDP | SYSTÈME D’EXPLOITATION DE LA MACHINE VIRTUELLE | Date de publication | Date d’expiration du support | Date de mise hors service | Haute disponibilité |  Disponibilité dans le portail Azure |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HDInsight 3.5 |HDP 2.5 |Ubuntu 16.0.4 LTS |30 septembre 2016 |5 septembre 2017 |28 juin 2018 |Oui |Non |
| HDInsight 3.4 |HDP 2.4 |Ubuntu 14.0.4 LTS |29 mars 2016 |29 décembre 2016 |9 janvier 2018 |Oui |Non |
| HDInsight 3.3 |HDP 2.3 |Windows Server 2012 R2 |2 décembre 2015 |27 juin 2016 |31 juillet 2018 |Oui |Non |
| HDInsight 3.3 |HDP 2.3 |Ubuntu 14.0.4 LTS |2 décembre 2015 |27 juin 2016 |31 juillet 2017 |Oui |Non |
| HDInsight 3.2 |HDP 2.2 |Ubuntu 12.04 LTS ou Windows Server 2012 R2 |18 février 2015 |1 mars 2016 |1 avril 2017 |Oui |Non |
| HDInsight 3.1 |HDP 2,1 |Windows Server 2012 R2 |24 juin 2014 |18 mai 2015 |30 juin 2016 |Oui |Non |
| HDInsight 3.0 |HDP 2,0 |Windows Server 2012 R2 |11 février 2014 |17 septembre 2014 |30 juin 2015 |Oui |Non |
| HDInsight 2.1 |HDP 1,3 |Windows Server 2012 R2 |28 octobre 2013 |12 mai 2014 |31 mai 2015 |Oui |Non |
| HDInsight 1.6 |HDP 1.1 | |28 octobre 2013 |26 avril 2014 |31 mai 2015 |Non |Non |

> [!NOTE]
> Les clusters à haute disponibilité avec deux nœuds principaux sont déployés par défaut pour les clusters HDInsight 2.1 et versions ultérieures. Ils ne sont pas disponibles pour les clusters HDInsight version 1.6.

## <a name="service-level-agreement-for-hdinsight-cluster-versions"></a>Contrat de niveau de service pour les versions de cluster HDInsight

Le contrat de niveau de service est défini en termes de _fenêtre de support_. Une fenêtre de support désigne la durée pendant laquelle une version de HDInsight est prise en charge par le support technique et le service clientèle Microsoft. Si la version est associée à une _date d’expiration du support_ dépassée, cela signifie que le cluster HDInsight n’est plus inclus dans la fenêtre de support. L’expiration du support d’une version X de HDInsight (lorsqu’une version X+1 plus récente est disponible) est calculée comme suit :

- **Première formule :** ajoutez 180 jours à la date de lancement du cluster HDInsight version X.
- **Deuxième formule :** ajoutez 90 jours à la date de lancement du cluster HDInsight version X+1 dans le portail Azure.

La _date de mise hors service_ est la date après laquelle la version du cluster ne peut plus être créée sur HDInsight. Depuis le 31 juillet 2017, vous ne pouvez plus redimensionner un cluster HDInsight après sa date de mise hors service.

## <a name="default-node-configuration-and-virtual-machine-sizes-for-clusters"></a>Tailles des machines virtuelles et configuration des nœuds par défaut pour les clusters

Pour plus d’informations sur les références SKU de machine virtuelle à sélectionner pour votre cluster, consultez [Détails de configuration du cluster Azure HDInsight](hdinsight-supported-node-configuration.md).

## <a name="next-steps"></a>Étapes suivantes

- [Création de clusters pour Apache Hadoop, Spark, etc. dans HDInsight](hdinsight-hadoop-provision-linux-clusters.md)
- [Pack Sécurité Entreprise](./enterprise-security-package.md)
- [Notes de publication Hortonworks associées aux versions d’Azure HDInsight](./hortonworks-release-notes.md)
- [Travailler à partir d’un PC Windows dans Apache Hadoop sur HDInsight](hdinsight-hadoop-windows-tools.md)
