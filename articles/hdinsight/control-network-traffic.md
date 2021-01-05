---
title: Contrôler le trafic réseau dans Azure HDInsight
description: Découvrez les techniques permettant de contrôler le trafic entrant et sortant vers des clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: 348b9b80c74f085ce31ecce93753a253782fe3ea
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92543166"
---
# <a name="control-network-traffic-in-azure-hdinsight"></a>Contrôler le trafic réseau dans Azure HDInsight

Le trafic réseau dans les réseaux virtuels Azure peut être contrôlé à l’aide des méthodes suivantes :

* Les **Groupes de sécurité réseau** (NSG) vous permettent de filtrer le trafic entrant et sortant changé avec le réseau. Pour plus d’informations, voir le document [Filtrer le trafic réseau avec les groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).

* Les **appliances virtuelles réseau** (NVA) peuvent uniquement être utilisées avec le trafic sortant. Les appliances virtuelles réseau répliquent les fonctionnalités d’appareils tels que des routeurs et des pare-feux. Pour plus d’informations, voir le document [Appliances réseau](https://azure.microsoft.com/solutions/network-appliances).

En tant que service managé, HDInsight doit avoir un accès illimité aux services d’intégrité et de gestion de HDInsight pour le trafic entrant et sortant du réseau virtuel. Lorsque vous utilisez des groupes de sécurité réseau, vous devez vous assurer que ces services peuvent toujours communiquer avec le cluster HDInsight.

![Diagramme des entités HDInsight créées dans un réseau virtuel Azure personnalisé](./media/control-network-traffic/hdinsight-vnet-diagram.png)

## <a name="hdinsight-with-network-security-groups"></a>Utiliser HDInsight avec des groupes de sécurité réseau

Si vous prévoyez d’utiliser des **groupes de sécurité réseau** pour contrôler le trafic réseau, effectuez les actions suivantes avant d’installer HDInsight :

1. Identifiez la région Azure que vous projetez d’utiliser pour HDInsight.

2. Identifiez les étiquettes de service demandées par HDInsight pour votre région. Il existe plusieurs façons d’obtenir ces étiquettes de service :
    1. Consultez la liste des étiquettes de service publiées dans [Étiquettes de service de groupe de sécurité réseau (NSG) pour Azure HDInsight](hdinsight-service-tags.md). 
    2. Si votre région ne figure pas dans la liste, utilisez l’[API Service Tag Discovery](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview) pour rechercher une étiquette de service pour votre région.
    3. Si vous ne parvenez pas à utiliser l’API, téléchargez le [fichier JSON d’étiquette de service](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) et recherchez la région de votre choix.


3. Créez ou modifiez les groupes de sécurité réseau pour le sous-réseau dans lequel vous prévoyez d’installer HDInsight.

    * __Groupes de sécurité réseau__  : autorisez le trafic __entrant__ sur le port __443__ à partir des adresses IP. Cela garantit que les services de gestion HDInsight peuvent atteindre le cluster depuis un point en dehors du réseau virtuel. Pour les clusters compatibles avec le __proxy REST Kafka__ , autorisez également le trafic __entrant__ sur le port __9400__ . Cela garantit que le serveur proxy REST Kafka est accessible.

Pour plus d’informations sur les groupes de sécurité réseau, consultez [Présentation des groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).

## <a name="controlling-outbound-traffic-from-hdinsight-clusters"></a>Contrôle du trafic sortant à partir de clusters HDInsight

Pour plus d’informations sur le contrôle du trafic sortant à partir de clusters HDInsight, consultez [Configurer la restriction du trafic réseau sortant pour les clusters Azure HDInsight](hdinsight-restrict-outbound-traffic.md).

### <a name="forced-tunneling-to-on-premises"></a>Tunneling forcé en local

Le tunneling forcé est une configuration de routage défini par l’utilisateur où tout le trafic en provenance d’un sous-réseau est acheminé de force vers un réseau ou un emplacement spécifique, tel que votre réseau ou Pare-feu local. Le tunneling forcé de tous les transferts de données vers un emplacement local n’est _pas_ recommandé en raison des volumes importants de transfert de données et de l’impact potentiel sur les performances.

Les clients qui souhaitent configurer le tunneling forcé doivent utiliser des [metastores personnalisés](./hdinsight-use-external-metadata-stores.md) et configurer la connectivité appropriée entre le sous-réseau du cluster ou le réseau local et ces metastores personnalisés.

Pour voir un exemple de configuration du routage défini par l’utilisateur avec le Pare-feu Azure, consultez [Configurer la restriction du trafic réseau sortant pour les clusters Azure HDInsight](hdinsight-restrict-outbound-traffic.md).

## <a name="required-ports"></a>Ports requis

Si vous prévoyez d’utiliser un **pare-feu** et d’accéder au cluster depuis l’extérieur sur certains ports, vous devez autoriser le trafic sur ces ports nécessaires à votre scénario. Par défaut, aucun filtrage spécial des ports n’est nécessaire tant que le trafic de gestion Azure expliqué dans la section précédente est autorisé à atteindre le cluster sur le port 443.

Pour obtenir la liste des ports utilisés pour des services spécifiques, voir le document [Ports utilisés par les services Apache Hadoop sur HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Pour plus d’informations sur les règles de pare-feu pour les appliances virtuelles, voir le document [Scénario d’appliance virtuelle](../virtual-network/virtual-network-scenario-udr-gw-nva.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir des exemples de code et des exemples de création de réseaux virtuels Azure, consultez [Créer des réseaux virtuels pour les clusters Azure HDInsight](hdinsight-create-virtual-network.md).
* Pour un exemple de bout en bout de configuration de HDInsight pour se connecter à un réseau local, voir [Connecter HDInsight à un réseau local](./connect-on-premises-network.md).
* Pour plus d’informations sur les réseaux virtuels Azure, voir [Vue d'ensemble de Réseau virtuel Azure](../virtual-network/virtual-networks-overview.md).
* Pour plus d’informations sur les groupes de sécurité réseau, consultez [Groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).
* Pour plus d’informations sur les routages par l’utilisateur, consultez [Routage définis par l’utilisateur et transfert IP](../virtual-network/virtual-networks-udr-overview.md).
* Pour plus d’informations sur les réseaux virtuels, consultez [Planifier des réseaux virtuels pour HDInsight](./hdinsight-plan-virtual-network-deployment.md).