---
title: Connecter Azure HDInsight à votre réseau local
description: Découvrez comment créer un cluster HDInsight dans un réseau virtuel Azure, puis le connecter à votre réseau local. Découvrez comment configurer une résolution de noms entre HDInsight et votre réseau local à l’aide d’un serveur DNS personnalisé.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 03/04/2020
ms.openlocfilehash: 71ef902e909e552ade5174196f291630bc242ca0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005365"
---
# <a name="connect-hdinsight-to-your-on-premises-network"></a>Connecter HDInsight à votre réseau local

Découvrez comment connecter HDInsight à votre réseau local à l’aide de réseaux virtuels Azure et d’une passerelle VPN. Ce document fournit des informations de planification concernant :

* l’utilisation de HDInsight dans un réseau virtuel Azure qui se connecte à votre réseau local ;
* la configuration de la résolution de noms DNS entre le réseau virtuel et votre réseau local ;
* la configuration de groupes de sécurité réseau pour restreindre l’accès Internet à HDInsight ;
* les ports fournis par HDInsight sur le réseau virtuel.

## <a name="overview"></a>Vue d’ensemble

Pour permettre à HDInsight et aux ressources du réseau joint de communiquer par nom, vous devez effectuer les actions suivantes :

1. Créez un réseau virtuel Azure.
1. créer un serveur DNS personnalisé dans le réseau virtuel Azure ;
1. configurer le réseau virtuel pour utiliser le serveur DNS personnalisé au lieu du programme de résolution récursive Azure par défaut ;
1. configurer le transfert entre le serveur DNS personnalisé et votre serveur DNS local.

Ces configurations déclenchent le comportement suivant :

* Les demandes de noms de domaine complets (FQDN) qui ont le suffixe DNS __pour le réseau virtuel__ sont transférées au serveur DNS personnalisé. Le serveur DNS personnalisé transfère ensuite ces demandes au programme de résolution récursive Azure qui retourne l’adresse IP.
* Toutes les autres demandes sont transférées au serveur DNS local. Même les demandes de ressources Internet publiques telles que microsoft.com sont transférées au serveur DNS local pour la résolution de noms.

Dans le schéma suivant, les lignes vertes correspondent à des demandes de ressources qui se terminent par le suffixe DNS du réseau virtuel. Les lignes bleues correspondent à des demandes de ressources dans le réseau local ou sur l’Internet public.

![Schéma illustrant la façon dont les demandes de DNS sont résolues dans la configuration](./media/connect-on-premises-network/on-premises-to-cloud-dns.png)

## <a name="prerequisites"></a>Prérequis

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](./hdinsight-hadoop-linux-use-ssh-unix.md).
* Si vous utilisez PowerShell, vous aurez besoin du [module AZ](/powershell/azure/).
* Si vous voulez utiliser Azure CLI et que vous ne l’avez pas encore installé, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="create-virtual-network-configuration"></a>Créer la configuration du réseau virtuel

Pour découvrir comment créer un réseau virtuel Azure connecté à votre réseau local, consultez les documents suivants :

* [En passant par le portail Azure](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)
* [Utilisation de Microsoft Azure PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)
* [Utilisation de l’interface de ligne de commande Azure](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)

## <a name="create-custom-dns-server"></a>Créer un serveur DNS personnalisé

> [!IMPORTANT]  
> Vous devez créer et configurer le serveur DNS avant d’installer HDInsight dans le réseau virtuel.

