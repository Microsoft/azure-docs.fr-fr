---
title: Private Link - Azure Database pour MySQL
description: Découvrez le fonctionnement d’Azure Private Link avec Azure Database pour MySQL (préversion).
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: 1ae35d1ac35dacfab2690980d57973dce050382b
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93242855"
---
# <a name="private-link-for-azure-database-for-mysql"></a>Private Link pour Azure Database pour MySQL

Private Link vous permet de vous connecter à différents services PaaS dans Azure par le biais d’un point de terminaison privé. Azure Private Link intègre essentiellement les services Azure à votre Réseau virtuel privé. Vous pouvez accéder aux ressources PaaS à l’aide de l’adresse IP privée, comme toute autre ressource dans le réseau virtuel.

Pour obtenir la liste des services PaaS prenant en charge la fonctionnalité Private Link, consultez la [documentation](../private-link/index.yml) de Private Link. Un point de terminaison privé est une adresse IP privée au sein d’un [réseau virtuel](../virtual-network/virtual-networks-overview.md) et d’un sous-réseau spécifiques.

> [!NOTE]
> La fonctionnalité de lien privé est disponible uniquement pour les serveurs Azure Database pour MySQL dans les niveaux tarifaires Usage général ou Mémoire optimisée. Vérifiez que le serveur de base de données se trouve dans l’un de ces niveaux tarifaires.

## <a name="data-exfiltration-prevention"></a>Prévention de l’exfiltration de données

L’exfiltration de données dans Azure Database pour MySQL se produit quand un utilisateur autorisé, tel qu’un administrateur de base de données, est en mesure d’extraire des données d’un système et de les déplacer vers un autre emplacement ou système en dehors de l’organisation. C’est par exemple le cas quand un utilisateur déplace des données vers un compte de stockage détenu par un tiers.

Imaginez un scénario avec un utilisateur exécutant MySQL Workbench à l’intérieur d’une machine virtuelle Azure qui se connecte à un serveur Azure Database pour MySQL approvisionné dans la région USA Ouest. L’exemple ci-dessous montre comment utiliser des contrôles d’accès réseau pour limiter l’accès à Azure Database pour MySQL par le biais de points de terminaison publics.

* Désactivez tout le trafic des services Azure à destination d’Azure Database pour MySQL par le biais du point de terminaison public en désactivant l’option *Autoriser les services Azure*. Assurez-vous qu’aucune adresse IP ou plage n’est autorisée à accéder au serveur via des [règles de pare-feu](./concepts-firewall-rules.md) ou des [points de terminaison de service de réseau virtuel](./concepts-data-access-and-security-vnet.md).

* Autorisez uniquement le trafic à destination d’Azure Database pour MySQL utilisant l’adresse IP privée de la machine virtuelle. Pour plus d’informations, consultez les articles sur le [point de terminaison de service](concepts-data-access-and-security-vnet.md) et les [règle de pare-feu du réseau virtuel](howto-manage-vnet-using-portal.md).

* Sur la machine virtuelle Azure, limitez l’étendue de la connexion sortante à l’aide de groupes de sécurité réseau (NSG) et d’étiquettes de service comme suit.

    * Spécifiez une règle NSG pour autoriser le trafic pour *Service Tag = SQL.WestUs* (autorise uniquement la connexion à Azure Database pour MySQL dans la région USA Ouest)
    * Spécifiez une règle NSG (avec une priorité plus élevée) pour refuser le trafic pour *Service Tag = SQL* (refuse les connexions à Mettre à jour vers Azure Database pour MySQL dans toutes les régions)</br></br>

À la fin de cette configuration, la machine virtuelle Azure peut uniquement se connecter à Azure Database pour MySQL dans la région USA Ouest. Toutefois, la connectivité n’est pas limitée à une seule instance Azure Database pour MySQL. La machine virtuelle peut toujours se connecter à toute instance Azure Database pour MySQL de la région USA Ouest, même si elle ne fait pas partie de l’abonnement. Bien que nous ayons réduit l’étendue de l’exfiltration de données dans le scénario ci-dessus à une région spécifique, nous ne l’avons pas encore totalement éliminée.</br>

Grâce à Private Link, vous pouvez désormais configurer des contrôles d’accès réseau comme des groupes de sécurité réseau pour restreindre l’accès au point de terminaison privé. Les ressources Azure PaaS individuelles sont ensuite mappées à des points de terminaison privés spécifiques. Un utilisateur interne malveillant peut uniquement accéder à la ressource PaaS mappée (par exemple, une instance Azure Database pour MySQL). Il n’a accès à aucune autre ressource.

