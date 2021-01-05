---
title: Vue d’ensemble des réseaux - Azure Database pour PostgreSQL - Serveur flexible
description: Découvrez les options de connectivité et de réseau de l’option de déploiement Serveur flexible pour Azure Database pour PostgreSQL.
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: cf7c5fa8563544add55a7fad5075848eb5116fe1
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901930"
---
# <a name="networking-overview---azure-database-for-postgresql---flexible-server"></a>Vue d’ensemble des réseaux - Azure Database pour PostgreSQL - Serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en préversion

Cet article décrit les concepts de connectivité et de réseau pour Azure Database pour PostgreSQL - Serveur flexible. 

## <a name="choosing-a-networking-option"></a>Choix d’une option réseau
Vous disposez de deux options réseau pour votre serveur flexible Azure Database pour PostgreSQL. Il s’agit de l’**accès privé (intégration au réseau virtuel)** et de l’**accès public (adresses IP autorisées)** . Lors de la création du serveur, vous devez choisir une seule option. 

> [!NOTE]
> Vous ne pourrez pas changer votre option réseau une fois le serveur créé. 

* **Accès privé (intégration au réseau virtuel)**  : vous pouvez déployer votre serveur flexible sur votre [réseau virtuel Azure](../../virtual-network/virtual-networks-overview.md). Les réseaux virtuels Azure offrent des communications réseau privées et sécurisées. Les ressources incluses sur un réseau virtuel peuvent communiquer par le biais d’adresses IP privées.

   Choisissez l’option d’intégration au réseau virtuel si vous voulez avoir les capacités suivantes :
   * Connexion à partir de ressources Azure du même réseau virtuel à votre serveur flexible à l’aide d’adresses IP privées
   * Utilisation d’un VPN ou du service ExpressRoute pour vous connecter à partir de ressources non-Azure à votre serveur flexible
   * Le serveur flexible n’a pas de point de terminaison public.

* **Accès public (adresses IP autorisées)**  : votre serveur flexible est accessible par le biais d’un point de terminaison public. Le point de terminaison public est une adresse DNS résolvable publiquement. L’expression « adresses IP autorisées » fait référence à une plage d’adresses IP que vous choisissez d’autoriser à accéder à votre serveur. Ces autorisations sont appelées **règles de pare-feu**. 

   Choisissez la méthode d’accès public si vous voulez avoir les capacités suivantes :
   * Connexion à partir de ressources Azure qui ne prennent pas en charge les réseaux virtuels
   * Connexion à partir de ressources situées en dehors d’Azure qui ne sont pas connectées par un VPN ou ExpressRoute 
   * Le serveur flexible a un point de terminaison public.

