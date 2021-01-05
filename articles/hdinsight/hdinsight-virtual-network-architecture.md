---
title: Architecture de réseau virtuel Azure HDInsight
description: Découvrez les ressources disponibles pour créer un cluster HDInsight dans un réseau virtuel Azure.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/14/2020
ms.openlocfilehash: ad0e0250b32f2bdef4944e6e148be3215f3822f7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81390219"
---
# <a name="azure-hdinsight-virtual-network-architecture"></a>Architecture de réseau virtuel Azure HDInsight

Cet article décrit les ressources présentes lorsque vous déployez un cluster HDInsight dans un réseau virtuel Azure personnalisé. Ces informations vous aideront à connecter des ressources locales à votre cluster HDInsight dans Azure. Pour plus d’informations sur les réseaux virtuels Azure, consultez [Présentation du réseau virtuel Azure](../virtual-network/virtual-networks-overview.md).

## <a name="resource-types-in-azure-hdinsight-clusters"></a>Types de ressources des clusters Azure HDInsight

Les clusters Azure HDInsight présentent différents types de machines virtuelles ou nœuds. Chaque type de nœud joue un rôle dans le fonctionnement du système. Le tableau suivant récapitule les types de nœuds et leurs rôles dans le cluster.

| Type | Description |
| --- | --- |
| Nœud principal |  Pour tous les types de cluster, à l’exception d’Apache Storm, les nœuds principaux hébergent les processus en charge de l'exécution de l’application distribuée. Le nœud principal est aussi le nœud sur lequel vous pouvez établir une connexion SSH et exécuter des applications ensuite coordonnées pour s’exécuter sur les ressources du cluster. Le nombre de nœuds principaux est fixé à deux pour tous les types de cluster. |
| Nœud Zookeeper | Zookeeper coordonne les tâches entre les nœuds qui assurent le traitement des données. Il désigne également le nœud principal et suit le nœud principal qui exécute un service maître spécifique. Le nombre de nœuds ZooKeeper est fixé à trois. |
| Nœud Worker | Représente les nœuds qui prennent en charge des fonctionnalités de traitement des données. Les nœuds Worker peuvent être ajoutés ou supprimés du cluster pour mettre à l'échelle les capacités de calcul et gérer les coûts. |
| Nœud de périphérie R Server | Le nœud de périphérie R Server représente le nœud sur lequel vous pouvez établir une connexion SSH et exécuter des applications ensuite coordonnées pour s’exécuter sur les ressources de cluster. Un nœud de périphérie ne participe pas activement à l’analyse de données au sein du cluster. Ce nœud héberge également R Studio Server, ce qui vous permet d’exécuter l’application R à l’aide d’un navigateur. |
| Nœud de région | Pour le type de cluster HBase, le nœud de région (également appelé nœud de données) s’exécute le serveur de région. Les serveurs de région gèrent une partie des données gérées par HBase. Les nœuds région peuvent être ajoutés ou supprimés du cluster pour mettre à l'échelle les capacités de calcul et gérer les coûts.|
| Nœud Nimbus | Pour le type de cluster Storm, le nœud Nimbus offre des fonctionnalités similaires à celles du nœud principal. Le nœud Nimbus attribue des tâches aux autres nœuds d'un cluster via Zookeeper, qui coordonne l’exécution des topologies Storm. |
| Nœud superviseur | Pour le type de cluster Storm, le nœud superviseur exécute les instructions fournies par le nœud Nimbus afin de procéder au traitement. |

## <a name="resource-naming-conventions"></a>Conventions de nommage des ressources

Utilisez des noms de domaine complets (FQDN) lors de l'adressage des nœuds de votre cluster. Vous pouvez obtenir les noms de domaine complets de différents types de nœuds dans votre cluster à l’aide de l’[API Ambari](hdinsight-hadoop-manage-ambari-rest-api.md).

Ces noms de domaine complets sont au format suivant : `<node-type-prefix><instance-number>-<abbreviated-clustername>.<unique-identifier>.cx.internal.cloudapp.net`.

