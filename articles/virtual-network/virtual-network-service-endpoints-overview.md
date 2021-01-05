---
title: Points de terminaison de service de réseau virtuel Azure
titlesuffix: Azure Virtual Network
description: Découvrez comment activer l’accès direct aux ressources Azure à partir d’un réseau virtuel à l’aide de points de terminaison de service.
services: virtual-network
documentationcenter: na
author: sumeetmittal
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/08/2019
ms.author: sumi
ms.custom: ''
ms.openlocfilehash: 7d937542201792c0d1c0be69df9bd1c2b34edea3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004940"
---
# <a name="virtual-network-service-endpoints"></a>Points de terminaison de service de réseau virtuel

Le point de terminaison de service de réseau virtuel fournit une connexion sécurisée et directe aux services Azure sur un itinéraire optimisé du réseau principal Azure. Les points de terminaison permettent de sécuriser vos ressources critiques du service Azure pour vos réseaux virtuels uniquement. Les points de terminaison de service permettent aux adresses IP privées du réseau virtuel d'atteindre le point de terminaison d'un service Azure sans qu'une adresse IP publique soit nécessaire sur le réseau virtuel.

Cette fonctionnalité est disponible pour les services et régions Azure suivants. La ressource *Microsoft.\** est entre parenthèses. Activez cette ressource depuis le côté du sous-réseau lors de la configuration des points de terminaison de service pour votre service :

**Mise à la disposition générale**