Ces étapes utilisent le [portail Azure](https://portal.azure.com) pour créer une machine virtuelle Azure. Pour d’autres manières de créer une machine virtuelle, consultez [Créer une machine virtuelle avec Azure CLI](../virtual-machines/linux/quick-create-cli.md) et [Créer une machine virtuelle avec Azure PowerShell](../virtual-machines/linux/quick-create-powershell.md).  Pour créer une machine virtuelle Linux utilisant le logiciel DNS [Bind](https://www.isc.org/downloads/bind/), procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
  
1. Dans le menu du haut, sélectionnez **+ Créer une ressource**.

    ![Créer une machine virtuelle Ubuntu](./media/connect-on-premises-network/azure-portal-create-resource.png)

1. Sélectionnez **Calcul** > **Machine virtuelle** pour accéder à la page **Créer une machine virtuelle**.

1. Sous l’onglet __De base__, entrez les informations suivantes :  
  
    | Champ | Valeur |
    | --- | --- |
    |Abonnement |sélectionnez votre abonnement approprié.|
    |Resource group |Sélectionnez le groupe de ressources qui contient le réseau virtuel créé précédemment.|
    |Nom de la machine virtuelle | Entrez un nom convivial qui identifie cette machine virtuelle. Cet exemple utilise **DNSProxy**.|
    |Région | Sélectionnez la même région que celle du réseau virtuel créé précédemment.  Certaines tailles de machine virtuelle ne sont pas disponibles dans toutes les régions.  |
    |Options de disponibilité |  Sélectionnez votre niveau de disponibilité souhaité.  Azure propose une gamme d’options pour gérer la disponibilité et la résilience de vos applications.  Concevez l’architecture de votre solution pour utiliser des machines virtuelles répliquées dans des Zones de disponibilité ou des Groupes à haute disponibilité afin de protéger vos applications et données en cas de panne de centre de données ou d’événements de maintenance. Cet exemple utilise **Aucune redondance d’infrastructure nécessaire**. |
    |Image | Laissez **Ubuntu Server 18.04 LTS**. |
    |Type d'authentification | __Mot de passe__ ou __Clé publique SSH__ : méthode d’authentification pour le compte SSH. Nous recommandons d’utiliser des clés publiques, car elles sont plus sécurisées. Cet exemple utilise **Mot de passe**.  Pour plus d’informations, consultez [Créer et utiliser des clés SSH pour des machines virtuelles Linux](../virtual-machines/linux/mac-create-ssh-keys.md).|
    |Nom d'utilisateur |Entrez le nom d’administrateur pour la machine virtuelle.  Cet exemple utilise **sshuser**.|
    |Mot de passe ou clé publique SSH | Le champ disponible est déterminé par votre choix de **Type d’authentification**.  Entrez la valeur appropriée.|
    |Aucun port d’entrée public|Sélectionnez **Autoriser les ports sélectionnés**. Ensuite, sélectionnez **SSH (22)** dans la liste déroulante **Sélectionner des ports d’entrée**.|

    ![Configuration de base de machine virtuelle](./media/connect-on-premises-network/virtual-machine-basics.png)

    Conservez les valeurs par défaut des autres entrées, puis sélectionnez l’onglet **Mise en réseau**.

4. Sous l’onglet **Mise en réseau**, entrez les informations suivantes :

    | Champ | Valeur |
    | --- | --- |
    |Réseau virtuel | Sélectionnez le réseau virtuel que vous avez créé précédemment.|
    |Subnet | Sélectionnez le sous-réseau par défaut pour le réseau virtuel que vous avez créé précédemment. Ne sélectionnez __pas__ le sous-réseau utilisé par la passerelle VPN.|
    |Adresse IP publique | Utilisez la valeur renseignée automatiquement.  |

    ![HDInsight - Paramètres de réseau virtuel](./media/connect-on-premises-network/virtual-network-settings.png)

    Conservez les valeurs par défaut des autres entrées, puis sélectionnez l’onglet **Vérifier + créer**.

5. Sous l’onglet **Vérifier + créer**, sélectionnez **Créer** pour créer la machine virtuelle.

### <a name="review-ip-addresses"></a>Passer en revue les adresses IP

Une fois la machine virtuelle créée, vous recevrez une notification **Déploiement réussi** avec un bouton **Accéder à la ressource**.  Sélectionnez **Accéder à la ressource** pour accéder à votre nouvelle machine virtuelle.  À partir de la vue par défaut de votre nouvelle machine virtuelle, effectuez les étapes suivantes pour identifier les adresses IP associées :

1. Sous **Paramètres**, sélectionnez **Propriétés**.

2. Notez les valeurs d’**Adresse IP publique/étiquette du nom DNS**  et d’**Adresse IP privée**. Vous en aurez besoin plus tard.

   ![Adresses IP publique et privée](./media/connect-on-premises-network/virtual-machine-ip-addresses.png)

### <a name="install-and-configure-bind-dns-software"></a>Installer et configurer Bind (logiciel DNS)

1. Utilisez SSH pour vous connecter à l’__adresse IP publique__ de la machine virtuelle. Remplacez `sshuser` par le compte d’utilisateur SSH que vous avez spécifié lors de la création de la machine virtuelle. L’exemple suivant établit une connexion à une machine virtuelle dont l’adresse IP est 40.68.254.142 :

    ```bash
    ssh sshuser@40.68.254.142
    ```

2. Pour installer Bind, utilisez les commandes suivantes à partir de la session SSH :

    ```bash
    sudo apt-get update -y
    sudo apt-get install bind9 -y
    ```

3. Pour configurer Bind afin de transférer les demandes de résolution de noms à votre serveur DNS local, utilisez le texte suivant comme contenu du fichier `/etc/bind/named.conf.options` :

    ```DNS Zone file
    acl goodclients {
        10.0.0.0/16; # Replace with the IP address range of the virtual network
        10.1.0.0/16; # Replace with the IP address range of the on-premises network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            forwarders {
            192.168.0.1; # Replace with the IP address of the on-premises DNS server
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```

    > [!IMPORTANT]  
    > Remplacez les valeurs de la section `goodclients` par la plage d’adresses IP du réseau virtuel et du réseau local. Cette section définit les adresses d’origine des demandes que ce serveur DNS accepte.
    >
    > Remplacez l’entrée `192.168.0.1` dans la section `forwarders` par l’adresse IP de votre serveur DNS local. Cette entrée route les demandes DNS vers votre serveur DNS local pour la résolution.

    Pour modifier ce fichier, utilisez la commande suivante :

    ```bash
    sudo nano /etc/bind/named.conf.options
    ```

    Pour enregistrer le fichier, appuyez sur __Ctrl+X__, __O__, puis __Entrée__.

4. À partir de la session SSH, utilisez la commande suivante :

    ```bash
    hostname -f
    ```

    Cette commande retourne une valeur semblable au texte suivant :

    ```output
    dnsproxy.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net
    ```

    Le texte `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` est le __suffixe DNS__ de ce réseau virtuel. Enregistrez cette valeur, car vous en aurez besoin plus tard.

5. Pour configurer Bind afin de résoudre les noms DNS pour les ressources au sein du réseau virtuel, utilisez le texte suivant en tant que contenu du fichier `/etc/bind/named.conf.local` :

    ```DNS Zone file
    // Replace the following with the DNS suffix for your virtual network
    zone "icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net" {
        type forward;
        forwarders {168.63.129.16;}; # The Azure recursive resolver
    };
    ```

    > [!IMPORTANT]  
    > Vous devez remplacer `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` par le suffixe DNS que vous avez récupéré précédemment.

    Pour modifier ce fichier, utilisez la commande suivante :

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    Pour enregistrer le fichier, appuyez sur __Ctrl+X__, __O__, puis __Entrée__.

6. Pour démarrer Bind, utilisez la commande suivante :

    ```bash
    sudo service bind9 restart
    ```

7. Pour vérifier que Bind peut résoudre les noms de ressources de votre réseau local, utilisez les commandes suivantes :

    ```bash
    sudo apt install dnsutils
    nslookup dns.mynetwork.net 10.0.0.4
    ```

    > [!IMPORTANT]  
    > Remplacez `dns.mynetwork.net` par le nom de domaine complet (FQDN) d’une ressource de votre réseau local.
    >
    > Remplacez `10.0.0.4` par __l’adresse IP interne__ de votre serveur DNS personnalisé dans le réseau virtuel.

    La réponse est similaire au texte suivant :

    ```output
    Server:         10.0.0.4
    Address:        10.0.0.4#53

    Non-authoritative answer:
    Name:   dns.mynetwork.net
    Address: 192.168.0.4
    ```

## <a name="configure-virtual-network-to-use-the-custom-dns-server"></a>Configurer le réseau virtuel pour qu’il utilise le serveur DNS personnalisé

Pour configurer le réseau virtuel afin d’utiliser le serveur DNS personnalisé au lieu du programme de résolution récursive Azure, effectuez les étapes suivantes dans le [portail Azure](https://portal.azure.com) :

1. Dans le menu gauche, accédez à **Tous les services** > **Mise en réseau** > **Réseaux virtuels**.

2. Sélectionnez votre réseau virtuel dans la liste afin d’ouvrir la vue par défaut de votre réseau virtuel.  

3. Dans la vue par défaut, sous **Paramètres**, sélectionnez **Serveurs DNS**.  

4. Sélectionnez __Personnalisé__, puis entrez l’**adresse IP privée** du serveur DNS personnalisé.

5. Sélectionnez __Enregistrer__.  <br />  

    ![Définir le serveur DNS personnalisé pour le réseau](./media/connect-on-premises-network/configure-custom-dns.png)

## <a name="configure-on-premises-dns-server"></a>Configurer le serveur DNS local

Dans la section précédente, vous avez configuré le serveur DNS personnalisé de façon à transférer les demandes au serveur DNS local. Ensuite, vous devez configurer le serveur DNS local pour transférer les demandes au serveur DNS personnalisé.

Pour savoir comment configurer votre serveur DNS, consultez la documentation relative au logiciel de votre serveur DNS. Recherchez la procédure à suivre pour configurer un __redirecteur conditionnel__.

Un redirecteur conditionnel transfère uniquement les demandes relatives à un suffixe DNS spécifique. En l’occurrence, vous devez configurer un redirecteur pour le suffixe DNS du réseau virtuel. Les demandes relatives à ce suffixe doivent être transmises à l’adresse IP du serveur DNS personnalisé. 

Le texte suivant est un exemple de configuration de redirecteur conditionnel pour le logiciel DNS **Bind** :

```DNS Zone file
zone "icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net" {
    type forward;
    forwarders {10.0.0.4;}; # The custom DNS server's internal IP address
};
```

Pour plus d’informations sur l’utilisation de DNS sur **Windows Server 2016**, consultez la documentation [Add-DnsServerConditionalForwarderZone](/powershell/module/dnsserver/add-dnsserverconditionalforwarderzone).

Une fois que vous avez configuré le serveur DNS local, vous pouvez utiliser `nslookup` à partir du réseau local pour vérifier que vous pouvez résoudre les noms dans le réseau virtuel. Par exemple : 

```bash
nslookup dnsproxy.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net 196.168.0.4
```

Cet exemple utilise le serveur DNS local à l’adresse 196.168.0.4 pour résoudre le nom du serveur DNS personnalisé. Remplacez l’adresse IP par celle du serveur DNS local. Remplacez l’adresse `dnsproxy` par le nom de domaine complet (FQDN) du serveur DNS personnalisé.

## <a name="optional-control-network-traffic"></a>Facultatif : contrôler le trafic réseau

Vous pouvez utiliser des groupes de sécurité réseau (NSG) ou des routages définis par l’utilisateur (UDR) pour contrôler le trafic réseau. Les groupes de sécurité réseau permettent de filtrer le trafic entrant et sortant, ainsi que d’autoriser ou refuser le trafic. Les routages définis par l’utilisateur vous permettent de contrôler le flux du trafic entre les ressources du réseau virtuel, Internet et le réseau local.

> [!WARNING]  
> HDInsight requiert un accès entrant en provenance d’adresses IP spécifiques dans le cloud Azure, et un accès sortant non restreint. Lorsque vous utilisez des groupes de sécurité réseau ou des routages définis par l’utilisateur pour contrôler le trafic, procédez comme suit :

1. Recherchez les adresses IP de l’emplacement contenant votre réseau virtuel. Pour obtenir la liste des adresses IP requises par emplacement, consultez [Adresses IP requises](./hdinsight-management-ip-addresses.md).

2. Pour les adresses IP identifiées à l’étape 1, autorisez le trafic entrant à partir de ces adresses.

   * Si vous utilisez l’option __NSG__ : autorisez le trafic __entrant__ sur le port __443__ pour les adresses IP.
   * Si vous utilisez l’option __UDR__ : définissez le type __Tronçon suivant__ de la route sur __Internet__ pour les adresses IP.

Pour un exemple d’utilisation d’Azure PowerShell ou d’Azure CLI afin de créer des groupes de sécurité réseau, consultez [Étendre HDInsight avec des réseaux virtuels Azure](hdinsight-create-virtual-network.md#hdinsight-nsg).

## <a name="create-the-hdinsight-cluster"></a>Créer le cluster HDInsight

> [!WARNING]  
> Vous devez configurer le serveur DNS personnalisé avant d’installer HDInsight sur le réseau virtuel.

Pour créer un cluster HDInsight, procédez de la manière décrite dans [Créer un cluster HDInsight à l’aide du portail Azure](./hdinsight-hadoop-create-linux-clusters-portal.md).

> [!WARNING]  
> * Lors de la création du cluster, vous devez choisir l’emplacement contenant votre réseau virtuel.
> * Dans les __Paramètres avancés__ de la configuration, vous devez sélectionner le réseau virtuel et le sous-réseau que vous avez créés précédemment.

## <a name="connecting-to-hdinsight"></a>Connexion à HDInsight

L’essentiel de la documentation relative à HDInsight part du principe que vous avez accès au cluster via Internet. Par exemple, vous devez pouvoir vous connecter au cluster à l’adresse `https://CLUSTERNAME.azurehdinsight.net`. Cette adresse utilise la passerelle publique qui n’est pas disponible si vous avez utilisé des groupes de sécurité réseau ou des itinéraires définis par l’utilisateur pour restreindre l’accès à partir d’Internet.

Certains documents font également référence à `headnodehost` lors de la connexion au cluster à partir d’une session SSH. Cette adresse est uniquement disponible à partir des nœuds d’un cluster, et elle n’est pas utilisable sur les clients connectés au réseau virtuel.

Pour vous connecter directement à HDInsight par le biais du réseau virtuel, procédez comme suit :

1. Pour découvrir les noms de domaine complets (FQDN) internes des nœuds de cluster HDInsight, utilisez l’une des méthodes suivantes :

    ```powershell
    $resourceGroupName = "The resource group that contains the virtual network used with HDInsight"

    $clusterNICs = Get-AzNetworkInterface -ResourceGroupName $resourceGroupName | where-object {$_.Name -like "*node*"}

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
    az network nic list --resource-group <resourcegroupname> --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

2. Pour déterminer le port sur lequel un service est disponible, consultez [Ports utilisés par les services Apache Hadoop dans HDInsight](./hdinsight-hadoop-port-settings-for-services.md).

    > [!IMPORTANT]  
    > Certains services hébergés sur les nœuds principaux ne sont actifs que sur un seul nœud à la fois. Si vous tentez d’accéder à un service sur un nœud principal et que l’opération échoue, basculez vers l’autre nœud principal.
    >
    > Par exemple, Apache Ambari n’est actif que sur un nœud principal à la fois. Si vous tentez d’accéder à Ambari sur un nœud principal et recevez une erreur 404, cela signifie qu’Ambari s’exécute sur l’autre nœud principal.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation de HDInsight sur un réseau virtuel, consultez [Planifier un déploiement de réseau virtuel pour des clusters Azure HDInsight](./hdinsight-plan-virtual-network-deployment.md).

* Pour plus d’informations sur les réseaux virtuels Azure, voir [Vue d'ensemble de Réseau virtuel Azure](../virtual-network/virtual-networks-overview.md).

* Pour plus d’informations sur les groupes de sécurité réseau, consultez [Groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).

* Pour plus d’informations sur les routages par l’utilisateur, consultez [Routage définis par l’utilisateur et transfert IP](../virtual-network/virtual-networks-udr-overview.md).