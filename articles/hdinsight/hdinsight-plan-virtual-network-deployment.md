---
title: Planifier un réseau virtuel pour Azure HDInsight
description: Apprenez à planifier un déploiement Azure Virtual Network pour connecter HDInsight à d'autres ressources de cloud ou à des ressources de votre centre de données.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,seoapr2020
ms.date: 05/04/2020
ms.openlocfilehash: 2d6d18cc73c02d3139074555fdd22a40c797ce40
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825145"
---
# <a name="plan-a-virtual-network-for-azure-hdinsight"></a>Planifier un réseau virtuel pour Azure HDInsight

Cet article fournit des informations générales sur l’utilisation de [Réseaux virtuels Azure](../virtual-network/virtual-networks-overview.md) avec Azure HDInsight. Il aborde également les décisions de conception et d’implémentation qui doivent être prises avant de pouvoir implémenter un réseau virtuel pour votre cluster HDInsight. Une fois la phase de planification terminée, vous pouvez procéder à la [création de réseaux virtuels pour les clusters Azure HDInsight](hdinsight-create-virtual-network.md). Pour plus d’informations sur les adresses IP de gestion HDInsight nécessaires pour configurer correctement des groupes de sécurité réseau (NSG) et des itinéraires définis par l’utilisateur, voir [Adresses IP de gestion HDInsight](hdinsight-management-ip-addresses.md).

L’utilisation d’un réseau virtuel Azure permet les scénarios suivants :

* connexion à HDInsight directement à partir d’un réseau local ;
* connexion de HDInsight à des banques de données dans un réseau virtuel Azure ;
* accès direct aux services Apache Hadoop qui ne sont pas disponibles publiquement sur Internet ; par exemple, les API Apache Kafka ou l’API Java Apache HBase.

> [!IMPORTANT]
> La création d’un cluster HDInsight dans un réseau virtuel permet de créer plusieurs ressources réseau, telles que des ressources réseau et des équilibreurs de charge. Ne **supprimez pas** ces ressources réseau, car elles sont nécessaires au bon fonctionnement de votre cluster avec le réseau virtuel (VNET).
>
> Après le 28 février 2019, les ressources réseau (par exemple, les cartes réseau, les équilibreurs de charge, etc.) des NOUVEAUX clusters HDInsight créés dans un réseau virtuel seront approvisionnées dans le même groupe de ressources de cluster HDInsight. Auparavant, ces ressources étaient approvisionnées dans le groupe de ressources du réseau virtuel. Les clusters en cours d’exécution et les clusters créés sans réseau virtuel ne seront pas modifiés.

## <a name="planning"></a>Planification

Les questions auxquelles vous devez répondre lors de la planification de l’installation de HDInsight dans un réseau virtuel sont les suivantes :

