---
title: Réplication de cluster HBase dans des réseaux virtuels Azure – Azure HDInsight
description: Découvrez comment configurer la réplication HBase d’une version HDInsight à une autre pour l’équilibrage de charge, la haute disponibilité, la mise à jour et migration sans interruption de service , ainsi que la récupération d’urgence.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 12/06/2019
ms.openlocfilehash: 8fc5ba2280b5ad68a40f4992adc170408e80e5a6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021790"
---
# <a name="set-up-apache-hbase-cluster-replication-in-azure-virtual-networks"></a>Configurer la réplication de cluster Apache HBase dans les réseaux virtuels Azure

Découvrez comment configurer la réplication [Apache HBase](https://hbase.apache.org/) dans un réseau virtuel ou entre deux réseaux virtuels dans Azure.

La réplication en cluster utilise une méthodologie par émission de données source. Un cluster HBase peut être une source, une destination ou jouer les deux rôles à la fois. La réplication est asynchrone. L'objectif de la réplication est une cohérence éventuelle. Quand la source reçoit une modification apportée à une famille de colonnes lorsque la réplication activée, cette modification est propagée à tous les clusters de destination. Quand les données sont répliquées d’un cluster à un autre, le cluster source et tous les clusters qui ont déjà utilisé les données font l’objet d’un suivi afin d’empêcher les boucles de réplication.

À l’aide de cet article, vous allez configurer une réplication source-destination. Pour d'autres topologies de cluster, consultez le [Guide de référence d'Apache HBase](https://hbase.apache.org/book.html#_cluster_replication).

Cas d’utilisation de la réplication HBase pour un seul réseau virtuel :

* Équilibrage de charge : vous pouvez par exemple exécuter des analyses ou des travaux MapReduce sur le cluster de destination et ingérer des données sur le cluster source
* Ajout de haute disponibilité
* Migration de données d’un cluster HBase vers un autre
* Mise à niveau d’un cluster Azure HDInsight d’une version vers une autre

Cas d’utilisation de la réplication HBase pour deux réseaux virtuels :

* Configuration d’une récupération d'urgence
* Équilibrage de charge et partitionnement de l’application
* Ajout de haute disponibilité

Vous pouvez répliquer des clusters à l’aide de scripts [d’action de script](../hdinsight-hadoop-customize-cluster-linux.md) disponibles dans [GitHub](https://github.com/Azure/hbase-utils/tree/master/replication).

## <a name="prerequisites"></a>Conditions préalables requises
Avant de commencer cet article, vous devez disposer d’un abonnement Azure. Consultez [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

## <a name="set-up-the-environments"></a>Configurer les environnements

Il existe trois options de configuration :

- Deux clusters Apache HBase dans un réseau virtuel Azure
- Deux clusters Apache HBase dans deux réseaux virtuels différents dans la même région
- Deux clusters Apache HBase dans deux réseaux virtuels différents dans deux régions distinctes (géoréplication)

Cet article présente le scénario de géoréplication.

Pour vous aider à configurer les environnements, nous avons créé des [modèles Azure Resource Manager](../../azure-resource-manager/management/overview.md). Si vous préférez configurer les environnements en utilisant d’autres méthodes, consultez :

- [Créer des clusters Apache Hadoop dans HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)
- [Créer des clusters Apache HBase dans un réseau virtuel Azure](apache-hbase-provision-vnet.md)

### <a name="set-up-two-virtual-networks-in-two-different-regions"></a>Configurer deux réseaux virtuels dans deux régions différentes

Pour utiliser un modèle qui crée deux réseaux virtuels dans deux régions différentes et la connexion VPN entre les deux, sélectionnez le bouton suivant **Déployer sur Azure**. La définition de modèle est stockée dans un [stockage Blob public](https://hditutorialdata.blob.core.windows.net/hbaseha/azuredeploy.json).

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Fhbaseha%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-replication/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

Voici quelques-unes des valeurs codées en dur dans le modèle :

**Réseau virtuel 1**

| Propriété | Valeur |
|----------|-------|
| Location | USA Ouest |
| Nom du réseau virtuel | &lt;ClusterNamePrevix>-vnet1 |
| Préfixe de l’espace d’adressage | 10.1.0.0/16 |
| Nom du sous-réseau | subnet 1 |
| Préfixe du sous-réseau | 10.1.0.0/24 |
| Nom du sous-réseau (passerelle) | GatewaySubnet (non modifiable) |
| Préfixe du sous-réseau (passerelle) | 10.1.255.0/27 |
| Nom de la passerelle | vnet1gw |
| Type de passerelle | Vpn |
| Type de VPN de la passerelle | RouteBased |
| SKU de la passerelle | De base |
| Adresse IP de la passerelle | vnet1gwip |

**Réseau virtuel 2**

| Propriété | Valeur |
|----------|-------|
| Location | USA Est |
| Nom du réseau virtuel | &lt;ClusterNamePrevix>-vnet2 |
| Préfixe de l’espace d’adressage | 10.2.0.0/16 |
| Nom du sous-réseau | subnet 1 |
| Préfixe du sous-réseau | 10.2.0.0/24 |
| Nom du sous-réseau (passerelle) | GatewaySubnet (non modifiable) |
| Préfixe du sous-réseau (passerelle) | 10.2.255.0/27 |
| Nom de la passerelle | vnet2gw |
| Type de passerelle | Vpn |
| Type de VPN de la passerelle | RouteBased |
| SKU de la passerelle | De base |
| Adresse IP de la passerelle | vnet1gwip |

## <a name="setup-dns"></a>Configurer DNS

Dans la dernière section, le modèle crée une machine virtuelle Ubuntu dans chacun des deux réseaux virtuels.  Dans cette section, vous installez Bind sur les deux machines virtuelles DNS puis configurez la redirection DNS sur les deux machines virtuelles.

Pour installer Bind, vous devez rechercher l’adresse IP publique des deux machines virtuelles DNS.

1. Ouvrez le [portail Azure](https://portal.azure.com).
2. Ouvrez la machine virtuelle DNS en sélectionnant **Groupes de ressources > [nom du groupe de ressources] > [vnet1DNS]** .  Le nom du groupe de ressources est celui que vous créez dans la dernière procédure. Les noms des machines virtuelles DNS par défaut sont *vnet1DNS* et *vnet2NDS*.
3. Sélectionnez **Propriétés** pour ouvrir la page des propriétés du réseau virtuel.
4. Notez l’**adresse IP publique** et vérifiez l’**adresse IP privée**.  L’adresse IP privée doit être **10.1.0.4** pour vnet1DNS et **10.2.0.4** pour vnet2DNS.  
5. Remplacez les serveurs DNS des deux réseaux virtuels par les serveurs DNS par défaut (fournis par Azure) pour autoriser l’accès entrant et sortant afin de télécharger les packages visant à installer Bind dans les étapes suivantes.

Pour installer Bind, procédez comme suit :

1. Utilisez SSH pour vous connecter à l’__adresse IP publique__ de la machine virtuelle DNS. L’exemple suivant établit une connexion à une machine virtuelle dont l’adresse IP est 40.68.254.142 :

    ```bash
    ssh sshuser@40.68.254.142
    ```

    Remplacez `sshuser` par le compte d’utilisateur SSH que vous avez spécifié lors de la création de la machine virtuelle DNS.

    > [!NOTE]  
    > Il existe plusieurs façons d’obtenir l’utilitaire `ssh`. Sur Linux, Unix et macOS, il est généralement fourni en tant que composant du système d’exploitation. Si vous utilisez Windows, envisagez l’une des options suivantes :
    >
    > * [Azure Cloud Shell](../../cloud-shell/quickstart.md)
    > * [Bash sur Ubuntu sur Windows 10](/windows/wsl/about)
    > * [Git (https://git-scm.com/)](https://git-scm.com/)
    > * [OpenSSH (https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)

2. Pour installer Bind, utilisez les commandes suivantes à partir de la session SSH :

    ```bash
    sudo apt-get update -y
    sudo apt-get install bind9 -y
    ```

3. Configurez Bind afin de transférer les demandes de résolution de noms à votre serveur DNS local. Pour ce faire, utilisez le texte suivant comme contenu du fichier `/etc/bind/named.conf.options` :

    ```
    acl goodclients {
        10.1.0.0/16; # Replace with the IP address range of the virtual network 1
        10.2.0.0/16; # Replace with the IP address range of the virtual network 2
        localhost;
        localhost;
    };
    
    options {
        directory "/var/cache/bind";
        recursion yes;
        allow-query { goodclients; };

        forwarders {
            168.63.129.16; #This is the Azure DNS server
        };

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
    };
    ```
    
    > [!IMPORTANT]  
    > Remplacez les valeurs de la section `goodclients` par la plage d’adresses IP des deux réseaux virtuels. Cette section définit les adresses d’origine des demandes que ce serveur DNS accepte.

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
    vnet1DNS.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net
    ```

    Le texte `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` est le __suffixe DNS__ de ce réseau virtuel. Enregistrez cette valeur, car vous l’utiliserez plus tard.

    Vous devez également déterminer le suffixe DNS de l’autre serveur DNS. Vous en aurez besoin à l’étape suivante.

5. Pour configurer Bind afin de résoudre les noms DNS pour les ressources au sein du réseau virtuel, utilisez le texte suivant en tant que contenu du fichier `/etc/bind/named.conf.local` :

    ```
    // Replace the following with the DNS suffix for your virtual network
    zone "v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net" {
            type forward;
            forwarders {10.2.0.4;}; # The Azure recursive resolver
    };
    ```

    > [!IMPORTANT]  
    > Vous devez remplacer `v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net` par le suffixe DNS de l’autre réseau virtuel. Et l’adresse IP de redirection est l’adresse IP privée du serveur DNS de l’autre réseau virtuel.

    Pour modifier ce fichier, utilisez la commande suivante :

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    Pour enregistrer le fichier, appuyez sur __Ctrl+X__, __O__, puis __Entrée__.

6. Pour démarrer Bind, utilisez la commande suivante :

    ```bash
    sudo service bind9 restart
    ```

7. Pour vérifier que Bind peut résoudre les noms de ressources de l’autre réseau virtuel, utilisez les commandes suivantes :

    ```bash
    sudo apt install dnsutils
    nslookup vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net
    ```

    > [!IMPORTANT]  
    > Remplacez `vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net` par le nom de domaine complet (FQDN) de la machine virtuelle DNS de l’autre réseau.
    >
    > Remplacez `10.2.0.4` par __l’adresse IP interne__ de votre serveur DNS personnalisé dans l’autre réseau virtuel.

    La réponse est similaire au texte suivant :

    ```output
    Server:         10.2.0.4
    Address:        10.2.0.4#53
    
    Non-authoritative answer:
    Name:   vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net
    Address: 10.2.0.4
    ```

    Jusqu'à présent, vous ne pouviez pas rechercher l’adresse IP à partir de l’autre réseau sans l’adresse IP du serveur DNS.

### <a name="configure-the-virtual-network-to-use-the-custom-dns-server"></a>Configurer le réseau virtuel pour utiliser le serveur DNS personnalisé

Pour configurer le réseau virtuel afin d’utiliser le serveur DNS personnalisé au lieu du programme de résolution récursive Azure, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez le réseau virtuel, puis choisissez __Serveurs DNS__.

2. Sélectionnez __Personnalisé__, puis entrez l’__adresse IP interne__ du serveur DNS personnalisé. Enfin, sélectionnez __Enregistrer__.

6. Ouvrez la machine virtuelle du serveur DNS dans vnet1, puis cliquez sur **Redémarrer**.  Vous devez redémarrer toutes les machines virtuelles dans le réseau virtuel pour appliquer la configuration DNS.
7. Répétez les étapes pour configurer le serveur DNS personnalisé pour vnet2.

Pour tester la configuration DNS, vous pouvez vous connecter aux deux machines virtuelles DNS à l’aide de SSH, puis effectuez un test ping sur le serveur DNS de l’autre réseau virtuel à l’aide de son nom d’hôte. En cas d’échec, utilisez la commande suivante pour vérifier l’état DNS :

```bash
sudo service bind9 status
```

## <a name="create-apache-hbase-clusters"></a>Créer des clusters Apache HBase

Créez un cluster [Apache HBase](https://hbase.apache.org/) dans chacun des deux réseaux virtuels avec la configuration suivante :

- **Nom du groupe de ressources** : utilisez le même nom de groupe de ressources que vous avez créé pour les réseaux virtuels.
- **Type de cluster** : HBase
- **Version** : HBase 1.1.2 (HDI 3.6)
- **Emplacement** : utilisez l’emplacement du réseau virtuel.  Par défaut, vnet1 représente *USA Ouest*, et vnet2 correspond à *USA Est*.
- **Stockage** : créez un nouveau compte de stockage pour le cluster.
- **Réseau virtuel** (dans les paramètres avancés sur le portail) : sélectionnez l’élément vnet1 que vous avez créé dans la dernière procédure.
- **Sous-réseau** : le nom par défaut utilisé dans le modèle est **subnet1**.

Pour garantir que l’environnement est correctement configuré, vous devez pouvoir effectuer un test ping sur le nom de domaine complet du nœud principal entre les deux clusters.

## <a name="load-test-data"></a>Charger les données de test

Lorsque vous répliquez un cluster, vous devez spécifier les tables à répliquer. Dans cette section, vous chargez des données dans le cluster source. Dans la section suivante, vous allez activer la réplication entre les deux clusters.

Pour créer une table **Contacts** et y insérer des données, suivez les instructions du [Tutoriel Apache HBase : Bien démarrer avec Apache HBase dans HDInsight](apache-hbase-tutorial-get-started-linux.md).

> [!NOTE]
> Si vous voulez répliquer des tables à partir d’un espace de noms personnalisé, vous devez vous assurer que les espaces de noms personnalisés appropriés sont également définis sur le cluster de destination.
>

## <a name="enable-replication"></a>Activer la réplication

Les étapes suivantes décrivent comment appeler le script d’action de script à partir du portail Azure. Pour savoir comment exécuter une action de script en utilisant Azure PowerShell et Azure Classic CLI, consultez [Personnaliser des clusters HDInsight à l’aide d’une action de script](../hdinsight-hadoop-customize-cluster-linux.md).

**Pour activer la réplication HBase à partir du portail Azure**

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Ouvrez le cluster HBase source.
3. Dans le menu de cluster, sélectionnez **Actions de script**.
4. En haut de la page, sélectionnez **Soumettre nouveau**.
5. Sélectionnez ou saisissez les informations suivantes :

   1. **Nom** : saisissez **Activer la réplication**.
   2. **URL du script Bash** : entrez **https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_enable_replication.sh** .
   3. **Principal** : assurez-vous que cette option est sélectionnée. Supprimez les autres types de nœuds.
   4. **Paramètres** : les paramètres d’exemple suivants activent la réplication pour toutes les tables existantes, puis copient toutes les données du cluster source vers le cluster de destination :

    `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -copydata`
    
      > [!NOTE]
      > Utilisez le nom d’hôte plutôt que le nom de domaine complet (FQDN) pour le nom DNS du cluster source et du cluster de destination.
      >
      > Cette procédure pas à pas suppose que hn1 est le nœud principal actif. Vérifiez votre cluster pour identifier le nœud principal actif.

6. Sélectionnez **Create** (Créer). L’exécution du script peut prendre un certain temps, en particulier lorsque vous utilisez l’argument **-copydata**.

Arguments requis :

|Name|Description|
|----|-----------|
|-s, --src-cluster | Spécifie le nom DNS du cluster HBase source. Par exemple : -s hbsrccluster, --src-cluster=hbsrccluster |
|-d, --dst-cluster | Spécifie le nom DNS du cluster HBase de destination (réplica). Par exemple : -s dsthbcluster, --src-cluster=dsthbcluster |
|-sp, --src-ambari-password | Spécifie le mot de passe d’administrateur pour Ambari sur le cluster HBase source. |
|-dp, --dst-ambari-password | Spécifie le mot de passe d’administrateur pour Ambari sur le cluster HBase de destination.|

Arguments facultatifs :

|Name|Description|
|----|-----------|
|-su, --src-ambari-user | Spécifie le nom d’utilisateur administrateur pour Ambari sur le cluster HBase source. La valeur par défaut est **admin**. |
|-du, --dst-ambari-user | Spécifie le nom d’utilisateur administrateur pour Ambari sur le cluster HBase de destination. La valeur par défaut est **admin**. |
|-t, --table-list | Spécifie les tables à répliquer. Par exemple : --table-list="table1;table2;table3". Si vous ne spécifiez pas de tables, toutes les tables HBase existantes sont répliquées.|
|-m, --machine | Spécifie le nœud principal sur lequel l’action de script s’exécute. La valeur doit être choisie en fonction du nœud principal actif. Utilisez cette option lorsque vous exécutez le script $0 comme une action de script à partir du portail HDInsight ou d’Azure PowerShell.|
|-cp, -copydata | Active la migration des données existantes sur les tables pour lesquelles la réplication est activée. |
|-rpm, -replicate-phoenix-meta | Active la réplication sur les tables système Phoenix. <br><br>*Utilisez cette option avec précaution.* Nous vous recommandons de recréer des tables Phoenix sur les clusters de réplica avant d’utiliser ce script. |
|-h, --help | Affiche des informations sur l’utilisation. |

La section `print_usage()` du [script](https://github.com/Azure/hbase-utils/blob/master/replication/hdi_enable_replication.sh) offre une explication détaillée des paramètres.

Une fois le déploiement de l’action de script terminé, vous pouvez utiliser SSH pour vous connecter au cluster HBase de destination, puis vérifier que les données ont été répliquées.

### <a name="replication-scenarios"></a>Scénarios de réplication

La liste suivante présente certains cas d’utilisation générale et la définition de leurs paramètres :

- **Activer la réplication sur toutes les tables entre les deux clusters**. Ce scénario ne nécessite pas la copie ou migration des données existantes dans les tables et n’utilise pas de tables Phoenix. Utilisez les paramètres suivants :

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password>`

- **Activer la réplication sur des tables spécifiques**. Pour activer la réplication sur table1, table2 et table3, utilisez les paramètres suivants :

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3"`

- **Activer la réplication sur des tables spécifiques et copier les données existantes**. Pour activer la réplication sur table1, table2 et table3, utilisez les paramètres suivants :

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3" -copydata`

- **Activer la réplication sur toutes les tables et répliquer les métadonnées Phoenix de la source vers la destination**. La réplication des métadonnées Phoenix n’est pas parfaite. Utilisez-la avec précaution. Utilisez les paramètres suivants :

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3" -replicate-phoenix-meta`

## <a name="copy-and-migrate-data"></a>Copier et migrer des données

Il existe deux scripts d’action de script distincts disponibles pour copier ou migrer des données une fois la réplication activée :

- [Script pour les petites tables](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_copy_table.sh) (tables dont la taille est de quelques gigaoctets et dont la copie globale est censée durer moins d’une heure)

- [Script pour les tables volumineuses](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/nohup_hdi_copy_table.sh) (tables dont la copie est censée durer plus d’une heure)

Vous pouvez suivre la même procédure que celle décrite dans la section [Activer la réplication](#enable-replication) pour appeler l’action de script. Utilisez les paramètres suivants :

`-m hn1 -t <table1:start_timestamp:end_timestamp;table2:start_timestamp:end_timestamp;...> -p <replication_peer> [-everythingTillNow]`

La section `print_usage()` du [script](https://github.com/Azure/hbase-utils/blob/master/replication/hdi_copy_table.sh) offre une description détaillée des paramètres.

### <a name="scenarios"></a>Scénarios

- **Copier des tables spécifiques (test1, test2 et test3) pour toutes les lignes modifiées jusqu’à présent (horodatage actuel)**  :

  `-m hn1 -t "test1::;test2::;test3::" -p "zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure" -everythingTillNow`

  Ou :

  `-m hn1 -t "test1::;test2::;test3::" --replication-peer="zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure" -everythingTillNow`

- **Copier des tables spécifiques avec une plage horaire spécifiée** :

  `-m hn1 -t "table1:0:452256397;table2:14141444:452256397" -p "zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure"`

## <a name="disable-replication"></a>Désactiver la réplication

Pour désactiver la réplication, utilisez un autre script d’action de script disponible dans [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_disable_replication.sh). Vous pouvez suivre la même procédure que celle décrite dans la section [Activer la réplication](#enable-replication) pour appeler l’action de script. Utilisez les paramètres suivants :

`-m hn1 -s <source hbase cluster name> -sp <source cluster Ambari password> <-all|-t "table1;table2;...">`

La section `print_usage()` du [script](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_disable_replication.sh) offre une explication détaillée des paramètres.

### <a name="scenarios"></a>Scénarios

- **Désactiver la réplication sur toutes les tables** :

  `-m hn1 -s <source hbase cluster name> -sp Mypassword\!789 -all`

  or

  `--src-cluster=<source hbase cluster name> --dst-cluster=<destination hbase cluster name> --src-ambari-user=<source cluster Ambari user name> --src-ambari-password=<source cluster Ambari password>`

- **Désactiver la réplication sur les tables spécifiées (table1, table2 et table3)**  :

  `-m hn1 -s <source hbase cluster name> -sp <source cluster Ambari password> -t "table1;table2;table3"`

> [!NOTE]
> Si vous envisagez de supprimer le cluster de destination, veillez à le faire à partir de la liste des pairs du cluster source. Pour cela, vous pouvez exécuter la commande remove_peer '1' dans l’interpréteur de commandes hbase sur le cluster source. Si vous ne procédez pas ainsi, le cluster source risque de ne pas fonctionner correctement.
>

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez vu comment configurer la réplication Apache HBase dans un réseau virtuel ou entre deux réseaux virtuels. Pour plus d’informations sur HDInsight et Apache HBase, consultez les articles suivants :

* [Didacticiel HBase : prise en main de HBase avec Hadoop dans HDInsight Linux](./apache-hbase-tutorial-get-started-linux.md)
* [Vue d’ensemble de HDInsight Apache HBase](./apache-hbase-overview.md)
* [Créer des clusters Apache HBase dans un réseau virtuel Azure](./apache-hbase-provision-vnet.md)