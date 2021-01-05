---
title: Sécuriser et isoler les clusters Azure HDInsight avec Azure Private Link (préversion)
description: Découvrez comment isoler les clusters Azure HDInsight dans un réseau virtuel à l’aide d’Azure Private Link.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 10/15/2020
ms.openlocfilehash: fac26c616c977eedc466f004a9455297ec995fb8
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352539"
---
# <a name="secure-and-isolate-azure-hdinsight-clusters-with-private-link-preview"></a>Sécuriser et isoler les clusters Azure HDInsight avec Azure Private Link (préversion)

Dans l’[architecture de réseau virtuel par défaut](./hdinsight-virtual-network-architecture.md) d’Azure HDInsight, le fournisseur de ressources HDInsight communique avec le cluster à l’aide d’IP publiques. Certains scénarios nécessitent un isolement réseau complet, sans utiliser d’IP publiques. Dans cet article, vous allez découvrir les contrôles avancés que vous pouvez utiliser pour créer un cluster HDInsight privé. Pour savoir comment restreindre le trafic vers et depuis votre cluster sans isolement réseau complet, consultez [Contrôler le trafic dans Azure HDInsight](./control-network-traffic.md).

Vous pouvez créer des clusters HDInsight privés en configurant des propriétés réseau spécifiques dans un modèle Resource Manager. Il existe deux propriétés que vous utilisez pour créer des clusters HDInsight privés :

* Supprimez les IP publiques en définissant `resourceProviderConnection` sur Sortante.
* Activez Azure Private Link et utilisez [Points de terminaison privés](../private-link/private-endpoint-overview.md) en activant `privateLink`.

## <a name="remove-public-ip-addresses"></a>Supprimer des IP publiques

Par défaut, le fournisseur de ressources HDInsight utilise une connexion *entrante* vers le cluster en utilisant des IP publiques. Lorsque la propriété réseau `resourceProviderConnection` est définie sur *sortante*, elle inverse les connexions vers le fournisseur de ressources HDInsight de sorte que les connexions sont toujours initiées de l’intérieur du cluster vers l’extérieur en direction du fournisseur de ressources. Sans connexion entrante, les étiquettes de service entrantes et les IP publiques ne sont pas nécessaires.

Les équilibreurs de charge de base utilisés dans l’architecture de réseau virtuel par défaut fournissent automatiquement une traduction d’adresses réseau (NAT) publique pour accéder aux dépendances sortantes requises, telles que le fournisseur de ressources HDInsight. Si vous souhaitez restreindre la connectivité sortante à l’Internet public, vous pouvez [configurer un pare-feu](./hdinsight-restrict-outbound-traffic.md), mais ce n’est pas obligatoire.

La configuration de `resourceProviderConnection` sur Sortante vous permet également d’accéder à des ressources spécifiques au cluster, telles que des metastores Azure Data Lake Storage Gen2 ou externes, en utilisant des points de terminaison privés. L’utilisation de points de terminaison privés pour ces ressources n’est pas obligatoire, mais si vous prévoyez d’avoir des points de terminaison privés pour ces ressources, vous devez configurer des points de terminaison privés et les entrées DNS `before` de créer le cluster HDInsight. Nous vous recommandons de créer et de fournir toutes les bases de données SQL externes dont vous avez besoin, comme Apache Ranger, Ambari, Oozie et les metastores Hive, au moment de la création du cluster. Toutes ces ressources doivent être accessibles depuis l’intérieur du sous-réseau du cluster, par le biais de leur propre point de terminaison privé ou d’une autre manière.

L’utilisation de points de terminaison privés pour Azure Key Vault n’est pas prise en charge. Si vous utilisez Azure Key Vault pour le chiffrement CMK au repos, le point de terminaison Azure Key Vault doit être accessible à partir du sous-réseau HDInsight sans point de terminaison privé.

Le diagramme suivant montre à quoi peut ressembler une potentielle architecture de réseau virtuel HDInsight lorsque `resourceProviderConnection` est définie sur Sortante :

:::image type="content" source="media/hdinsight-private-link/outbound-resource-provider-connection-only.png" alt-text="Diagramme de l’architecture HDInsight utilisant une connexion de fournisseur de ressources sortante":::

Après avoir créé votre cluster, vous devez configurer une résolution DNS appropriée. L’enregistrement DNS de nom canonique (CNAME) suivant est créé sur la zone DNS publique gérée par Azure : `azurehdinsight.net`.

```dns
<clustername>    CNAME    <clustername>-int
```

Pour accéder au cluster à l’aide de noms de domaine complets de cluster, vous pouvez utiliser directement les adresses IP privées d’équilibreurs de charge internes ou utiliser votre propre zone DNS privée pour remplacer les points de terminaison de cluster en fonction de vos besoins. Par exemple, vous pouvez avoir une zone DNS privée, `azurehdinsight.net`, et ajouter vos adresses IP privées le cas échéant :

```dns
<clustername>        A   10.0.0.1
<clustername-ssh>    A   10.0.0.2
```

## <a name="enable-private-link"></a>Activer Azure Private Link

Azure Private Link, qui est désactivé par défaut, requiert des connaissances approfondies en matière de réseau pour configurer correctement les règles de pare-feu et les itinéraires définis par l’utilisateur (UDR) avant de créer un cluster. L’utilisation de ce paramètre est facultative, mais n’est disponible que lorsque la propriété réseau `resourceProviderConnection` est définie sur *sortante*, comme décrit dans la section précédente.