* Devez installer HDInsight dans un réseau virtuel existant ? Ou bien créez-vous un réseau ?

    Si vous utilisez un réseau virtuel existant, il se peut que vous deviez modifier la configuration de celui-ci avant d’installer HDInsight. Pour plus d’informations, voir la section [Ajouter HDInsight à un réseau virtuel existant](#existingvnet).

* Vous souhaitez connecter le réseau virtuel contenant HDInsight à un autre réseau virtuel ou à votre réseau local ?

    Pour utiliser aisément des ressources de différents réseaux, il se peut que vous deviez créer un DNS personnalisé et configurer un transfert de DNS. Pour plus d’informations, voir la section [Connecter plusieurs réseaux](#multinet).

* Souhaitez-vous restreindre/rediriger le trafic entrant ou sortant échangé avec HDInsight ?

    HDInsight doit disposer d’une communication illimitée avec les adresses IP spécifiques dans le centre de données Azure. Il existe également plusieurs ports qui doivent être autorisés au travers de pare-feu pour la communication du client. Pour plus d’informations, consultez [Contrôler le trafic réseau](./control-network-traffic.md).

## <a name="add-hdinsight-to-an-existing-virtual-network"></a><a id="existingvnet"></a>Ajouter HDInsight à un réseau virtuel existant

Suivez les étapes de cette section pour découvrir comment ajouter un nouveau cluster HDInsight à un réseau virtuel Azure existant.

> [!NOTE]  
> Vous ne pouvez pas ajouter un cluster HDInsight existant à un réseau virtuel.

1. Utilisez-vous un modèle de déploiement classique ou Resource Manager pour le réseau virtuel ?

    HDInsight 3.4 et versions ultérieures nécessitent un réseau virtuel Resource Manager. Les versions antérieures de HDInsight nécessitaient un réseau virtuel classique.

    Si votre réseau existant est un réseau virtuel classique, vous devez créer un réseau virtuel Resource Manager, puis connecter les deux. [Connexion de réseaux virtuels classiques aux nouveaux réseaux virtuels](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md).

    Une fois joint, HDInsight installé dans le réseau peut interagir avec des ressources du réseau classique.

2. Utilisez-vous des groupes de sécurité réseau, des itinéraires définis par l'utilisateur ou des appliances de réseau virtuel pour restreindre le trafic échangé avec le réseau virtuel ?

    Service administré, HDInsight requiert un accès illimité à plusieurs adresses IP dans le centre de données Azure. Pour permettre la communication avec ces adresses IP, mettez à jour des groupes de sécurité réseau ou des itinéraires définis par l'utilisateur.

    HDInsight héberge plusieurs services qui une série de ports. Ne bloquez pas le trafic vers ces ports. Pour obtenir la liste des ports auxquels autoriser l’accès via des pare-feu d’appliance virtuelle, voir la section Sécurité.

    Pour rechercher votre configuration de sécurité existante, utilisez les commandes Azure PowerShell ou Azure CLI suivantes :

    * Groupes de sécurité réseau

        Remplacez `RESOURCEGROUP` par le nom du groupe de ressources qui contient le réseau virtuel, puis entrez la commande :

        ```powershell
        Get-AzNetworkSecurityGroup -ResourceGroupName  "RESOURCEGROUP"
        ```

        ```azurecli
        az network nsg list --resource-group RESOURCEGROUP
        ```

        Pour plus d’informations, voir le document [Résoudre les problèmes relatifs aux groupes de sécurité réseau](../virtual-network/diagnose-network-traffic-filter-problem.md).

        > [!IMPORTANT]  
        > Les règles de groupe de sécurité réseau sont appliquées dans un ordre basé sur leur priorité. La première règle correspondant au modèle de trafic est appliquée, et aucune autre n’est appliquée à ce trafic. Règles d’ordre de la plus permissive à la moins permissive. Pour plus d’informations, voir le document [Filtrer le trafic réseau avec les groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).

    * Itinéraires définis par l’utilisateur

        Remplacez `RESOURCEGROUP` par le nom du groupe de ressources qui contient le réseau virtuel, puis entrez la commande :

        ```powershell
        Get-AzRouteTable -ResourceGroupName "RESOURCEGROUP"
        ```

        ```azurecli
        az network route-table list --resource-group RESOURCEGROUP
        ```

        Pour plus d’informations, voir le document [Résoudre les problèmes relatifs aux itinéraires](../virtual-network/diagnose-network-routing-problem.md).

3. Créez un cluster HDInsight et sélectionnez le réseau virtuel Azure pendant la configuration. Pour comprendre le processus de création du cluster, utilisez les étapes indiquées dans les documents suivants :

    * [Créer un cluster HDInsight à l’aide du portail Azure](hdinsight-hadoop-create-linux-clusters-portal.md)
    * [Créer un cluster HDInsight à l’aide d’Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
    * [Créer un cluster HDInsight à l’aide d’Azure Classic CLI](hdinsight-hadoop-create-linux-clusters-azure-cli.md)
    * [Créer un cluster HDInsight à l’aide d’un modèle Azure Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md)

   > [!IMPORTANT]  
   > L’ajout de HDInsight à un réseau virtuel est une étape de configuration facultative. Veillez à sélectionner le réseau virtuel lors de la configuration du cluster.

## <a name="connecting-multiple-networks"></a><a id="multinet"></a>Connexion de plusieurs réseaux

Le principal défi avec une configuration de réseau multiples est la résolution de noms entre les réseaux.

Azure assure la résolution de noms pour les services Azure installés dans un réseau virtuel. Cette résolution de noms intégrée permet à HDInsight de se connecter aux ressources suivantes à l’aide d’un nom de domaine complet (FQDN) :

* Toute ressource disponible sur Internet. Par exemple, microsoft.com ou windowsupdate.com.

* Toute ressource figurant dans le même réseau virtuel Azure, en utilisant le __nom DNS interne__ de la ressource. Par exemple, lors de l’utilisation de la résolution de noms par défaut, voici des exemples de noms DNS internes attribués aux nœuds Worker HDInsight :

  * wn0-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net
  * wn2-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net

    Les deux nœuds peuvent communiquer directement entre eux et avec d’autres nœuds dans HDInsight en utilisant des noms DNS internes.

La résolution de noms par défaut ne permet __pas__ à HDInsight de résoudre les noms des ressources en réseaux joints au réseau virtuel. Par exemple, il est courant de joindre un réseau local au réseau virtuel. Avec uniquement la résolution de noms par défaut, HDInsight ne peut pas accéder aux ressources du réseau local par leur nom. L’inverse est également vrai, car les ressources de votre réseau local ne peuvent pas accéder aux ressources du réseau virtuel par leur nom.

> [!WARNING]  
> Vous devez créer le serveur DNS personnalisé et configurer le réseau virtuel pour l’utiliser avant de créer le cluster HDInsight.

Pour permettre la résolution de noms entre le réseau virtuel et les ressources dans des réseaux joints, vous devez effectuer les actions suivantes :

1. Créez un serveur DNS personnalisé dans le réseau virtuel Azure où vous prévoyez d’installer HDInsight.

2. Configurez le réseau virtuel pour utiliser le serveur DNS personnalisé.

3. Recherchez le suffixe DNS qu'Azure à affecté à votre réseau virtuel. Cette valeur est similaire à `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net`. Pour plus d’informations sur la recherche de suffixe DNS, consultez la section [Exemple : DNS personnalisé](hdinsight-create-virtual-network.md#example-dns).

4. Configurez le transfert entre les serveurs DNS. La configuration dépend du type de réseau distant.

   * Si le réseau distant est un réseau local, configurez le DNS comme suit :

     * __DNS personnalisé__ (dans le réseau virtuel) :

         * Transférez les demandes relatives au suffixe DNS du réseau virtuel au programme de résolution récursive d’Azure (168.63.129.16). Azure gère les demandes de ressources dans le réseau virtuel.

         * Transférez toutes les autres demandes au serveur DNS local. Le serveur DNS local gère toutes les autres demandes de résolution de noms, y compris les demandes de ressources Internet telles que Microsoft.com.

     * __DNS local__ : transférer les requêtes de suffixe DNS de réseau virtuel vers le serveur DNS personnalisé. Le serveur DNS personnalisé transfère alors les demandes au programme de résolution récursive d’Azure.

       Cette configuration a pour effet de router les demandes de noms de domaine complets (FQDN) qui contiennent le suffixe DNS du réseau virtuel vers le serveur DNS personnalisé. Toutes les autres demandes (même d’adresses Internet publiques) sont gérées par le serveur DNS local.

   * Si le réseau distant est un autre réseau virtuel Azure, configurez DNS comme suit :

     * __DNS personnalisé__ (dans chaque réseau virtuel) :

         * Les demandes de suffixe DNS des réseaux virtuels sont transférées aux serveurs DNS personnalisés. Le DNS de chaque réseau virtuel est chargé de résoudre les ressources au sein de son réseau.

         * Transférez toutes les autres demandes au programme de résolution récursive d’Azure. Le programme de résolution récursive est responsable de la résolution des ressources locales et Internet.

       Le serveur DNS de chaque réseau transfère les demandes à l’autre, en fonction du suffixe DNS. Les autres requêtes sont résolues à l’aide du programme de résolution récursive d’Azure.

     Pour obtenir un exemple de chaque configuration, consultez la section [Exemple : DNS personnalisé](hdinsight-create-virtual-network.md#example-dns).

Pour plus d’informations, voir le document [Résolution de noms pour les machines virtuelles et les instances de rôle](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="directly-connect-to-apache-hadoop-services"></a>Se connecter directement aux services Apache Hadoop

Vous pouvez vous connecter au cluster à l’adresse `https://CLUSTERNAME.azurehdinsight.net`. Il s’agit d’une adresse IP publique, qui ne sera peut-être pas accessible si vous avez utilisé des groupes de sécurité réseau pour limiter le trafic entrant d’Internet. En outre, lorsque vous déployez le cluster dans un réseau virtuel, vous pouvez y accéder à partir du point de terminaison privé `https://CLUSTERNAME-int.azurehdinsight.net`. Ce point de terminaison correspond à une adresse IP privée au sein du réseau virtuel pour l’accès au cluster.

Pour vous connecter à Apache Ambari et à d’autres pages web via le réseau virtuel, procédez comme suit :

1. Pour découvrir les noms de domaine complets (FQDN) internes des nœuds de cluster HDInsight, utilisez l’une des méthodes suivantes :

    Remplacez `RESOURCEGROUP` par le nom du groupe de ressources qui contient le réseau virtuel, puis entrez la commande :

    ```powershell
    $clusterNICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP" | where-object {$_.Name -like "*node*"}

    $nodes = @()
    foreach($nic in $clusterNICs) {
        $node = new-object System.Object
        $node | add-member -MemberType NoteProperty -name "Type" -value $nic.Name.Split('-')[1]
        $node | add-member -MemberType NoteProperty -name "InternalIP" -value $nic.IpConfigurations.PrivateIpAddress
        $node | add-member -MemberType NoteProperty -name "InternalFQDN" -value $nic.DnsSettings.InternalFqdn
        $nodes += $node
    }
    $nodes | sort-object Type
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

    Dans la liste des nœuds retournés, recherchez le nom de domaine complet des nœuds principaux, puis utilisez-les pour vous connecter à Ambari et à d’autres services web. Par exemple, utilisez `http://<headnode-fqdn>:8080` pour accéder à Ambari.

    > [!IMPORTANT]  
    > Certains services hébergés sur les nœuds principaux ne sont actifs que sur un seul nœud à la fois. Si vous tentez d’accéder à un service sur un nœud principal et que l’opération retourne l’erreur 404, basculez vers l’autre nœud principal.

2. Pour déterminer le nœud et le port sur lesquels un service est disponible, voir [Ports utilisés par les services Hadoop dans HDInsight](./hdinsight-hadoop-port-settings-for-services.md).

## <a name="load-balancing"></a>Équilibrage de la charge

Lorsque vous créez un cluster HDInsight, un équilibreur de charge est également créé. Le type de cet équilibreur de charge se trouve au [niveau de la référence SKU de base](../load-balancer/skus.md) qui a certaines contraintes. L’une de ces contraintes est que si vous avez deux réseaux virtuels dans des régions différentes, vous ne pouvez pas vous connecter aux équilibreurs de charge de base. Pour plus d’informations, consultez [FAQ sur les réseaux virtuels : contraintes sur le peering de réseaux virtuels globaux](../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers).

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir des exemples de code et des exemples de création de réseaux virtuels Azure, consultez [Créer des réseaux virtuels pour les clusters Azure HDInsight](hdinsight-create-virtual-network.md).
* Pour un exemple de bout en bout de configuration de HDInsight pour se connecter à un réseau local, voir [Connecter HDInsight à un réseau local](./connect-on-premises-network.md).
* Pour plus d’informations sur les réseaux virtuels Azure, voir [Vue d'ensemble de Réseau virtuel Azure](../virtual-network/virtual-networks-overview.md).
* Pour plus d’informations sur les groupes de sécurité réseau, consultez [Groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).
* Pour plus d’informations sur les routages par l’utilisateur, consultez [Routage définis par l’utilisateur et transfert IP](../virtual-network/virtual-networks-udr-overview.md).
* Pour plus d’informations sur le contrôle du trafic, y compris l’intégration du pare-feu, consultez [Contrôler le trafic réseau](./control-network-traffic.md).