Les caractéristiques suivantes s’appliquent si vous choisissez d’utiliser l’option d’accès privé ou d’accès public :
* Les connexions à partir d’adresses IP autorisées ont besoin de s’authentifier auprès du serveur PostgreSQL avec des informations d’identification valides.
* Le [chiffrement de la connexion](#tls-and-ssl) est disponible pour votre trafic réseau.
* Le serveur a un nom de domaine complet (FQDN). Pour la propriété du nom d’hôte dans les chaînes de connexion, nous vous recommandons d’utiliser le nom de domaine complet plutôt qu’une adresse IP.
* Les deux options contrôlent l’accès au niveau du serveur, pas au niveau de la base de données ni de la table. Vous pouvez utiliser les propriétés des rôles de PostgreSQL pour contrôler l’accès à la base de données, à la table et à d’autres objets.


## <a name="private-access-vnet-integration"></a>Accès privé (intégration au réseau virtuel)
L’accès privé avec intégration au réseau virtuel offre des communications privées et sécurisées à votre serveur flexible PostgreSQL.

### <a name="virtual-network-concepts"></a>Concepts de réseau virtuel
Voici quelques concepts à connaître quand vous utilisez des réseaux virtuels avec des serveurs flexibles PostgreSQL.

* **Réseau virtuel** : un réseau virtuel Azure contient un espace d’adressage IP privé configuré pour votre utilisation. Pour plus d’informations sur les réseaux virtuels Azure, consultez [Vue d’ensemble de Réseau virtuel Azure](../../virtual-network/virtual-networks-overview.md).

    Votre réseau virtuel doit se trouver dans la même région Azure que votre serveur flexible.


* **Sous-réseau délégué** : un réseau virtuel contient des sous-réseaux. Les sous-réseaux vous permettent de segmenter votre réseau virtuel en espaces d’adressage plus petits. Les ressources Azure sont déployées sur des sous-réseaux spécifiques au sein d’un réseau virtuel. 

   Votre serveur flexible PostgreSQL doit se trouver sur un sous-réseau **délégué** réservé à l’usage de serveur flexible PostgreSQL. Cette délégation spécifie que seuls les serveurs flexibles Azure Database pour PostgreSQL peuvent utiliser ce sous-réseau. Aucun autre type de ressource Azure ne peut se trouver sur le sous-réseau délégué. Pour déléguer un sous-réseau, affectez à sa propriété de délégation la valeur Microsoft.DBforPostgreSQL/flexibleServers.

* **Groupes de sécurité réseau (NSG)**  : les règles de sécurité dans les groupes de sécurité réseau permettent de filtrer le type de trafic qui peut circuler vers et depuis les interfaces réseau et les sous-réseaux de réseau virtuel. Pour plus d’informations, consultez [Vue d’ensemble des groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md).


### <a name="unsupported-virtual-network-scenarios"></a>Scénarios de réseau virtuel non pris en charge
* Point de terminaison public (ou adresse IP publique ou DNS) : un serveur flexible déployé sur un réseau virtuel ne peut pas avoir de point de terminaison public.
* Une fois le serveur flexible déployé sur un réseau virtuel et un sous-réseau, vous ne pouvez pas le déplacer vers un autre réseau virtuel ou sous-réseau. Vous ne pouvez pas déplacer le réseau virtuel vers un autre groupe de ressources ou abonnement.
* Il est impossible d’augmenter la taille d’un sous-réseau (espaces d’adressage) une fois qu’il existe des ressources sur ce sous-réseau.
* L’appairage (peering) de réseaux virtuels entre régions n’est pas pris en charge.

Découvrez comment créer un serveur flexible avec accès privé (intégration au réseau virtuel) dans le [portail Azure](how-to-manage-virtual-network-portal.md) ou [Azure CLI](how-to-manage-virtual-network-cli.md).

> [!NOTE]
> Si vous utilisez le serveur DNS personnalisé, vous devez utiliser un redirecteur DNS pour résoudre le nom de domaine complet du serveur Azure Database pour MySQL - Serveur flexible. Consultez [Résolution de noms utilisant votre propre serveur DNS](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) pour en savoir plus.

## <a name="public-access-allowed-ip-addresses"></a>Accès public (adresses IP autorisées)
Les caractéristiques de la méthode d’accès public sont les suivantes :
* Seules les adresses IP que vous autorisez peuvent accéder à votre serveur flexible PostgreSQL. Par défaut, aucune adresse IP n’est autorisée. Vous pouvez ajouter des adresses IP lors de la création du serveur ou par la suite.
* Votre serveur PostgreSQL a un nom DNS résolvable publiquement.
* Votre serveur flexible ne se trouve pas sur l’un de vos réseaux virtuels Azure.
* Le trafic réseau à destination et en provenance de votre serveur ne passe pas par un réseau privé. Le trafic utilise les chemins Internet généraux.

### <a name="firewall-rules"></a>Règles de pare-feu
L’octroi d’une autorisation à une adresse IP s’appelle une règle de pare-feu. Si une tentative de connexion provient d’une adresse IP que vous n’avez pas autorisée, le client d’origine voit une erreur.

Découvrez comment créer un serveur flexible avec accès public (adresses IP autorisées) dans le [portail Azure](how-to-manage-firewall-portal.md) ou [Azure CLI](how-to-manage-firewall-cli.md).


### <a name="allowing-all-azure-ip-addresses"></a>Autorisation de toutes les adresses IP Azure
Si aucune adresse IP sortante fixe n’est disponible pour votre service Azure, vous pouvez envisager d’activer les connexions à partir de toutes les adresses IP du centre de données Azure.

> [!IMPORTANT]
> L’option **Autoriser l’accès public à partir des ressources et services Azure dans Azure** permet de configurer le pare-feu afin d’autoriser toutes les connexions en provenance d’Azure, notamment les connexions à partir des abonnements d’autres clients. Lorsque vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.

### <a name="troubleshooting-public-access-issues"></a>Résolution des problèmes d’accès public
Considérez les points suivants quand l’accès au service de serveur de base de données Microsoft Azure pour PostgreSQL présente un comportement anormal :

* **Les modifications apportées à la liste verte n’ont pas encore pris effet :** Jusqu’à cinq minutes peuvent s’écouler avant que les changements apportés à la configuration du pare-feu du serveur Azure Database pour PostgreSQL ne soient effectives.

* **Échec de l’authentification :** Si un utilisateur n’a pas d’autorisation sur le serveur Azure Database pour PostgreSQL ou que le mot de passe est incorrect, la connexion au serveur Azure Database pour PostgreSQL est refusée. La création d’un paramètre de pare-feu permet uniquement aux clients de tenter de se connecter à votre serveur. Chaque client doit quand même fournir les informations d’identification de sécurité nécessaires.

* **Adresse IP de client dynamique :** Si vous avez une connexion Internet avec un adressage IP dynamique et que le pare-feu demeure infranchissable, vous pouvez essayer une des solutions suivantes :

   * Demandez à votre fournisseur de services Internet (ISP) la plage d’adresses IP affectée à vos ordinateurs clients qui accèdent au serveur de base de données Azure pour PostgreSQL, puis ajoutez cette plage dans une règle de pare-feu.
   * Obtenez un adressage IP statique à la place pour vos ordinateurs clients, puis ajoutez l’adresse IP statique en tant que règle de pare-feu.

* **La règle de pare-feu n’est pas disponible pour le format IPv6 :** Les règles de pare-feu doivent être au format IPv4. Si vous spécifiez des règles de pare-feu au format IPv6, l’erreur de validation s’affiche.

## <a name="hostname"></a>Nom d’hôte
Quelle que soit l’option réseau choisie, nous vous recommandons de toujours utiliser un nom de domaine complet (FQDN) comme nom d’hôte lors de la connexion à votre serveur flexible. Il n’est pas garanti que l’adresse IP du serveur reste statique. L’utilisation du nom de domaine complet vous permet d’éviter d’apporter des modifications à votre chaîne de connexion. 

Exemple
* Recommandé : `hostname = servername.postgres.database.azure.com`
* Dans la mesure du possible, évitez d’utiliser `hostname = 10.0.0.4` (adresse privée) ou `hostname = 40.2.45.67` (adresse publique)


## <a name="tls-and-ssl"></a>TLS et SSL
Azure Database pour PostgreSQL - Serveur flexible prend en charge la connexion de vos applications clientes au service PostgreSQL à l’aide du protocole TLS (Transport Layer Security). Le protocole TLS est une norme qui garantit le chiffrement des connexions réseau entre votre serveur de base de données et vos applications clientes. Il s’agit d’une mise à jour du protocole SSL (Secure Sockets Layer).

Azure Database pour PostgreSQL - Serveur flexible prend uniquement en charge les connexions chiffrées à l’aide du protocole TLS. Toutes les connexions entrantes qui utilisent les protocoles TLS 1.0 et TLS 1.1 sont refusées. 

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment créer un serveur flexible avec **accès privé (intégration au réseau virtuel)** dans le [portail Azure](how-to-manage-virtual-network-portal.md) ou [Azure CLI](how-to-manage-virtual-network-cli.md).
* Découvrez comment créer un serveur flexible avec **accès public (adresses IP autorisées)** dans le [portail Azure](how-to-manage-firewall-portal.md) ou [Azure CLI](how-to-manage-firewall-cli.md).