Lorsque `privateLink` est définie sur *activé*, des [équilibreurs de charge standard](../load-balancer/load-balancer-overview.md) (SLB) internes sont créés et un service Azure Private Link est approvisionné pour chaque SLB. Le service Azure Private Link vous permet d’accéder au cluster HDInsight à partir de points de terminaison privés.

Les équilibreurs de charge standard n’approvisionnent pas automatiquement la [NAT sortante publique](../load-balancer/load-balancer-outbound-connections.md) comme le font les équilibreurs de charge de base. Vous devez fournir votre propre solution NAT, par exemple [NAT de réseau virtuel Azure](../virtual-network/nat-overview.md) ou un [pare-feu](./hdinsight-restrict-outbound-traffic.md), pour les dépendances sortantes. Votre cluster HDInsight doit toujours avoir accès à ses dépendances sortantes. Si ces dépendances sortantes ne sont pas autorisées, la création du cluster peut échouer.

### <a name="prepare-your-environment"></a>Préparation de votre environnement

Pour réussir la création de services de liaison privée, vous devez explicitement [désactiver les stratégies réseau pour le service de liaison privée](../private-link/disable-private-link-service-network-policy.md).

Le diagramme suivant montre un exemple de configuration de mise en réseau requise avant de créer un cluster. Dans cet exemple, tout le trafic sortant est [forcé](../firewall/forced-tunneling.md) vers Pare-feu Azure en utilisant un UDR, et les dépendances sortantes requises doivent être « autorisées » sur le pare-feu avant de créer un cluster. Pour les clusters Pack Sécurité Entreprise, la connectivité réseau à Azure Active Directory Domain Services peut être fournie par l’appairage de réseaux virtuels.

:::image type="content" source="media/hdinsight-private-link/before-cluster-creation.png" alt-text="Diagramme de l’environnement de liaison privée avant la création du cluster":::

Une fois que vous avez configuré la mise en réseau, vous pouvez créer un cluster avec une connexion de fournisseur de ressources sortante et une liaison privée activée, comme illustré dans la figure suivante. Dans cette configuration, il n’y a pas d’IP publiques et Azure Private Link est configuré pour chaque équilibreur de charge standard.

:::image type="content" source="media/hdinsight-private-link/after-cluster-creation.png" alt-text="Diagramme de l’environnement de liaison privée après la création du cluster":::

### <a name="access-a-private-cluster"></a>Accéder à un cluster privé

Pour accéder aux clusters privés, vous pouvez utiliser directement les adresses IP privées des équilibreurs de charge internes, ou vous pouvez utiliser des extensions DNS Azure Private Link et des points de terminaison privés. Lorsque le paramètre `privateLink` est défini sur activé, vous pouvez créer vos propres points de terminaison privés et configurer la résolution DNS par le biais de zones DNS privées.

Les entrées Azure Private Link créées dans la zone DNS publique gérée par Azure `azurehdinsight.net` sont les suivantes :

```dns
<clustername>        CNAME    <clustername>.privatelink
<clustername>-int    CNAME    <clustername>-int.privatelink
<clustername>-ssh    CNAME    <clustername>-ssh.privatelink
```

L’illustration suivante montre un exemple des entrées DNS privées requises pour accéder au cluster à partir d’un réseau virtuel qui n’est pas appairé ou qui ne dispose pas d’une ligne de vue directe sur les équilibreurs de charge du cluster. Vous pouvez utiliser la zone privée Azure pour remplacer les noms de domaine complets `*.privatelink.azurehdinsight.net` et résoudre les adresses IP de vos propres points de terminaison privés.

:::image type="content" source="media/hdinsight-private-link/access-private-clusters.png" alt-text="Diagramme de l’architecture de liaison privée":::

## <a name="how-to-create-clusters"></a>Comment créer des clusters ?
### <a name="use-arm-template-properties"></a>Utiliser les propriétés du modèle Resource Manager

L’extrait de code JSON suivant comprend les deux propriétés réseau que vous devez configurer dans votre modèle Resource Manager pour créer un cluster HDInsight privé.

```json
networkProperties: {
    "resourceProviderConnection": "Inbound" | "Outbound",
    "privateLink": "Enabled" | "Disabled"
}
```

Pour obtenir un modèle complet avec de nombreuses fonctionnalités de sécurité d’entreprise HDInsight, notamment Azure Private Link, consultez [Modèle de sécurité d’entreprise HDInsight](https://github.com/Azure-Samples/hdinsight-enterprise-security/tree/main/ESP-HIB-PL-Template).

### <a name="use-azure-powershell"></a>Utiliser Azure PowerShell

Pour utiliser PowerShell, consultez l'exemple disponible [ici](/powershell/module/az.hdinsight/new-azhdinsightcluster?view=azps-5.1.0#example-4--create-an-azure-hdinsight-cluster-with-relay-outbound-and-private-link-feature).

### <a name="use-azure-cli"></a>Utiliser l’interface de ligne de commande Microsoft Azure
Pour utiliser l'interface de ligne de commande Azure, consultez l'exemple disponible [ici](/cli/azure/hdinsight?view=azure-cli-latest#az_hdinsight_create-examples).

## <a name="next-steps"></a>Étapes suivantes

* [Pack Sécurité Entreprise pour Azure HDInsight](enterprise-security-package.md)
* [Informations et recommandations générales sur la sécurité d’entreprise dans Azure HDInsight](./domain-joined/general-guidelines.md)