## <a name="on-premises-connectivity-over-private-peering"></a>Connectivité locale sur un appairage privé

Lorsque vous vous connectez au point de terminaison public à partir de machines locales, votre adresse IP doit être ajoutée au pare-feu IP à l’aide d’une règle de pare-feu au niveau du serveur. Bien que ce modèle fonctionne bien pour autoriser l’accès à des machines individuelles pour des charges de travail de développement ou de test, il est difficile à gérer dans un environnement de production.

Grâce à Private Link, vous pouvez activer l’accès entre différents locaux au point de terminaison privé en utilisant [ExpressRoute](https://azure.microsoft.com/services/expressroute/) (ER), un peering privé ou un [tunnel VPN](../vpn-gateway/index.yml). Vous pouvez ensuite désactiver tous les accès via le point de terminaison public et ne pas utiliser le pare-feu IP.

> [!NOTE]
> Il peut arriver que l’instance Azure Database pour MySQL et le sous-réseau de réseau virtuel se trouvent dans des abonnements différents. Dans ce cas, vous devez vérifier les configurations suivantes :
> - Assurez-vous que le fournisseur de ressources **Microsoft.DBforMySQL** est inscrit pour les deux abonnements. Pour plus d’informations, reportez-vous à [resource-manager-registration][resource-manager-portal]

## <a name="configure-private-link-for-azure-database-for-mysql"></a>Configurer Azure Private Link pour Azure Database pour MySQL

### <a name="creation-process"></a>Processus de création

Des points de terminaison privés sont requis pour activer Private Link. Pour ce faire, vous pouvez utiliser les guides pratiques suivants.

* [Azure portal](./howto-configure-privatelink-portal.md)
* [INTERFACE DE LIGNE DE COMMANDE](./howto-configure-privatelink-cli.md)

### <a name="approval-process"></a>Processus d’approbation
Une fois que l’administrateur réseau a créé le point de terminaison privé (PE), l’administrateur MySQL peut gérer la connexion de point de terminaison privé (PEC) à Azure Database pour MySQL. Cette séparation des tâches entre l’administrateur réseau et l’administrateur de bases de données facilite la gestion de la connectivité Azure Database pour MySQL. 

* Accédez à la ressource de serveur Azure Database pour MySQL dans le Portail Azure. 
    * Sélectionner les connexions de point de terminaison privé dans le volet gauche
    * Affiche la liste de toutes les connexions de point de terminaison privé (PEC)
    * Point de terminaison privé (PE) correspondant créé

:::image type="content" source="media/concepts-data-access-and-security-private-link/select-private-link-portal.png" alt-text="sélectionnez le portail du point de terminaison privé":::

* Sélectionnez un PEC dans la liste.

:::image type="content" source="media/concepts-data-access-and-security-private-link/select-private-link.png" alt-text="sélectionnez l’approbation en attente du point de terminaison privé":::

* L’administrateur du serveur MySQL peut choisir d’approuver ou de rejeter un PEC. Il peut aussi ajouter une brève réponse sous forme de texte.

:::image type="content" source="media/concepts-data-access-and-security-private-link/select-private-link-message.png" alt-text="sélectionnez le message du point de terminaison privé":::

* Après l’approbation ou le rejet, la liste reflète l’état approprié et le texte de réponse

:::image type="content" source="media/concepts-data-access-and-security-private-link/show-private-link-approved-connection.png" alt-text="sélectionnez l’état final du point de terminaison privé":::

## <a name="use-cases-of-private-link-for-azure-database-for-mysql"></a>Cas d’usage d’Azure Private Link pour Azure Database pour MySQL

Les clients peuvent se connecter au point de terminaison privé à partir du même réseau virtuel, d’un réseau virtuel appairé dans la même région ou par le biais d’une connexion réseau virtuel à réseau virtuel entre régions. Les clients peuvent également se connecter localement avec ExpressRoute, un appairage privé ou un tunneling VPN. Vous trouverez ci-dessous un diagramme simplifié montrant les cas d’usage courants.

:::image type="content" source="media/concepts-data-access-and-security-private-link/show-private-link-overview.png" alt-text="sélectionnez la vue d’ensemble du point de terminaison privé":::

### <a name="connecting-from-an-azure-vm-in-peered-virtual-network-vnet"></a>Connexion à partir d’une machine virtuelle Azure dans un réseau virtuel appairé
Configurez le [Peering de réseau virtuel](../virtual-network/tutorial-connect-virtual-networks-powershell.md) pour établir la connectivité à Azure Database pour MySQL à partir d’une machine virtuelle Azure dans un réseau virtuel appairé.

### <a name="connecting-from-an-azure-vm-in-vnet-to-vnet-environment"></a>Connexion à partir d’une machine virtuelle Azure dans un environnement de réseau virtuel à réseau virtuel
Configurez une [connexion de passerelle VPN de réseau virtuel à réseau virtuel](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) pour établir la connectivité à une instance Azure Database pour MySQL à partir d’une machine virtuelle Azure dans une autre région ou un autre abonnement.

### <a name="connecting-from-an-on-premises-environment-over-vpn"></a>Connexion à partir d’un environnement local sur un VPN
Pour établir la connectivité entre un environnement local et Azure Database pour MySQL, choisissez et implémentez l’une des options suivantes :

* [Connexion point à site](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)
* [Connexion VPN de site à site](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)
* [Circuit ExpressRoute](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)

## <a name="private-link-combined-with-firewall-rules"></a>Private Link combiné à des règles de pare-feu

Les situations et résultats suivants sont possibles lorsque vous utilisez Private Link en association avec des règles de pare-feu :

* Si vous ne configurez pas de règles de pare-feu, par défaut, aucun trafic ne pourra accéder à Azure Database pour MySQL.

* Si vous configurez un trafic public ou un point de terminaison de service et que vous créez des points de terminaison privés, différents types de trafic entrant sont alors autorisés par le type de règle de pare-feu correspondant.

* Si vous ne configurez aucun trafic public ni point de terminaison de service et que vous créez des points de terminaison privés, Azure Database pour MySQL est alors uniquement accessible via les points de terminaison privés. Si vous ne configurez aucun trafic public ni point de terminaison de service, après le rejet ou la suppression de tous les points de terminaison privés approuvés, aucun trafic ne sera en mesure d’accéder à Azure Database pour MySQL.

## <a name="deny-public-access-for-azure-database-for-mysql"></a>Refuser l’accès public pour Azure Database pour MySQL

Si vous souhaitez uniquement vous fier aux points de terminaison privés pour accéder à leur base de données Azure Database pour MySQL, vous pouvez désactiver la définition de tous les points de terminaison publics (à savoir, [règles de pare-feu](concepts-firewall-rules.md) et [points de terminaison de service de réseau virtuel](concepts-data-access-and-security-vnet.md)) en définissant la configuration **Refuser l’accès au réseau public** sur le serveur de base de données. 

Lorsque ce paramètre est défini sur *OUI* , seules les connexions via des points de terminaison privés sont autorisées vers votre base de données Azure Database pour MySQL. Lorsque ce paramètre est défini sur *NON* , les clients peuvent se connecter à votre base de données Azure Database pour MySQL en fonction des paramètres des points de terminaison de votre service de pare-feu ou de réseau virtuel. En outre, une fois la valeur d'accès au réseau privé définie, les clients ne peuvent ajouter et/ou mettre à jour ni des « règles de pare-feu » existantes, ni des « règles de points de terminaison du service de réseau virtuel ».

> [!Note]
> Cette fonctionnalité est disponible dans toutes les régions Azure où Azure Database pour PostgreSQL - Serveur unique prend en charge les niveaux tarifaires Usage général et Mémoire optimisée.
>
> Ce paramètre n’a aucun impact sur les configurations SSL et TLS pour votre base de données Azure Database pour MySQL.

Pour savoir comment définir l’option **Refuser l’accès au réseau public** pour votre base de données Azure Database pour MySQL à partir du portail Azure, consultez [Configurer Refuser l’accès au réseau public](howto-deny-public-network-access.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les fonctionnalités de sécurité d’Azure Database pour MySQL, consultez les articles suivants :

* Pour configurer un pare-feu pour Azure Database pour MySQL, consultez [Prise en charge du pare-feu](./concepts-firewall-rules.md).

* Pour découvrir comment configurer un point de terminaison de service de réseau virtuel pour Azure Database pour MySQL, consultez [Configurer l’accès à partir de réseaux virtuels](./concepts-data-access-and-security-vnet.md).

* Pour obtenir une vue d’ensemble de la connectivité Azure Database pour MySQL, consultez [Architecture de connectivité Azure Database pour MySQL](./concepts-connectivity-architecture.md).

<!-- Link references, to text, Within this same GitHub repo. -->
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md