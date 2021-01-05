---
title: Vue d’ensemble des réseaux - Serveur flexible Azure Database pour MySQL
description: Découvrir les options de connectivité et de réseau de l’option de déploiement Serveur flexible pour Azure Database pour MySQL
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: conceptual
ms.date: 9/23/2020
ms.openlocfilehash: a8e2d77ff3c7cb2e4352b21cd87d630331e28660
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96906146"
---
# <a name="connectivity-and-networking-concepts-for-azure-database-for-mysql---flexible-server-preview"></a>Concepts de connectivité et de réseau pour Azure Database pour MySQL - Serveur flexible (préversion)

Cet article décrit les options de connectivité publique et privée disponibles pour votre serveur. Vous allez découvrir les détails des concepts de réseau d’un serveur flexible Azure Database pour MySQL afin de créer un serveur de façon sécurisée dans Azure.

> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est en préversion.

## <a name="choosing-a-networking-option"></a>Choix d’une option réseau
Vous avez deux possibilités de réseau pour votre serveur flexible Azure Database pour MySQL. Vous avez le choix entre l’**accès privé (intégration au réseau virtuel)** et l’**accès public (adresses IP autorisées)** . Lors de la création du serveur, vous devez choisir une seule option. 

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
* Les connexions à partir d’adresses IP autorisées ont besoin de s’authentifier auprès du serveur MySQL avec des informations d’identification valides.
* Le [chiffrement de la connexion](#tls-and-ssl) est disponible pour votre trafic réseau.
* Le serveur a un nom de domaine complet (FQDN). Pour la propriété du nom d’hôte dans les chaînes de connexion, nous vous recommandons d’utiliser le nom de domaine complet plutôt qu’une adresse IP.
* Les deux options contrôlent l’accès au niveau du serveur, pas au niveau de la base de données ni de la table. Vous pouvez utiliser les propriétés des rôles de MySQL pour contrôler l’accès à la base de données, à la table et à d’autres objets.


## <a name="private-access-vnet-integration"></a>Accès privé (intégration au réseau virtuel)
L’accès privé avec intégration au réseau virtuel offre des communications privées et sécurisées à votre serveur flexible MySQL.

### <a name="virtual-network-concepts"></a>Concepts de réseau virtuel
Voici quelques concepts à connaître quand vous utilisez des réseaux virtuels avec des serveurs flexibles MySQL.

* **Réseau virtuel** : Un réseau virtuel Azure contient un espace d’adressage IP privé configuré pour votre utilisation. Pour plus d’informations sur les réseaux virtuels Azure, consultez [Vue d’ensemble de Réseau virtuel Azure](../../virtual-network/virtual-networks-overview.md).

    Votre réseau virtuel doit se trouver dans la même région Azure que votre serveur flexible.


* **Sous-réseau délégué** : un réseau virtuel contient des sous-réseaux. Les sous-réseaux vous permettent de segmenter votre réseau virtuel en espaces d’adressage plus petits. Les ressources Azure sont déployées sur des sous-réseaux spécifiques au sein d’un réseau virtuel. 

   Votre serveur flexible MySQL doit se trouver dans un sous-réseau **délégué** à l’usage de serveur flexible MySQL uniquement. Cette délégation spécifie que seuls les serveurs flexibles Azure Database pour MySQL peuvent utiliser ce sous-réseau. Aucun autre type de ressource Azure ne peut se trouver dans le sous-réseau délégué. Pour déléguer un sous-réseau, affectez à sa propriété de délégation la valeur Microsoft.DBforMySQL/flexibleServers.

* **Groupes de sécurité réseau (NSG)**  : les règles de sécurité dans les groupes de sécurité réseau permettent de filtrer le type de trafic qui peut circuler vers et depuis les interfaces réseau et les sous-réseaux de réseau virtuel. Pour plus d’informations, consultez [Vue d’ensemble des groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md).


### <a name="unsupported-virtual-network-scenarios"></a>Scénarios de réseau virtuel non pris en charge
* Point de terminaison public (ou adresse IP publique ou DNS) : un serveur flexible déployé sur un réseau virtuel ne peut pas avoir de point de terminaison public.
* Une fois le serveur flexible déployé sur un réseau virtuel et un sous-réseau, vous ne pouvez pas le déplacer vers un autre réseau virtuel ou sous-réseau. Vous ne pouvez pas déplacer le réseau virtuel vers un autre groupe de ressources ou abonnement.
* Il est impossible d’augmenter la taille d’un sous-réseau (espaces d’adressage) une fois qu’il existe des ressources sur ce sous-réseau.
* L’appairage (peering) de réseaux virtuels entre régions n’est pas pris en charge.

Découvrez comment activer l’accès privé (intégration au réseau virtuel) à l’aide du [portail Azure](how-to-manage-virtual-network-portal.md) ou d’[Azure CLI](how-to-manage-virtual-network-cli.md).

> [!NOTE]
> Si vous utilisez le serveur DNS personnalisé, vous devez utiliser un redirecteur DNS pour résoudre le nom de domaine complet du serveur Azure Database pour MySQL – Serveur flexible. Consultez [Résolution de noms utilisant votre propre serveur DNS](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) pour en savoir plus.

## <a name="public-access-allowed-ip-addresses"></a>Accès public (adresses IP autorisées)
Les caractéristiques de la méthode d’accès public sont les suivantes :
* Seules les adresses IP que vous autorisez peuvent accéder à votre serveur flexible MySQL. Par défaut, aucune adresse IP n’est autorisée. Vous pouvez ajouter des adresses IP lors de la création du serveur ou par la suite.
* Votre serveur MySQL porte un nom DNS résolvable publiquement.
* Votre serveur flexible ne se trouve pas dans l’un de vos réseaux virtuels Azure.
* Le trafic réseau à destination et en provenance de votre serveur ne passe pas par un réseau privé. Le trafic utilise les chemins Internet généraux.

### <a name="firewall-rules"></a>Règles de pare-feu
L’octroi d’une autorisation à une adresse IP s’appelle une règle de pare-feu. Si une tentative de connexion provient d’une adresse IP que vous n’avez pas autorisée, le client d’origine voit une erreur.


### <a name="allowing-all-azure-ip-addresses"></a>Autorisation de toutes les adresses IP Azure
Si aucune adresse IP sortante fixe n’est disponible pour votre service Azure, vous pouvez envisager d’activer les connexions à partir de toutes les adresses IP du centre de données Azure.

> [!IMPORTANT]
> L’option **Autoriser l’accès public à partir des ressources et services Azure dans Azure** permet de configurer le pare-feu afin d’autoriser toutes les connexions en provenance d’Azure, notamment les connexions à partir des abonnements d’autres clients. Lorsque vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.

Découvrez comment activer et gérer l’accès public (adresses IP autorisées) à l’aide du [portail Azure](how-to-manage-firewall-portal.md) ou d’[Azure CLI](how-to-manage-firewall-cli.md).


### <a name="troubleshooting-public-access-issues"></a>Résolution des problèmes d’accès public
Considérez les points suivants quand l’accès au service de serveur Microsoft Azure Database pour MySQL présente un comportement anormal :

* **Les modifications apportées à la liste verte n’ont pas encore pris effet :** jusqu’à cinq minutes peuvent s’écouler avant que les changements apportés à la configuration du pare-feu du serveur Azure Database pour MySQL ne soient effectives.

* **Échec de l’authentification :** Si un utilisateur n’a pas d’autorisations sur le serveur Azure Database pour MySQL ou que le mot de passe est incorrect, la connexion au serveur Azure Database pour MySQL est refusée. La création d’un paramètre de pare-feu permet uniquement aux clients de tenter de se connecter à votre serveur. Chaque client doit quand même fournir les informations d’identification de sécurité nécessaires.

* **Adresse IP de client dynamique :** Si vous avez une connexion Internet avec un adressage IP dynamique et que le pare-feu demeure infranchissable, vous pouvez essayer une des solutions suivantes :

   * Demandez à votre fournisseur de services Internet (ISP) la plage d’adresses IP affectée à vos ordinateurs clients qui accèdent au serveur Azure Database pour MySQL, puis ajoutez cette plage à une règle de pare-feu.
   * Obtenez un adressage IP statique à la place pour vos ordinateurs clients, puis ajoutez l’adresse IP statique en tant que règle de pare-feu.
  
* **La règle de pare-feu n’est pas disponible pour le format IPv6 :** Les règles de pare-feu doivent être au format IPv4. Si vous spécifiez des règles de pare-feu au format IPv6, l’erreur de validation s’affiche.


## <a name="hostname"></a>Nom d’hôte
Quelle que soit l’option réseau choisie, nous vous recommandons de toujours utiliser un nom de domaine complet (FQDN) comme nom d’hôte lors de la connexion à votre serveur flexible. Il n’est pas garanti que l’adresse IP du serveur reste statique. L’utilisation du nom de domaine complet vous permet d’éviter d’apporter des modifications à votre chaîne de connexion. 

Exemple
* Recommandé : `hostname = servername.mysql.database.azure.com`
* Dans la mesure du possible, évitez d’utiliser `hostname = 10.0.0.4` (adresse privée) ou `hostname = 40.2.45.67` (IP publique)



## <a name="tls-and-ssl"></a>TLS et SSL
Le serveur flexible Azure Database pour MySQL prend en charge la connexion de vos applications clientes au service MySQL avec le protocole TLS (Transport Layer Security). Le protocole TLS est un standard du secteur qui garantit des connexions réseau chiffrées entre votre serveur de base de données et vos applications clientes. Il s’agit d’une mise à jour du protocole SSL (Secure Sockets Layer).

Le serveur flexible Azure Database pour MySQL prend uniquement en charge les connexions chiffrées avec le protocole TLS 1.2. Toutes les connexions entrantes qui utilisent les protocoles TLS 1.0 et TLS 1.1 sont refusées. Vous ne pouvez pas désactiver ni changer la version du protocole TLS utilisé pour la connexion au serveur flexible Azure Database pour MySQL.


## <a name="next-steps"></a>Étapes suivantes
* Découvrir comment activer l’accès privé (intégration au réseau virtuel) à l’aide du [portail Azure](how-to-manage-virtual-network-portal.md) ou d’[Azure CLI](how-to-manage-virtual-network-cli.md)
* Découvrir comment activer l’accès public (adresses IP autorisées) à l’aide du [portail Azure](how-to-manage-firewall-portal.md) ou d’[Azure CLI](how-to-manage-firewall-cli.md)
* Découvrir comment [utiliser le protocole TLS](how-to-connect-tls-ssl.md)
