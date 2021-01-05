---
title: Résoudre les problèmes de création de cluster avec Azure HDInsight
description: Découvrez comment résoudre les problèmes de création de cluster Apache pour Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: troubleshooting
ms.date: 04/14/2020
ms.openlocfilehash: b8be230044d868cc3ec03f6dc3fc2d21e102f121
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91856294"
---
# <a name="troubleshoot-cluster-creation-failures-with-azure-hdinsight"></a>Résoudre les problèmes de création de cluster avec Azure HDInsight

Les problèmes suivants sont les causes principales les plus courantes des échecs de création de clusters :

- Problèmes d’autorisation
- Restrictions des stratégies de ressources
- Pare-feux
- Verrous de ressources
- Versions de composants non prises en charge
- Restrictions des noms des comptes de stockage
- Interruptions de service

## <a name="permissions-issues"></a>Problèmes d’autorisations

Si vous utilisez Azure Data Lake Storage Gen2 et que vous recevez le message d’erreur `AmbariClusterCreationFailedErrorCode` : « :::no-loc text="Internal server error occurred while processing the request. Please retry the request or contact support."::: », ouvrez le portail Azure, accédez à votre compte de stockage et, sous Contrôle d’accès (IAM), assurez-vous que le rôle **Contributeur aux données blob du stockage** ou **Propriétaire des données blob du stockage** dispose d’un accès affecté à l’**identité managée affectée par l’utilisateur** pour l’abonnement. Pour plus d’informations, consultez [Définir les autorisations de l’identité managée sur Data Lake Storage Gen2](../hdinsight-hadoop-use-data-lake-storage-gen2-portal.md#set-up-permissions-for-the-managed-identity-on-the-data-lake-storage-gen2).

Si vous utilisez Azure Data Lake Storage Gen1, consultez les instructions d’installation et de configuration [Utiliser Azure Data Lake Storage Gen1 avec des clusters Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen1.md). Data Lake Storage Gen1 n’est pas pris en charge pour les clusters HBase ni dans HDInsight version 4.0.

Si vous utilisez le stockage Azure, vérifiez que le nom du compte de stockage est valide pendant la création du cluster.

## <a name="resource-policy-restrictions"></a>Restrictions des stratégies de ressources

Des stratégies Azure basées sur un abonnement peuvent refuser la création d’adresses IP publiques. La création du cluster HDInsight requiert deux adresses IP publiques.  

En général, les stratégies suivantes peuvent affecter la création du cluster :

* Stratégies empêchant la création d’adresses IP et d’équilibreurs de charge au sein de l’abonnement.
* Stratégie empêchant la création d’un compte de stockage.
* Stratégie empêchant la suppression de ressources réseau (adresse IP/équilibreurs de charge).

## <a name="firewalls"></a>Pare-feux

Les pare-feu de votre réseau virtuel ou de votre compte de stockage peuvent refuser la communication avec les adresses IP de gestion HDInsight.

Autorisez le trafic à partir des adresses IP dans le tableau ci-dessous.

| Adresse IP source | Destination | Sens |
|---|---|---|
| 168.61.49.99 | *:443 | Trafic entrant |
| 23.99.5.239 | *:443 | Trafic entrant |
| 168.61.48.131 | *:443 | Trafic entrant |
| 138.91.141.162 | *:443 | Trafic entrant |

Ajoutez également les adresses IP spécifiques à la région où le cluster est créé. Consultez [Adresses IP de gestion HDInsight](../hdinsight-management-ip-addresses.md) pour obtenir la liste des adresses de chaque région Azure.

Si vous utilisez ExpressRoute ou votre propre serveur DNS personnalisé, consultez [Planifier un réseau virtuel pour Azure HDInsight, Connexion de plusieurs réseaux](../hdinsight-plan-virtual-network-deployment.md#multinet).

## <a name="resources-locks"></a>Verrous de ressources  

Vérifiez qu’il n’y aucun [verrou sur votre réseau virtuel ou groupe de ressources](../../azure-resource-manager/management/lock-resources.md). Les clusters ne peuvent pas être créés ou supprimés si le groupe de ressources est verrouillé. 

## <a name="unsupported-component-versions"></a>Versions de composants non prises en charge

Vérifiez que vous utilisez une [version prise en charge d’Azure HDInsight](../hdinsight-component-versioning.md) et des [composants Apache Hadoop](../hdinsight-component-versioning.md#apache-components-available-with-different-hdinsight-versions) dans votre solution.  

## <a name="storage-account-name-restrictions"></a>Restrictions des noms des comptes de stockage

Les noms de compte de stockage ne peuvent pas compter plus de 24 caractères ni contenir de caractères spéciaux. Ces restrictions s’appliquent également au nom de conteneur par défaut dans le compte de stockage.

D’autres restrictions concernant l’attribution de noms s’appliquent également à la création du cluster. Pour plus d’informations; consultez [Restrictions des noms de cluster](../hdinsight-hadoop-provision-linux-clusters.md#cluster-name).

## <a name="service-outages"></a>Interruptions de service

Pour les éventuels problèmes de service ou interruptions, vérifiez l’[état Azure](https://status.azure.com).

## <a name="next-steps"></a>Étapes suivantes

* [Étendre HDInsight à l’aide d’un Réseau virtuel Azure](../hdinsight-plan-virtual-network-deployment.md)
* [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md)  
* [Utiliser Stockage Azure avec des clusters Azure HDInsight](../hdinsight-hadoop-use-blob-storage.md)
* [Configurer des clusters dans HDInsight avec Apache Hadoop, Apache Spark, Apache Kafka, etc.](../hdinsight-hadoop-provision-linux-clusters.md)