- **[Stockage Azure](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network)** (*Microsoft.Storage*) : mis à la disposition générale dans toutes les régions Azure.
- **[Azure SQL Database](../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.Sql*) : mis à la disposition générale dans toutes les régions Azure.
- **[Azure Synapse Analytics](../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.Sql*) : mis à la disposition générale dans toutes les régions Azure.
- **[Serveur Azure Database pour PostgreSQL](../postgresql/howto-manage-vnet-using-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.Sql*) : mis à la disposition générale dans les régions Azure où le service de base de données est disponible.
- **[Serveur Azure Database pour MySQL](../mysql/howto-manage-vnet-using-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.Sql*) : mis à la disposition générale dans les régions Azure où le service de base de données est disponible.
- **[Azure Database for MariaDB](https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-vnet)** (*Microsoft.Sql*) : mis à la disposition générale dans les régions Azure où le service de base de données est disponible.
- **[Azure Cosmos DB](../cosmos-db/vnet-service-endpoint.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.AzureCosmosDB*) : mis à la disposition générale dans toutes les régions Azure.
- **[Azure Key Vault](../key-vault/general/overview-vnet-service-endpoints.md)** (*Microsoft.KeyVault*) : mis à la disposition générale dans toutes les régions Azure.
- **[Azure Service Bus](../service-bus-messaging/service-bus-service-endpoints.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.ServiceBus*) : mis à la disposition générale dans toutes les régions Azure.
- **[Azure Event Hubs](../event-hubs/event-hubs-service-endpoints.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.EventHub*) : mis à la disposition générale dans toutes les régions Azure.
- **[Azure Data Lake Store Gen 1](../data-lake-store/data-lake-store-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.AzureActiveDirectory*) : Disponibilité générale dans toutes les régions Azure où ADLS Gen1 est disponible.
- **[Azure App Service](https://docs.microsoft.com/azure/app-service/app-service-ip-restrictions)** (*Microsoft.Web*) : En disponibilité générale dans toutes les régions Azure où App Service est disponible.
- **[Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-virtual-networks?tabs=portal)** (*Microsoft.CognitiveServices*) : En disponibilité générale dans toutes les régions Azure où Cognitive Services est disponible.

**Préversion publique**

- **[Azure Container Registry](../container-registry/container-registry-vnet.md)** (*Microsoft.ContainerRegistry*) : préversion disponible dans certaines régions Azure où Azure Container Registry est disponible.

Pour obtenir les notifications les plus récentes, vérifiez la page [Mises à jour du réseau virtuel Microsoft Azure](https://azure.microsoft.com/updates/?product=virtual-network).

## <a name="key-benefits"></a>Principaux avantages

Les points de terminaison de service fournissent les avantages suivants :

- **Sécurité améliorée de vos ressources de service Azure** : Les espaces d’adressage privé de réseau virtuel peuvent se chevaucher. Vous ne pouvez pas utiliser d’espaces qui se chevauchent pour identifier de manière unique le trafic provenant de votre réseau virtuel. Les points de terminaison de service permettent de sécuriser les ressources de service Azure sur votre réseau virtuel en étendant l’identité du réseau virtuel à ce service. Une fois que vous avez activé les points de terminaison de service dans votre réseau virtuel, vous pouvez ajouter une règle de réseau virtuel afin de sécuriser les ressources du service Azure pour votre réseau virtuel. L’ajout de règles améliore la sécurité en supprimant totalement l’accès Internet public aux ressources et en autorisant le trafic uniquement à partir de votre réseau virtuel.
- **Routage optimal pour le trafic de service Azure à partir de votre réseau virtuel** : Aujourd’hui, tous les itinéraires dans votre réseau virtuel qui forcent le trafic Internet vers vos appliances locales et/ou virtuelles forcent également le trafic de service Azure à prendre le même itinéraire que le trafic Internet. Les points de terminaison de service fournissent un routage optimal pour le trafic Azure. 

  Les points de terminaison acheminent toujours le trafic de service directement à partir de votre réseau virtuel vers le service sur le réseau principal de Microsoft Azure. La conservation du trafic sur le réseau principal d’Azure vous permet de continuer l’audit et la surveillance du trafic Internet sortant à partir de vos réseaux virtuels, via le tunneling forcé, sans affecter le trafic de service. Pour plus d’informations sur les itinéraires définis par l’utilisateur et le tunneling forcé, consultez [Routage du trafic de réseau virtuel Azure](virtual-networks-udr-overview.md).
- **Une configuration simple et un temps de gestion réduit** : les adresses IP publiques réservées dans vos réseaux virtuels ne sont désormais plus nécessaires pour sécuriser les ressources Azure via le pare-feu IP. Aucune traduction d’adresses réseau (NAT) ni aucun appareil de passerelle n’est requis pour configurer les points de terminaison de service. Vous pouvez configurer des points de terminaison de service par un simple clic sur un sous-réseau. La conservation des points de terminaison ne requiert aucun temps système supplémentaire.

## <a name="limitations"></a>Limites

- La fonctionnalité est disponible uniquement pour les réseaux virtuels déployés à l’aide du modèle de déploiement Azure Resource Manager.
- Les points de terminaison sont activés sur les sous-réseaux configurés dans les réseaux virtuels Azure. Les points de terminaison ne peuvent pas être utilisés pour le trafic de votre réseau local aux services Azure. Pour plus d’informations, consultez [Sécuriser l’accès au service en local](#secure-azure-services-to-virtual-networks)
- Pour Azure SQL, un point de terminaison de service concerne uniquement le trafic de service Azure dans la région d’un réseau virtuel. Pour Stockage Azure, les points de terminaison s’étendent également de façon à inclure des régions jumelées dans lesquelles vous déployez le réseau virtuel pour prendre en charge le trafic du stockage géographiquement redondant avec accès en lecture (RA-GRS) et du stockage géoredondant (GRS). Pour plus d’informations, consultez [Régions jumelées Azure](../best-practices-availability-paired-regions.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-paired-regions).
- Avec Azure Data Lake Storage (ADLS) Gen 1, la fonctionnalité d’intégration au réseau virtuel est uniquement disponible pour les réseaux virtuels situés dans une même région. Notez également que l’intégration au réseau virtuel dans ADLS Gen1 utilise la sécurité des points de terminaison de service de réseau virtuel entre votre réseau virtuel et Azure Active Directory (Azure AD) afin de générer des revendications de sécurité supplémentaires dans le jeton d’accès. Ces revendications permettent ensuite d’authentifier votre réseau virtuel à votre compte Data Lake Storage Gen1 et d’y accéder. La balise *Microsoft.AzureActiveDirectory* répertoriée sous les services prenant en charge les points de terminaison de service est uniquement utilisée pour gérer ces derniers dans ADLS Gen 1. Azure AD ne prend pas en charge les points de terminaison de service en mode natif. Pour plus d’informations sur l’intégration au réseau virtuel d’Azure Data Lake Storage Gen1, consultez [Sécurité du réseau dans Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="secure-azure-services-to-virtual-networks"></a>Sécuriser les services Azure pour des réseaux virtuels

- Un point de terminaison de service de réseau virtuel fournit l’identité de votre réseau virtuel au service Azure. Une fois que vous avez activé les points de terminaison de service dans votre réseau virtuel, vous pouvez ajouter une règle de réseau virtuel afin de sécuriser les ressources du service Azure pour votre réseau virtuel.
- Aujourd’hui, le trafic du service Azure à partir d’un réseau virtuel utilise des adresses IP publiques en tant qu’adresses IP source. Avec les points de terminaison de service, le trafic de service change pour utiliser des adresses privées de réseau virtuel en tant qu’adresses IP source lors de l’accès au service Azure à partir d’un réseau virtuel. Ce changement permet d’accéder aux services sans avoir besoin d’adresses IP publiques réservées et utilisées dans les pare-feux IP.

   >[!NOTE]
   > Avec les points de terminaison de service, les adresses IP sources des machines virtuelles dans le sous-réseau pour le trafic de service passe d’une utilisation des adresses IPv4 publiques à une utilisation des adresses IPv4 privées. Les règles de pare-feu de service Azure existantes utilisant des adresses IP publiques Azure cesseront de fonctionner avec ce changement. Vérifiez que les règles de pare-feu de service Azure autorisent ce changement avant de définir des points de terminaison de service. Vous pouvez également rencontrer une interruption temporaire du trafic de service à partir de ce sous-réseau lors de la configuration des points de terminaison de service. 
 
## <a name="secure-azure-service-access-from-on-premises"></a>Sécuriser l’accès au service Azure en local

  Par défaut, les ressources du service Azure sécurisées pour des réseaux virtuels ne sont pas accessibles à partir des réseaux locaux. Si vous souhaitez autoriser le trafic depuis un réseau local, vous devez également autoriser les adresses IP publiques (généralement NAT) à partir de vos circuits locaux ou ExpressRoute. Vous pouvez ajouter ces adresses IP via la configuration du pare-feu IP des ressources du service Azure.

  ExpressRoute : Si vous utilisez [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour le Peering public ou Microsoft en local, vous devez identifier les adresses IP NAT utilisées. Pour le Peering public, chaque circuit ExpressRoute utilise par défaut deux adresses IP NAT qui sont appliquées au trafic du service Azure lorsque le trafic entre dans le réseau principal de Microsoft Azure. Pour le Peering Microsoft, les adresses IP NAT sont fournies par le client ou par le fournisseur du service.  Pour autoriser l’accès à vos ressources de votre service, vous devez autoriser ces adresses IP publiques dans le paramètre de pare-feu IP de ressource. Pour trouver les adresses IP de votre circuit ExpressRoute de peering public, [ouvrez un ticket de support avec ExpressRoute](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) via le portail Azure. Pour plus d’informations sur le Peering Microsoft et public NAT pour ExpressRoute, consultez [Configuration NAT requise pour ExpressRoute](../expressroute/expressroute-nat.md?toc=%2fazure%2fvirtual-network%2ftoc.json#nat-requirements-for-azure-public-peering).

![Sécurisation des services Azure pour des réseaux virtuels](./media/virtual-network-service-endpoints-overview/VNet_Service_Endpoints_Overview.png)

### <a name="configuration"></a>Configuration

- Configurez les points de terminaison de service sur un sous-réseau dans un réseau virtuel. Les points de terminaison fonctionnent avec n’importe quel type d’instances de calcul en cours d’exécution dans ce sous-réseau.
- Vous pouvez configurer plusieurs points de terminaison de service pour tous les services Azure pris en charge (par exemple, Stockage Azure ou Azure SQL Database) sur un sous-réseau.
- Pour Azure SQL Database, les réseaux virtuels doivent être dans la même région que la ressource de service Azure. Si vous utilisez les comptes de stockage Azure GRS et RA-GRS, le compte principal doit être dans la même région que le réseau virtuel. Pour tous les autres services, vous pouvez sécuriser les ressources du service Azure pour les réseaux virtuels de n’importe quelle région. 
- Le réseau virtuel dans lequel est configuré le point de terminaison peut être dans le même abonnement ou dans un abonnement différent de celui de la ressource du service Azure. Pour plus d’informations sur les autorisations requises pour la configuration de points de terminaison et la sécurisation des services Azure, consultez [Approvisionnement](#provisioning).
- Pour les services pris en charge, vous pouvez sécuriser des ressources nouvelles ou existantes pour des réseaux virtuels à l’aide de points de terminaison de service.

### <a name="considerations"></a>Considérations

- Après l’activation d’un point de terminaison de service, les adresses IP sources basculent de l’utilisation des adresses IPv4 publiques à l’utilisation de leur adresse IPv4 privée lors de la communication avec le service à partir de ce sous-réseau. Toute connexion TCP ouverte existante pour le service sera fermée lors de ce changement. Assurez-vous qu’aucune tâche critique n’est en cours d’exécution lors de l’activation ou de la désactivation d’un point de terminaison de service à un service pour un sous-réseau. En outre, assurez-vous que vos applications peuvent se connecter automatiquement aux services Azure après le changement d’adresse IP.

  Le changement d’adresse IP affecte uniquement le trafic de service à partir de votre réseau virtuel. L’impact sur tout autre trafic adressé vers ou depuis des adresses IPv4 publiques attribuées à vos machines virtuelles est inexistant. Si vous possédez des règles de pare-feu existantes à l’aide d’adresses IP publiques Azure pour les services Azure, ces règles cessent de fonctionner avec le changement pour les adresses privées de réseau virtuel.
- Avec les points de terminaison de service, les entrées DNS pour les services Azure ne sont pas modifiées et continuent de résoudre les adresses IP publiques assignées au service Azure.

- Groupes de sécurité réseau (NSG) avec points de terminaison de service :
  - Par défaut, les groupes de sécurité réseau autorisent le trafic Internet sortant, ainsi que le trafic de votre réseau virtuel vers les services Azure. Ce trafic continue de fonctionner avec les points de terminaison de service tels quels. 
  - Si vous souhaitez refuser tout trafic Internet sortant et autoriser le trafic uniquement vers des services Azure spécifiques, vous pouvez le faire à l’aide de [balises de service](security-overview.md#service-tags) dans vos groupes de sécurité réseau. Vous pouvez spécifier les services Azure pris en charge en tant que destination dans vos règles de groupe de sécurité réseau, et Azure fournit également la maintenance des adresses IP sous-tendant chaque balise. Pour plus d’informations, voir [Balises de Service Azure pour les groupes de sécurité réseau.](security-overview.md#service-tags) 

### <a name="scenarios"></a>Scénarios

- **Réseaux virtuels appariés, connectés ou multiples** : afin de sécuriser les services Azure pour plusieurs sous-réseaux au sein d’un réseau virtuel ou sur plusieurs réseaux virtuels, vous pouvez activer les points de terminaison de service sur chacun des sous-réseaux indépendamment et sécuriser les ressources de service Azure pour l’ensemble des sous-réseaux.
- **Filtrage du trafic sortant vers les services Azure à partir du réseau virtuel** : Si vous souhaitez inspecter ou filtrer le trafic envoyé à un service Azure à partir d’un réseau virtuel, vous pouvez déployer une appliance virtuelle réseau au sein du réseau virtuel. Vous pouvez ensuite appliquer des points de terminaison de service au sous-réseau sur lequel est déployée l’appliance virtuelle réseau et sécuriser les ressources de service Azure uniquement pour ce sous-réseau. Ce scénario peut être utile si vous souhaitez utiliser le filtrage de l’appliance virtuelle réseau pour restreindre l’accès aux services Azure à partir de votre réseau virtuel uniquement pour des ressources Azure spécifiques. Pour plus d’informations, consultez la section relative à la [sortie avec les appliances virtuelles réseau](/azure/architecture/reference-architectures/dmz/nva-ha).
- **Sécurisation des ressources Azure pour les services déployés directement dans les réseaux virtuels** : Vous pouvez déployer directement divers services Azure dans des sous-réseaux spécifiques au sein d’un réseau virtuel. Vous pouvez sécuriser les ressources du service Azure pour les sous-réseaux du [service administré](virtual-network-for-azure-services.md) en configurant un point de terminaison de service sur le sous-réseau de service administré.
- **Trafic de disques à partir d’une machine virtuelle Azure** : Le trafic des disques de machine virtuelle pour les disques managés et non managés n’est pas perturbé par la modification du routage des points de terminaison de service pour Stockage Azure. Ce trafic comprend l’E/S disque ainsi que le montage et le démontage. Vous pouvez limiter l’accès REST aux objets blob de page pour sélectionner les réseaux via des points de terminaison de service et les [règles de réseau de Stockage Azure](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json). 

### <a name="logging-and-troubleshooting"></a>Journalisation et résolution des problèmes

Une fois que vous avez configuré les points de terminaison de service pour un service spécifique, vérifiez que l’itinéraire de point de terminaison de service est activé en procédant comme suit : 
 
- Validation de l’adresse IP source de toute demande de service dans les diagnostics du service. Toutes les nouvelles demandes avec les points de terminaison de service affichent l’adresse IP source de la demande en tant qu’adresse IP privée du réseau virtuel, attribuée au client qui effectue la demande à partir de votre réseau virtuel. Sans le point de terminaison, cette adresse est une adresse IP publique Azure.
- Affichage des itinéraires effectifs sur n’importe quelle interface réseau d’un sous-réseau. L’itinéraire jusqu’au service :
  - affiche un itinéraire par défaut plus spécifique jusqu’aux plages de préfixes d’adresses de chaque service ;
  - possède un type de tronçon *VirtualNetworkServiceEndpoint* ;
  - indique qu’une connexion plus directe pour le service est appliquée, par rapport à n’importe quel tunneling forcé.

>[!NOTE]
> Les itinéraires de point de terminaison de service remplacent tout itinéraire BGP ou UDR pour la correspondance de préfixe d’adresse d’un service Azure. Pour plus d’informations, consultez [Résolution des problèmes à l’aide d’itinéraires effectifs](diagnose-network-routing-problem.md).

## <a name="provisioning"></a>Approvisionnement

Les points de terminaison de service peuvent être configurés indépendamment sur les réseaux virtuels par un utilisateur avec accès en écriture à un réseau virtuel. Pour sécuriser les ressources du service Azure pour un réseau virtuel, l’utilisateur doit disposer des autorisations sur *Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action* pour les sous-réseaux ajoutés. Les rôles Administrateur de service fédérés intégrés incluent cette autorisation par défaut. Vous pouvez modifier l’autorisation en créant des rôles personnalisés.

Pour plus d’informations sur les rôles intégrés, consultez [Rôles intégrés Azure](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Pour plus d’informations sur l’attribution d’autorisations spécifiques à des rôles personnalisés, consultez [Rôles Azure personnalisés](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Les réseaux virtuels et les ressources du service Azure peuvent être dans des abonnements identiques ou différents. Si le réseau virtuel et les ressources de service Azure se trouvent dans différents abonnements, les ressources doivent être sous le même client Active Directory (AD). 

## <a name="pricing-and-limits"></a>Tarification et limites

L’utilisation de points de terminaison de service n’engendre pas de frais supplémentaires. Le modèle tarifaire actuel pour les services Azure (Stockage Azure, Azure SQL Database, etc.) reste le même.

Il n’existe aucune limite sur le nombre total de points de terminaison de service dans un réseau virtuel.

Certains services Azure, comme Comptes Stockage Azure, peuvent appliquer des limites sur le nombre de sous-réseaux utilisés pour sécuriser la ressource. Pour en savoir plus, reportez-vous à la documentation pour les différents services dans la section [Étapes suivantes](#next-steps).

## <a name="vnet-service-endpoint-policies"></a>Stratégies de points de terminaison de service de réseau virtuel 

Les stratégies de point de terminaison de service de réseau virtuel vous permettent de filtrer le trafic de réseau virtuel vers les services Azure. Ce filtre autorise uniquement des ressources de service Azure spécifiques sur les points de terminaison de service. Les stratégies de points de terminaison de service fournissent un contrôle d’accès granulaire pour le trafic de réseau virtuel vers les services Azure. Pour plus d’informations, consultez [Stratégies de points de terminaison de service de réseau virtuel](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoint-policies-overview).

## <a name="faqs"></a>FAQ

Pour consulter les Forums aux questions, consultez les [FAQ sur les points de terminaison de service de réseau virtuel](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq#virtual-network-service-endpoints).

## <a name="next-steps"></a>Étapes suivantes

- [Configurer des points de terminaison de service de réseau virtuel](tutorial-restrict-network-access-to-resources.md)
- [Sécuriser un compte de stockage Azure pour un réseau virtuel](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Sécuriser une base de données Azure SQL Database pour un réseau virtuel](../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Sécuriser une instance Azure Synapse Analytics dans un réseau virtuel](../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fsql-data-warehouse%2ftoc.json)
- [Intégration des services Azure aux réseaux virtuels](virtual-network-for-azure-services.md)
- [Stratégies de points de terminaison de service de réseau virtuel](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoint-policies-overview)
- [Modèle Azure Resource Manager](https://azure.microsoft.com/resources/templates/201-vnet-2subnets-service-endpoints-storage-integration)