`<node-type-prefix>` est *hn* pour les nœuds principaux, *wn* pour les nœuds Worker et *zn* pour les nœuds ZooKeeper.

Si vous n’avez besoin que du nom d’hôte, utilisez uniquement la première partie du nom de domaine complet : `<node-type-prefix><instance-number>-<abbreviated-clustername>`

## <a name="basic-virtual-network-resources"></a>Ressources de base du réseau virtuel

Le diagramme suivant illustre le placement des nœuds HDInsight et des ressources réseau dans Azure.

![Diagramme des entités HDInsight créées dans un réseau virtuel Azure personnalisé](./media/hdinsight-virtual-network-architecture/hdinsight-vnet-diagram.png)

Les ressources par défaut d'un réseau virtuel Azure incluent les types de nœuds de cluster mentionnés dans le tableau précédent. Et les périphériques réseau qui prennent en charge la communication entre le réseau virtuel et les réseaux extérieurs.

Le tableau suivant récapitule les neufs nœuds de cluster créés lors du déploiement de HDInsight dans un réseau virtuel Azure personnalisé.

| Type de ressource | Nombre présent | Détails |
| --- | --- | --- |
|Nœud principal | two |    |
|Nœud zookeeper | three | |
|Nœud Worker | two | Ce nombre peut varier en fonction de la configuration et de la mise à l'échelle du cluster. Un minimum de trois nœuds Worker est nécessaire pour Apache Kafka.  |
|Nœud de passerelle | two | Les nœuds de passerelle sont des machines virtuelles créées dans Azure, mais ne sont pas visibles dans votre abonnement. Contactez le support technique si vous devez redémarrer ces nœuds. |

Les ressources réseau suivantes sont automatiquement créées à l’intérieur du réseau virtuel utilisé avec HDInsight :

| Ressource de mise en réseau | Nombre présent | Détails |
| --- | --- | --- |
|Équilibrage de charge | three | |
|Interfaces réseau | neuf | Cette valeur est basée sur un cluster normal, où chaque nœud dispose de sa propre interface réseau. Les neuf interfaces sont destinées aux deux nœuds principaux, aux trois nœuds Zookeeper, aux deux nœuds Worker et aux deux nœuds de passerelle mentionnés dans le tableau précédent. |
|Adresses IP publiques | two |    |

## <a name="endpoints-for-connecting-to-hdinsight"></a>Points de terminaison pour la connexion à HDInsight

Vous pouvez accéder à votre cluster HDInsight de trois façons :

- Point de terminaison HTTPS à l'extérieur du réseau virtuel au niveau de `CLUSTERNAME.azurehdinsight.net`.
- Point de terminaison SSH pour se connecter directement au nœud principal à `CLUSTERNAME-ssh.azurehdinsight.net`.
- Point de terminaison HTTPS au sein du réseau virtuel `CLUSTERNAME-int.azurehdinsight.net`. Notez la présence de le « `-int` » dans cette URL. Ce point de terminaison est résolu en adresse IP privée dans ce réseau virtuel et n'est pas accessible à partir du réseau Internet public.

Ces trois points de terminaison se voient attribuer chacun un équilibreur de charge.

Des adresses IP publiques sont également fournies aux deux points de terminaison permettant la connexion depuis l'extérieur du réseau virtuel.

1. Une adresse IP publique est attribuée à l'équilibreur de charge pour le nom de domaine complet (FQDN) à utiliser pour vous connecter au cluster depuis Internet `CLUSTERNAME.azurehdinsight.net`.
1. La deuxième adresse IP publique est utilisée pour le nom de domaine SSH `CLUSTERNAME-ssh.azurehdinsight.net`.

## <a name="next-steps"></a>Étapes suivantes

- [Sécuriser le trafic entrant vers les clusters HDInsight dans un réseau virtuel avec point de terminaison privé](https://azure.microsoft.com/blog/secure-incoming-traffic-to-hdinsight-clusters-in-a-vnet-with-private-endpoint/)
