---
title: FAQ Apache Kafka dans Azure HDInsight
description: Obtenez des réponses aux questions courantes sur Apache Kafka sur Azure HDInsight, un service cloud Hadoop géré.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/14/2019
ms.openlocfilehash: 84c48f14ae57cfa4aa3290924d3d170c075fe3a6
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92532536"
---
# <a name="frequently-asked-questions-about-apache-kafka-in-azure-hdinsight"></a>Forum aux questions sur Apache Kafka dans Azure HDInsight

Cet article répond à certaines questions courantes sur l’utilisation d’Apache Kafka sur Azure HDInsight.

## <a name="what-kafka-versions-are-supported-by-hdinsight"></a>Quelles sont les versions de Kafka prises en charge par HDInsight ?

Pour plus d’informations sur les versions et composants HDInsight officiellement pris en charge, consultez [Quels sont les composants et versions Apache Hadoop disponibles avec HDInsight ?](../hdinsight-component-versioning.md#supported-hdinsight-versions). Nous vous recommandons de toujours utiliser la version la plus récente pour garantir des performances et une expérience utilisateur optimales.

## <a name="what-resources-are-provided-in-an-hdinsight-kafka-cluster-and-what-resources-am-i-charged-for"></a>Quelles ressources sont fournies dans un cluster HDInsight Kafka et quelles ressources sont facturées ?

Un cluster Kafka HDInsight comprend les ressources suivantes :

* Nœuds principaux
* Nœuds Zookeeper
* Nœuds de service de répartiteur (worker) 
* Les disques managés Azure attachés aux nœuds de service de répartiteur
* Nœuds de passerelle

Toutes ces ressources sont facturées en fonction de notre [modèle de tarification HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/), à l’exception des nœuds de passerelle. Vous n’êtes pas facturé pour les nœuds de passerelle.

Pour obtenir une description plus détaillée de différents types de nœuds, consultez [Architecture de réseau virtuel Azure HDInsight](../hdinsight-virtual-network-architecture.md). La tarification est basée sur l’utilisation du nœud à la minute. Les prix varient selon la taille du nœud, le nombre de nœuds, le type de disque géré utilisé et la région.

## <a name="do-apache-kafka-apis-work-with-hdinsight"></a>Les API Apache Kafka fonctionnent-elles avec HDInsight ?

Oui, HDInsight utilise les API Kafka natives. Le code de votre application cliente n’a pas besoin d’être modifié. Voir le [tutoriel : Utiliser les API de producteur et de consommateur Apache Kafka](./apache-kafka-producer-consumer-api.md) pour voir comment vous pouvez utiliser les API de producteur/consommateur basées sur Java avec votre cluster.

## <a name="can-i-change-cluster-configurations"></a>Puis-je modifier les configurations de cluster ?

Oui, via le portail Ambari. Chaque composant du portail comporte une section **configs** qui peut être utilisée pour modifier les configurations des composants. Certaines modifications peuvent nécessiter des redémarrages du service de répartiteur.

## <a name="what-type-of-authentication-does-hdinsight-support-for-apache-kafka"></a>Quel type d’authentification HDInsight prend-il en charge pour Apache Kafka ?

Avec le [Package de sécurité d'entreprise (ESP)](../domain-joined/apache-domain-joined-architecture.md), vous pouvez obtenir une sécurité au niveau de la rubrique pour les clusters Kafka. Voir le [tutoriel : Configurer des stratégies Apache Kafka dans HDInsight avec le Pack Sécurité Entreprise (préversion)](../domain-joined/apache-domain-joined-run-kafka.md) pour plus d’informations.

## <a name="is-my-data-encrypted-can-i-use-my-own-keys"></a>Mes données sont-elles chiffrées ? Puis-je utiliser mes propres clés ?

Tous les messages Kafka sur les disques managés sont chiffrés avec [Azure Storage Service Encryption (SSE).](../../storage/common/storage-service-encryption.md) Les données en transit (par exemple, les données transmises par les clients aux répartiteurs et inversement) ne sont pas chiffrées par défaut. Il est possible de chiffrer ce trafic en [configurant TLS vous-même](./apache-kafka-ssl-encryption-authentication.md). En outre, HDInsight vous permet de gérer ses propres clés pour chiffrer les données au repos. Pour plus d'informations, consultez [Chiffrement de disque avec clé gérée par le client](../disk-encryption.md).

## <a name="how-do-i-connect-clients-to-my-cluster"></a>Comment connecter des clients à mon cluster ?

Pour que les clients Kafka communiquent avec les répartiteurs Kafka, ils doivent être en mesure d’atteindre les répartiteurs sur le réseau. Pour les clusters HDInsight, le réseau virtuel est la limite de sécurité. Par conséquent, le moyen le plus simple de connecter des clients à votre cluster HDInsight consiste à créer des clients sur le même réseau virtuel que le cluster. Parmi les autres scénarios :

* Connexion de clients dans un autre réseau virtuel Azure : Homologuez le réseau virtuel du cluster et le réseau virtuel du client et configurez le cluster pour la [publicité IP](apache-kafka-connect-vpn-gateway.md#configure-kafka-for-ip-advertising). Lorsque vous utilisez la publicité IP, les clients Kafka doivent utiliser des adresses IP de répartiteur pour se connecter aux répartiteurs au lieu de noms de domaine complets (FQDN).

* Connexion de clients locaux : Utilisez un réseau VPN et configurez des serveurs DNS personnalisés, comme décrit dans [Planifier un réseau virtuel pour Azure HDInsight](../hdinsight-plan-virtual-network-deployment.md).

* Création d’un point de terminaison public pour votre service Kafka : Si les exigences de sécurité de votre entreprise l’autorisent, vous pouvez déployer un point de terminaison public pour vos répartiteurs Kafka, ou un point de terminaison REST open source autogéré avec un point de terminaison public.

## <a name="can-i-add-more-disk-space-on-an-existing-cluster"></a>Puis-je ajouter de l’espace disque sur un cluster existant ?

Pour augmenter la quantité d’espace disponible pour les messages Kafka, vous pouvez augmenter le nombre de nœuds. Actuellement, l’ajout de disques à un cluster existant n’est pas pris en charge.

## <a name="can-a-kafka-cluster-work-with-databricks"></a>Un cluster Kafka peut-il fonctionner avec Databricks ? 

Oui, les clusters Kafka peuvent fonctionner avec Databricks tant qu’ils se trouvent dans le même réseau virtuel. Pour utiliser un cluster Kafka avec Databricks, créez un réseau virtuel avec un cluster HDInsight Kafka dans celui-ci, puis spécifiez ce réseau virtuel lorsque vous créez votre espace de travail Databricks et utilisez l’injection de réseau virtuel. Pour plus d’informations, consultez [Déployer Azure Databricks dans votre propre réseau virtuel Azure (injection de réseau virtuel)](/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject). Vous devrez fournir les noms du répartiteur de démarrage du cluster Kafka lors de la création de l’espace de travail Databricks. Pour plus d’informations sur la récupération des noms de service du répartiteur Kafka, consultez [Obtenir les informations sur les hôtes Apache Zookeeper et Répartiteur](./apache-kafka-get-started.md#getkafkainfo).

## <a name="how-can-i-have-maximum-data-durability"></a>Comment puis-je bénéficier d’une durabilité maximale des données ?

La durabilité des données vous permet d’atteindre le niveau de risque de perte de messages le plus faible. Pour atteindre une durabilité maximale des données, nous vous recommandons d’utiliser les paramètres suivants :

* Utilisez un facteur de réplication minimal de 3 dans la plupart des régions
* Utilisez un facteur de réplication minimal de 4 dans les régions avec seulement deux domaines d’erreur
* Désactivez les choix de responsable non propres
* Réglez **min.insync.replicas** sur 2 ou plus : cela modifie le nombre de réplicas qui doivent être complètement synchronisés avec le responsable avant qu’une écriture puisse continuer
* Définissez la propriété **acks** sur **all** : cette propriété requiert que tous les réplicas accusent réception de tous les messages

La configuration de Kafka pour une plus grande cohérence des données affecte la disponibilité des répartiteurs pour produire des requêtes.

## <a name="can-i-replicate-my-data-to-multiple-clusters"></a>Puis-je répliquer mes données dans plusieurs clusters ?

Oui, les données peuvent être répliquées sur plusieurs clusters à l’aide de Kafka MirrorMaker. Pour plus d’informations sur la configuration de MirrorMaker, consultez [Mettre en miroir les rubriques Apache Kafka](apache-kafka-mirroring.md). En outre, il existe d’autres technologies et fournisseurs open source autogérés qui peuvent vous aider à effectuer la réplication vers plusieurs clusters, comme [Brooklin](https://github.com/linkedin/Brooklin/).

## <a name="can-i-upgrade-my-cluster-how-should-i-upgrade-my-cluster"></a>Puis-je mettre à niveau mon cluster ? Comment mettre à niveau mon cluster ?

Nous ne prenons actuellement pas en charge les mises à niveau de version des clusters en place. Pour mettre à jour votre cluster vers une version plus récente de Kafka, créez un nouveau cluster avec la version que vous souhaitez et migrez vos clients Kafka pour utiliser le nouveau cluster.

## <a name="how-do-i-monitor-my-kafka-cluster"></a>Comment surveiller mon cluster Kafka ?

Utilisez Azure Monitor pour analyser vos [journaux Kafka](./apache-kafka-log-analytics-operations-management.md).

## <a name="next-steps"></a>Étapes suivantes

* [Configurer le chiffrement et l’authentification TLS pour Apache Kafka dans Azure HDInsight](./apache-kafka-ssl-encryption-authentication.md)
* [Utiliser MirrorMaker pour répliquer des rubriques Apache Kafka avec Kafka sur HDInsight](./apache-kafka-mirroring.md)