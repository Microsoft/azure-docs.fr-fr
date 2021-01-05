---
title: Configurer un réseau virtuel - Azure Cache pour Redis Premium
description: Découvrez comment créer et gérer la prise en charge de réseau virtuel pour vos instances de Cache Azure pour Redis de niveau Premium
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.date: 10/09/2020
ms.openlocfilehash: 5fd82105c94bb9be2d07c8843834465821acd8bc
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95803774"
---
# <a name="how-to-configure-virtual-network-support-for-a-premium-azure-cache-for-redis"></a>Comment configurer la prise en charge de réseau virtuel pour un Cache Azure Premium pour Redis
Le Cache Azure pour Redis offre différents types de caches permettant de choisir parmi plusieurs tailles et fonctionnalités de caches, notamment les fonctionnalités de niveau Premium telles que le clustering, la persistance et la prise en charge du réseau virtuel. Un réseau VNet est un réseau privé dans le cloud. Lorsqu’une instance de Cache Azure pour Redis est configurée avec un réseau virtuel, elle n’est pas adressable publiquement et est accessible uniquement à partir de machines virtuelles et d’applications sur le réseau virtuel. Cet article décrit comment configurer la prise en charge de réseau virtuel pour une instance Premium de Cache Azure pour Redis.

> [!NOTE]
> Le Cache Azure pour Redis prend en charge les réseaux virtuels classiques et de gestionnaire de ressources.
> 
> 

