---
title: Mettre à l’échelle HiveServer2 sur Azure HDInsight
description: Mettez à l’échelle les clusters HiveServer2 sur Azure HDInsight horizontalement à l’aide de nœuds de périphérie pour augmenter la tolérance de panne et la disponibilité.
services: hdinsight
ms.service: hdinsight
ms.topic: conceptual
ms.reviewer: hrasheed-msft
ms.author: kecheung
author: kcheeeung
ms.date: 08/12/2020
ms.openlocfilehash: d0a0df4791492c1c9f0d600630d723024c46c1b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88226841"
---
# <a name="scale-hiveserver2-on-azure-hdinsight-clusters-for-high-availability"></a>Mettre à l’échelle les clusters HiveServer2 sur Azure HDInsight pour la haute disponibilité

Découvrez comment déployer une instance HiveServer2 supplémentaire dans votre cluster pour augmenter la disponibilité et la distribution de la charge. Lorsque l’augmentation de la taille de nœud principal n’est pas souhaitée, vous pouvez également utiliser des nœuds de périphérie pour déployer HiveServer2. 

> [!NOTE]
> En fonction de votre utilisation, l’augmentation du nombre d’hôtes HiveServer2 peut augmenter le nombre de connexions au metastore Hive. Vérifiez également que votre base de données Azure SQL est correctement dimensionnée.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide, vous devez comprendre l’article suivant :
- [Utiliser des nœuds de périphérie vides sur des clusters Apache Hadoop dans HDInsight](hdinsight-apps-use-edge-node.md)

## <a name="install-hiveserver2"></a>Installer HiveServer2

Dans cette section, vous allez installer une instance HiveServer2 supplémentaire sur vos hôtes cibles.

1. Ouvrez Ambari dans votre navigateur, puis cliquez sur votre hôte cible.

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-a.png" alt-text="Menu des hôtes d’Ambari.":::

2. Cliquez sur le bouton Ajouter, puis sur HiveServer2.

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-b.png" alt-text="Menu des hôtes d’Ambari.":::

3. Confirmez et le processus s’exécutera. Répétez les étapes 1 à 3 pour tous les hôtes souhaités.

4. Une fois l’installation terminée, redémarrez tous les services dont la configuration est obsolète et démarrez HiveServer2.

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-c.png" alt-text="Menu des hôtes d’Ambari.":::

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à installer HiveServer2 sur votre cluster. Pour plus d’informations sur les nœuds de périphérie et les applications, consultez les articles suivants :

* [Installer un nœud de périphérie](hdinsight-apps-use-edge-node.md) : découvrez comment installer un nœud de périphérie sur votre cluster HDInsight.
* [Installer des applications HDInsight](hdinsight-apps-install-applications.md) : découvrez comment installer une application HDInsight sur vos clusters.
* [Limites de connexion de la DTU d’Azure SQL](../azure-sql/database/resource-limits-dtu-single-databases.md) : en savoir plus sur les limites de la base de données Azure SQL à l’aide de la DTU.
* [Limites de connexion de la mémoire à tores magnétiques virtuelle d’Azure SQL](../azure-sql/database/resource-limits-vcore-elastic-pools.md) : en savoir plus sur les limites de la base de données Azure SQL à l’aide des mémoires à tores magnétiques virtuelles.