## <a name="why-vnet"></a>Pourquoi un réseau virtuel ?
Le déploiement [Réseau virtuel Azure](https://azure.microsoft.com/services/virtual-network/) fournit une sécurité et une isolation améliorées pour le cache Azure pour Redis, ainsi que des sous-réseaux, des stratégies de contrôle d’accès et d’autres fonctionnalités permettant de restreindre davantage l’accès.

## <a name="virtual-network-support"></a>Prise en charge des réseaux virtuels
La configuration de la prise en charge du réseau virtuel (VNet) s’effectue dans le panneau **Nouveau cache Azure pour Redis** lors de la création du cache. 

1. Pour créer un cache Premium, connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Créer une ressource**. Notez que, en plus de créer des caches dans le portail Azure, vous pouvez en créer à l’aide de modèles Resource Manager, de PowerShell ou d’Azure CLI. Pour plus d’informations sur la création d’un cache Azure pour Redis, consultez la section [Création d’un cache](cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).

    :::image type="content" source="media/cache-private-link/1-create-resource.png" alt-text="Créer une ressource.":::
   
2. Dans la page **Nouvelle**, sélectionnez **Bases de données**, puis **Azure Cache pour Redis**.

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Sélectionnez Azure Cache pour Redis.":::

3. Dans la page **Nouveau cache Redis**, configurez les paramètres du nouveau cache Premium.
   
   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Nom DNS** | Entrez un nom globalement unique. | Le nom du cache doit être une chaîne de 1 à 63 caractères ne contenant que des chiffres, des lettres et des traits d’union. Le nom doit commencer et se terminer par un chiffre ou une lettre, et ne peut pas contenir de traits d’union consécutifs. Le *nom d’hôte* de votre instance de cache sera *\<DNS name>.redis.cache.windows.net*. | 
   | **Abonnement** | Dans la liste déroulante, sélectionnez votre abonnement. | Abonnement sous lequel créer cette nouvelle instance d’Azure Cache pour Redis. | 
   | **Groupe de ressources** | Dans la liste déroulante, sélectionnez un groupe de ressources ou choisissez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre cache et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
   | **Lieu** | Dans la liste déroulante, sélectionnez un emplacement. | Choisissez une [Région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre cache. |
   | **Type de cache** | Dans la liste déroulante, sélectionnez un cache Premium pour configurer les fonctionnalités Premium. Pour plus d’informations, consultez la page [Tarification Azure Cache pour Redis](https://azure.microsoft.com/pricing/details/cache/). |  Le niveau tarifaire détermine la taille, les performances et les fonctionnalités disponibles pour le cache. Pour plus d’informations, consultez [Présentation du cache Azure pour Redis](cache-overview.md). |

4. Sélectionnez l’onglet **Réseau** ou cliquez sur le bouton **Réseau** au bas de la page.

5. Sous l’onglet **Réseau**, sélectionnez **Réseaux virtuels** comme méthode de connectivité. Pour utiliser un nouveau réseau virtuel, vous devez tout d’abord le créer en suivant les étapes indiquées dans [Créer un réseau virtuel à l’aide du portail Azure](../virtual-network/manage-virtual-network.md#create-a-virtual-network) ou [Créer un réseau virtuel (classique) à l’aide du portail Azure](/previous-versions/azure/virtual-network/virtual-networks-create-vnet-classic-pportal). Revenez ensuite au panneau **Nouveau cache Azure pour Redis** pour créer et configurer votre cache Premium.

   > [!IMPORTANT]
   > Lorsque vous déployez un Cache Azure pour Redis vers un réseau virtuel Gestionnaire des ressources, le cache doit se trouver dans un sous-réseau dédié ne contenant pas de ressources autres que des instances du Cache Azure pour Redis. Si vous tentez de déployer un Cache Azure pour Redis sur un réseau virtuel Gestionnaire des ressources vers un sous-réseau contenant d’autres ressources, le déploiement échoue.
   > 
   > 

   | Paramètre      | Valeur suggérée  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Réseau virtuel** | Dans la liste déroulante, sélectionnez votre réseau virtuel. | Sélectionnez un réseau virtuel qui se trouve dans le même abonnement et au même emplacement que votre cache. | 
   | **Sous-réseau** | Dans la liste déroulante, sélectionnez votre sous-réseau. | La plage d’adresses du sous-réseau doit être en notation CIDR (par exemple, 192.168.1.0/24). Elle doit être contenue dans l’espace d’adressage du réseau virtuel. | 
   | **Adresse IP statique** | (Facultatif) Entrez une adresse IP statique. | Si vous ne spécifiez pas d’adresse IP statique, une adresse IP est choisie automatiquement. | 

   > [!IMPORTANT]
   > Azure réserve dans chaque sous-réseau des adresses IP qui ne peuvent pas être utilisées. Les première et dernière adresse IP des sous-réseaux sont réservées à la conformité du protocole, et 3 adresses supplémentaires sont utilisées pour les services Azure. Pour plus d’informations, consultez [Existe-t-il des restrictions sur l’utilisation des adresses IP au sein de ces sous-réseaux ?](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)
   > 
   > Outre les adresses IP utilisées par l’infrastructure réseau virtuel Azure, chaque d’instance Redis dans le sous-réseau utilise deux adresses IP par partition et une adresse IP supplémentaire pour l’équilibrage de charge. Un cache non cluster est considéré comme ayant une seule partition.
   > 
   > 

6. Sélectionnez le bouton **Suivant : Avancé** ou cliquez sur le bouton **Suivant : Avancé** en bas de la page.

7. Sous l’onglet **Avancé** d’une instance de cache Premium, configurez les paramètres pour le port non TLS, le clustering et la persistance des données. 

8. Sélectionnez le bouton **Suivant : Étiquettes** ou cliquez sur le bouton **Suivant : Étiquettes** au bas de la page.

9. Si vous le voulez, sous l’onglet **Étiquettes**, entrez le nom et la valeur si vous souhaitez catégoriser la ressource. 

10. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers l’onglet Vérifier + créer où Azure valide votre configuration.

11. Une fois que le message vert Validation réussie s’affiche, sélectionnez **Créer**.

La création du cache prend un certain temps. Vous pouvez surveiller la progression dans la page **Vue d’ensemble** du Azure Cache pour Redis. Lorsque **État** indique **En cours d’exécution**, le cache est prêt pour utilisation. Une fois le cache créé, vous pouvez afficher la configuration du réseau virtuel en cliquant sur **Réseau virtuel** dans le **menu Ressource**.

![Réseau virtuel][redis-cache-vnet-info]

Pour vous connecter à votre instance du Cache Azure pour Redis lorsque vous utilisez un réseau virtuel, spécifiez le nom d’hôte de votre cache dans la chaîne de connexion, comme indiqué dans l’exemple suivant :

```csharp
private static Lazy<ConnectionMultiplexer>
    lazyConnection = new Lazy<ConnectionMultiplexer> (() =>
    {
        return ConnectionMultiplexer.Connect("contoso5premium.redis.cache.windows.net,abortConnect=false,ssl=true,password=password");
    });

public static ConnectionMultiplexer Connection
{
    get
    {
        return lazyConnection.Value;
    }
}
```

## <a name="azure-cache-for-redis-vnet-faq"></a>FAQ sur les réseaux virtuels de cache Azure pour Redis
La liste suivante présente différentes réponses aux questions les plus fréquemment posées sur la mise à l’échelle du cache Azure pour Redis.

* Quels sont les problèmes de configuration les plus courants au niveau du Cache Azure pour Redis et des réseaux virtuels ?
* [Comment puis-je vérifier que mon cache fonctionne dans un réseau virtuel ?](#how-can-i-verify-that-my-cache-is-working-in-a-vnet)
* Lorsque j’essaie de me connecter à mon cache Azure pour Redis dans un réseau virtuel, pourquoi je reçois un message d’erreur indiquant que le certificat distant n’est pas valide ?
* [Puis-je utiliser des réseaux virtuels avec un cache De base ou Standard ?](#can-i-use-vnets-with-a-standard-or-basic-cache)
* Pourquoi la création d’un cache Azure pour Redis échoue-t-elle dans certains sous-réseaux mais pas d’autres ?
* [Quelles sont les exigences d’espace d’adressage du sous-réseau ?](#what-are-the-subnet-address-space-requirements)
* [Toutes les fonctionnalités fonctionnent-elles lorsque vous hébergez un cache dans un réseau virtuel ?](#do-all-cache-features-work-when-hosting-a-cache-in-a-vnet)

### <a name="what-are-some-common-misconfiguration-issues-with-azure-cache-for-redis-and-vnets"></a>Quels sont les problèmes de configuration les plus courants au niveau du Cache Azure pour Redis et des réseaux virtuels ?
Lorsque le Cache Azure pour Redis est hébergé dans un réseau virtuel, les ports dans les tableaux suivants sont utilisés. 

>[!IMPORTANT]
>Si ces ports dans les tableaux suivants sont bloqués, le cache risque de ne pas fonctionner correctement. Le blocage d’un ou plusieurs de ces ports constitue le problème de configuration le plus courant lorsque vous utilisez le Cache Azure pour Redis dans un réseau virtuel.
> 
> 

- [Configuration requise de port sortant](#outbound-port-requirements)
- [Configuration requise de port entrant](#inbound-port-requirements)

#### <a name="outbound-port-requirements"></a>Configuration requise de port sortant

Il existe neuf configurations requises de port sortant. Les demandes sortantes dans ces plages sont soit sortantes vers d’autres services nécessaires au cache pour fonctionner, soit internes au sous-réseau Redis pour la communication entre les nœuds. Pour la géoréplication, il existe d’autres règles de trafic sortant pour la communication entre les sous-réseaux du cache principal et de réplica.

| Port(s) | Sens | Protocole de transfert | Objectif | Adresse IP locale | Adresse IP distante |
| --- | --- | --- | --- | --- | --- |
| 80, 443 |Règle de trafic sortant |TCP |Dépendances Redis sur Azure Storage/l’infrastructure à clé publique (Internet) | (sous-réseau Redis) |* <sup>4</sup> |
| 443 | Règle de trafic sortant | TCP | Dépendance Redis à Azure Key Vault et à Azure Monitor | (sous-réseau Redis) | AzureKeyVault, AzureMonitor<sup>1</sup> |
| 53 |Règle de trafic sortant |TCP/UDP |Dépendances Redis sur le DNS (Internet/réseau virtuel) | (sous-réseau Redis) | 168.63.129.16 et 169.254.169.254 <sup>2</sup> et n’importe quel serveur DNS personnalisé pour le sous-réseau <sup>3</sup> |
| 8443 |Règle de trafic sortant |TCP |Communications internes pour Redis | (sous-réseau Redis) | (sous-réseau Redis) |
| 10221-10231 |Règle de trafic sortant |TCP |Communications internes pour Redis | (sous-réseau Redis) | (sous-réseau Redis) |
| 20226 |Règle de trafic sortant |TCP |Communications internes pour Redis | (sous-réseau Redis) |(sous-réseau Redis) |
| 13000-13999 |Règle de trafic sortant |TCP |Communications internes pour Redis | (sous-réseau Redis) |(sous-réseau Redis) |
| 15000-15999 |Règle de trafic sortant |TCP |Communications internes pour Redis et géoréplication | (sous-réseau Redis) |(Sous-réseau Redis) (Sous-réseau d’homologues géo-réplica) |
| 6379-6380 |Règle de trafic sortant |TCP |Communications internes pour Redis | (sous-réseau Redis) |(sous-réseau Redis) |

<sup>1</sup> Vous pouvez utiliser les étiquettes de service « AzureKeyVault » et « AzureMonitor » avec des groupes de sécurité réseau Resource Manager.

<sup>2</sup> Ces adresses IP appartenant à Microsoft sont utilisées pour adresser la machine virtuelle hôte qui sert Azure DNS.

<sup>3</sup> Non nécessaire pour les sous-réseaux sans serveur DNS personnalisé, ou pour des caches Redis plus récentes qui ignorent les DNS personnalisés.

<sup>4</sup> Pour plus d’informations, consultez [Exigences supplémentaires de connectivité réseau VNET](#additional-vnet-network-connectivity-requirements).

#### <a name="geo-replication-peer-port-requirements"></a>Exigences relatives aux ports homologues de géoréplication

Si vous utilisez la géoréplication entre caches dans des réseaux virtuels Azure, notez que la configuration recommandée consiste à débloquer les ports 15000 à 15999 pour l'ensemble du sous-réseau dans les directions entrantes ET sortantes vers les deux caches et ce, afin de permettre à tous les composants de réplica du sous-réseau de communiquer directement entre eux, même en cas de basculement géographique.

#### <a name="inbound-port-requirements"></a>Configuration requise des ports entrants

Il existe huit configurations requises de port entrant. Les requêtes entrantes dans ces plages proviennent d’autres services hébergés dans le même réseau virtuel ou des communications sur sous-réseau Redis interne.

| Port(s) | Sens | Protocole de transfert | Objectif | Adresse IP locale | Adresse IP distante |
| --- | --- | --- | --- | --- | --- |
| 6379, 6380 |Trafic entrant |TCP |Communication client avec Redis, équilibrage de charge Azure | (sous-réseau Redis) | (sous-réseau Redis), (sous-réseau client), AzureLoadBalancer <sup>1</sup> |
| 8443 |Trafic entrant |TCP |Communications internes pour Redis | (sous-réseau Redis) |(sous-réseau Redis) |
| 8500 |Trafic entrant |TCP/UDP |Équilibrage de charge Azure | (sous-réseau Redis) | AzureLoadBalancer |
| 10221-10231 |Trafic entrant |TCP |Communication client avec les clusters Redis, communications internes pour Redis | (sous-réseau Redis) |(Sous-réseau Redis), AzureLoadBalancer, (Sous-réseau client) |
| 13000-13999 |Trafic entrant |TCP |Communication client avec les clusters Redis, équilibrage de charge Azure | (sous-réseau Redis) | (sous-réseau Redis), (sous-réseau client), AzureLoadBalancer |
| 15000-15999 |Trafic entrant |TCP |Communication client avec les clusters Redis, équilibrage de charge Azure et géoréplication | (sous-réseau Redis) | (sous-réseau Redis), (sous-réseau client), AzureLoadBalancer, (sous-réseau homologue avec géoréplication) |
| 16001 |Trafic entrant |TCP/UDP |Équilibrage de charge Azure | (sous-réseau Redis) | AzureLoadBalancer |
| 20226 |Trafic entrant |TCP |Communications internes pour Redis | (sous-réseau Redis) |(sous-réseau Redis) |

<sup>1</sup> Vous pouvez utiliser la balise de service « AzureLoadBalancer » (Resource Manager) (ou « AZURE_LOADBALANCER » pour le mode classique) dans la création de règles de groupe de sécurité réseau.

#### <a name="additional-vnet-network-connectivity-requirements"></a>Conditions supplémentaires pour la connectivité réseau VNET

Il existe des exigences de connectivité réseau pour le Cache Azure pour Redis qui peuvent ne pas être initialement satisfaites dans un réseau virtuel. Le Cache Azure pour Redis nécessite tous les éléments suivants pour fonctionner correctement lorsqu’il est utilisé dans un réseau virtuel.

* Connectivité réseau sortante à des points de terminaison Azure Storage dans le monde entier. Cela inclut les points de terminaison situés dans la même région que le cache Azure pour Redis, ainsi que les points de terminaison de stockage situés dans d’**autres** régions Azure. Les points de terminaison du Stockage Azure se résolvent dans les domaines DNS suivants : *table.core.windows.net*, *blob.core.windows.net*, *queue.core.windows.net* et *file.core.windows.net*. 
* Connectivité réseau sortante à *ocsp.digicert.com*, *crl4.digicert.com*, *ocsp.msocsp.com*, *mscrl.microsoft.com*, *crl3.digicert.com*, *cacerts.digicert.com*, *oneocsp.microsoft.com* et *crl.microsoft.com*. Cette connectivité est nécessaire pour prendre en charge la fonctionnalité TSL/SSL.
* La configuration DNS pour le réseau virtuel doit être capable de résoudre tous les points de terminaison et les domaines mentionnés dans les points précédents. La configuration DNS requise peut être satisfaite en garantissant qu'une infrastructure DNS valide est configurée et gérée pour le réseau virtuel.
* Connectivité réseau sortante aux points de terminaison Azure Monitor suivants qui se résolvent dans les domaines DNS suivants : *shoebox2-black.shoebox2.metrics.nsatc.net*, *north-prod2.prod2.metrics.nsatc.net*, *azglobal-black.azglobal.metrics.nsatc.net*, *shoebox2-red.shoebox2.metrics.nsatc.net*, *east-prod2.prod2.metrics.nsatc.net* et *azglobal-red.azglobal.metrics.nsatc.net*.

### <a name="how-can-i-verify-that-my-cache-is-working-in-a-vnet"></a>Comment puis-je vérifier que mon cache fonctionne dans un réseau virtuel ?

>[!IMPORTANT]
>Lors de la connexion à une instance Azure Cache pour Redis hébergée dans un réseau virtuel, vos clients de cache doivent figurer dans le même réseau virtuel ou dans un réseau virtuel pour lequel le VNET Peering est activé dans la même région Azure. Le VNET Peering global n’est pas pris en charge actuellement. Cela inclut les applications de test ou les outils de requête de diagnostic. Quel que soit l’endroit où l’application cliente est hébergée, les groupes de sécurité réseau ou d’autres couches réseau doivent être configurés de façon à ce que le trafic du client soit autorisé à atteindre l’instance Redis.
>
>

Une fois les exigences de port configurées comme décrit dans la section précédente, vous pouvez vérifier que votre cache fonctionne en procédant comme suit.

- [Redémarrez](cache-administration.md#reboot) tous les nœuds de cache. Si toutes les dépendances de cache requises sont inaccessibles (comme décrit dans [Configuration requise des ports entrants](cache-how-to-premium-vnet.md#inbound-port-requirements) et [Configuration requise des ports sortants](cache-how-to-premium-vnet.md#outbound-port-requirements)), le cache ne redémarre plus correctement.
- Une fois les nœuds de cache redémarrés (comme indiqué par l’état du cache dans le portail Azure), vous pouvez effectuer les tests suivants :
  - Vous pouvez effectuer un test ping du point de terminaison du cache (en utilisant le port 6380) à partir d’une machine figurant dans le même réseau virtuel que le cache, à l’aide de la commande [tcping](https://www.elifulkerson.com/projects/tcping.php). Par exemple :
    
    `tcping.exe contosocache.redis.cache.windows.net 6380`
    
    Si l’outil `tcping` signale que le port est ouvert, cela signifie que le cache est disponible pour connexion à partir de clients dans le réseau virtuel.

  - Un autre test consiste à créer un client de cache de test (qui peut être une simple application console utilisant StackExchange.Redis) qui se connecte au cache et ajoute et récupère des éléments de celui-ci. Installez l’exemple d’application cliente sur une machine virtuelle figurant dans le même réseau virtuel que le cache, puis exécutez l’application pour vérifier la connectivité au cache.


### <a name="when-trying-to-connect-to-my-azure-cache-for-redis-in-a-vnet-why-am-i-getting-an-error-stating-the-remote-certificate-is-invalid"></a>Lorsque j’essaie de me connecter à mon cache Azure pour Redis dans un réseau virtuel, pourquoi je reçois un message d’erreur indiquant que le certificat distant n’est pas valide ?

Lorsque vous tentez de vous connecter à un cache Azure pour Redis dans un réseau virtuel, vous voyez une erreur de validation de certificat comme celle-ci :

`{"No connection is available to service this operation: SET mykey; The remote certificate is invalid according to the validation procedure.; …"}`

La cause peut être que vous vous connectez à l’hôte par le biais de l’adresse IP. Nous vous recommandons d’utiliser le nom d’hôte. En d’autres termes, utilisez les éléments suivants :     

`[mycachename].redis.windows.net:6380,password=xxxxxxxxxxxxxxxxxxxx,ssl=True,abortConnect=False`

Évitez d’utiliser l’adresse IP similaire à la chaîne de connexion suivante :

`10.128.2.84:6380,password=xxxxxxxxxxxxxxxxxxxx,ssl=True,abortConnect=False`

Si vous ne parvenez pas à résoudre le nom DNS, des bibliothèques clientes incluent des options de configuration comme `sslHost`, qui est fournie par le client StackExchange.Redis. Cela vous permet de remplacer le nom d’hôte utilisé pour la validation du certificat. Par exemple :

`10.128.2.84:6380,password=xxxxxxxxxxxxxxxxxxxx,ssl=True,abortConnect=False;sslHost=[mycachename].redis.windows.net`

### <a name="can-i-use-vnets-with-a-standard-or-basic-cache"></a>Puis-je utiliser des réseaux virtuels avec un cache De base ou Standard ?
Vous ne pouvez utiliser des réseaux virtuels qu’avec les caches de niveau Premium.

### <a name="why-does-creating-an-azure-cache-for-redis-fail-in-some-subnets-but-not-others"></a>Pourquoi la création d’un cache Azure pour Redis échoue-t-elle dans certains sous-réseaux mais pas d’autres ?
Si vous déployez une instance Azure Cache pour Redis sur un réseau virtuel, le cache doit se trouver dans un sous-réseau dédié qui ne contient aucun autre type de ressource. Si vous tentez de déployer une instance Azure Cache pour Redis sur un sous-réseau de réseau virtuel Resource Manager contenant d’autres ressources (comme des passerelles Application Gateway, des règles NAT de trafic sortant, etc.), le déploiement échoue en général. Vous devez supprimer les autres types de ressources existantes avant de pouvoir créer une nouvelle instance Azure Cache pour Redis.

Vous devez également disposer de suffisamment d’adresses IP disponibles dans le sous-réseau.

### <a name="what-are-the-subnet-address-space-requirements"></a>Quelles sont les exigences d’espace d’adressage du sous-réseau ?
Azure réserve dans chaque sous-réseau des adresses IP qui ne peuvent pas être utilisées. Les première et dernière adresse IP des sous-réseaux sont réservées à la conformité du protocole, et 3 adresses supplémentaires sont utilisées pour les services Azure. Pour plus d’informations, consultez [Existe-t-il des restrictions sur l’utilisation des adresses IP au sein de ces sous-réseaux ?](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)

Outre les adresses IP utilisées par l’infrastructure de réseau virtuel Azure, chaque instance Redis dans le sous-réseau utilise deux adresses IP par partition de cluster (plus des adresses IP supplémentaires pour les réplicas additionnels, le cas échéant) et une adresse IP supplémentaire pour l’équilibreur de charge. Un cache non cluster est considéré comme ayant une seule partition.

### <a name="do-all-cache-features-work-when-hosting-a-cache-in-a-vnet"></a>Toutes les fonctionnalités fonctionnent-elles lorsque vous hébergez un cache dans un réseau virtuel ?
Quand votre cache fait partie d’un réseau virtuel, seuls les clients de ce réseau virtuel peuvent accéder au cache. Par conséquent, les fonctionnalités de gestion de cache suivantes ne fonctionnent pas pour l’instant.

* Console Redis : étant donné que la console Redis s’exécute dans votre navigateur local, qui est généralement situé sur un ordinateur de développement qui n’est pas connecté au réseau virtuel, elle ne peut pas se connecter à votre cache.


## <a name="use-expressroute-with-azure-cache-for-redis"></a>Utiliser ExpressRoute avec le Cache Azure pour Redis

Les clients peuvent connecter un circuit [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/) à leur infrastructure de réseau virtuel pour étendre leur réseau local à Azure. 

Par défaut, un circuit ExpressRoute nouvellement créé n’effectue pas de tunneling forcé (publication d’un routage par défaut, 0.0.0.0/0) sur un réseau virtuel. Par conséquent, la connectivité Internet sortante est autorisée directement à partir du réseau virtuel, et les applications clientes sont en mesure de se connecter à d’autres points de terminaison Azure, dont le Cache Azure pour Redis.

Toutefois, une configuration de client courante consiste à utiliser un tunneling forcé (publication d’un routage par défaut), qui force le trafic Internet sortant à circuler localement à la place. Ce flux de trafic interrompt la connectivité avec le Cache Azure pour Redis si le trafic sortant est bloqué localement, par exemple, quand l’instance du Cache Azure pour Redis ne peut pas communiquer avec ses dépendances.

La solution consiste à définir un (ou plusieurs) itinéraires définis par l’utilisateur (UDR) sur le sous-réseau qui contient le Cache Azure pour Redis. Un itinéraire défini par l'utilisateur définit des itinéraires spécifiques au sous-réseau qui seront respectés au lieu de l'itinéraire par défaut.

Si possible, il est recommandé d'utiliser la configuration suivante :

* La configuration ExpressRoute annonce 0.0.0.0/0 et par défaut, tunnelise de force tout le trafic sortant sur site.
* L’UDR appliqué au sous-réseau contenant le Cache Azure pour Redis définit 0.0.0.0/0 avec un itinéraire de travail pour le trafic TCP/IP vers le réseau internet public, par exemple, en définissant le type de tronçon suivant vers « Internet ».

L’effet combiné de ces étapes est que l’UDR de niveau sous-réseau a la priorité sur le tunneling forcé ExpressRoute, garantissant ainsi un accès Internet sortant à partir du Cache Azure pour Redis.

La connexion à une instance du Cache Azure pour Redis à partir d’une application locale à l’aide d’ExpressRoute n’est pas un scénario d’utilisation classique pour des raisons de performances (pour des performances optimales, les clients du Cache Azure pour Redis doivent être situés dans la même région que le Cache Azure pour Redis).

>[!IMPORTANT] 
>Les itinéraires définis dans un UDR **doivent** être suffisamment spécifiques pour avoir la priorité sur les itinéraires annoncés par la configuration ExpressRoute. L'exemple suivant utilise la plage d'adresses 0.0.0.0/0 étendue qui peut potentiellement être remplacée accidentellement par les annonces de routage à l'aide de plages d'adresses plus spécifiques.

>[!WARNING]  
>Le cache Azure pour Redis n’est pas pris en charge avec les configurations ExpressRoute qui **publient incorrectement de façon croisée des itinéraires à partir du chemin d’accès de peering public vers le chemin d’accès de peering privé**. Les configurations ExpressRoute ayant un peering public configuré reçoivent des annonces de routage en provenance de Microsoft pour un grand ensemble de plages d’adresses IP Microsoft Azure. Si ces plages d’adresses sont incorrectement publiées de façon croisée sur le chemin d’accès de peering privé, il en résulte que tous les paquets réseau sortants du sous-réseau de l’instance du Cache Azure pour Redis sont incorrectement acheminés de force vers l’infrastructure réseau locale d’un client. Ce flux réseau interrompt le Cache Azure pour Redis. La solution à ce problème consiste à arrêter les itinéraires croisés depuis le chemin d'accès de peering public vers le chemin d'accès de peering privé.


Vous trouverez des informations générales sur les itinéraires définis par l'utilisateur dans cette [présentation](../virtual-network/virtual-networks-udr-overview.md).

Pour plus d’informations sur ExpressRoute, voir [Aperçu technique d’ExpressRoute](../expressroute/expressroute-introduction.md).

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les fonctionnalités d’Azure Cache pour Redis.

* [Niveaux de service Premium Azure Cache pour Redis](cache-overview.md#service-tiers)

<!-- IMAGES -->

[redis-cache-vnet]: ./media/cache-how-to-premium-vnet/redis-cache-vnet.png

[redis-cache-vnet-ip]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-ip.png

[redis-cache-vnet-info]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-info.png
