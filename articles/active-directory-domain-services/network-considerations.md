---
title: Planification et connexions réseau pour Azure AD Domain Services | Microsoft Docs
description: Apprenez-en davantage sur certaines considérations et ressources relatives à la conception du réseau virtuel utilisées pour la connectivité lorsque vous exécutez Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 246da3a35396430bbda86e5a5e927a456618ac05
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619281"
---
# <a name="virtual-network-design-considerations-and-configuration-options-for-azure-active-directory-domain-services"></a>Considérations relatives à la conception du réseau virtuel et options de configuration pour Azure Active Directory Domain Services

Azure Active Directory Domain Services (AD DS) fournit des services d’authentification et de gestion à d’autres applications et charges de travail. La connectivité réseau est une composante clé. Sans ressources de réseau virtuel correctement configurées, les applications et les charges de travail ne peuvent pas communiquer avec les fonctionnalités fournies par Azure AD DS ni les utiliser. Planifiez la configuration requise de votre réseau virtuel pour vous assurer qu’Azure AD DS puisse servir vos applications et vos charges si nécessaire.

Cet article décrit les points et les conditions à prendre en compte lors de la conception pour qu’un réseau virtuel Azure prenne en charge Azure AD DS.

## <a name="azure-virtual-network-design"></a>Conception du réseau virtuel Azure

Pour assurer la connectivité réseau et autoriser les applications et les services à s’authentifier auprès d’un domaine managé Azure AD DS, vous devez utiliser un réseau virtuel et un sous-réseau Azure. Dans l’idéal, le domaine managé doit être déployé dans son propre réseau virtuel.

Vous pouvez inclure un sous-réseau d’application distinct dans le même réseau virtuel pour héberger votre machine virtuelle de gestion ou vos charges de travail pour les petites applications. Un réseau virtuel distinct pour les charges de travail d’application plus volumineuses ou complexes homologuées au réseau virtuel Azure AD DS, est généralement la conception la plus appropriée.

Si vous respectez les conditions décrites dans les sections suivantes pour le réseau virtuel et le sous-réseau, vous pouvez choisir d’autres conceptions.

Tout au long de votre conception du réseau virtuel pour Azure AD DS, les considérations suivantes s’appliquent :

* Azure AD DS doit être déployé dans la même région Azure que votre réseau virtuel.
    * À ce stade, vous ne pouvez déployer qu’un seul domaine managé par locataire Azure AD. Le domaine managé est déployé sur une seule région. Assurez-vous de créer ou de sélectionner un réseau virtuel dans [une région qui prend en charge Azure AD DS](https://azure.microsoft.com/global-infrastructure/services/?products=active-directory-ds&regions=all).
* Tenez compte de la proximité des autres régions Azure et des réseaux virtuels qui hébergent les charges de travail de votre application.
    * Pour réduire la latence, gardez vos applications principales à proximité du sous-réseau du réseau virtuel, à dans la même région que celui-ci, pour votre domaine managé. Vous pouvez utiliser le peering du réseau virtuel ou les connexions de réseau privé virtuel (VPN) entre les réseaux virtuels Azure. Ces options de connexion sont présentées dans une prochaine section.
* Le réseau virtuel ne peut pas s’appuyer sur des services DNS autres que ceux fournis par le domaine managé.
    * Azure AD DS fournit son propre service DNS. Le réseau virtuel doit être configuré pour utiliser ces adresses de service DNS. La résolution de noms pour des espaces de noms supplémentaires peut être accomplie à l’aide de redirecteurs conditionnels.
    * Vous ne pouvez pas utiliser les paramètres de serveur DNS personnalisés pour diriger des requêtes en provenance d’autres serveurs DNS, y compris sur des machines virtuelles. Les ressources du réseau virtuel doivent utiliser le service DNS fourni par le domaine managé.

> [!IMPORTANT]
> Vous ne pouvez pas déplacer Azure AD DS vers un autre réseau virtuel une fois le service activé.

Un domaine managé se connecte à un sous-réseau dans un réseau virtuel Azure. Concevez ce sous-réseau pour Azure AD DS en prenant en compte les considérations suivantes :

* Un domaine managé doit être déployé dans son propre sous-réseau. N’utilisez pas un sous-réseau existant ou un sous-réseau de passerelle.
* Un groupe de sécurité réseau est créé pendant le déploiement d’un domaine managé. Ce groupe de sécurité réseau contient les règles requises pour permettre une communication de service appropriée.
    * Ne créez et n’utilisez pas un groupe de sécurité réseau existant avec vos propres règles personnalisées.
* Un domaine managé requiert de 3 à 5 adresses IP. Assurez-vous que la plage d’adresses IP de votre sous-réseau peut fournir ce nombre d’adresses.
    * La restriction des adresses IP disponibles peut empêcher Ale domaine managé de gérer deux contrôleurs de domaine.

L’exemple de diagramme suivant présente une conception valide dans laquelle le domaine managé possède son propre sous-réseau. Le sous-réseau de passerelle pour la connectivité externe et les charges de travail d’application sont dans un sous-réseau connecté du réseau virtuel :

![Conception de sous-réseau recommandée](./media/active-directory-domain-services-design-guide/vnet-subnet-design.png)

## <a name="connections-to-the-azure-ad-ds-virtual-network"></a>Connexions au réseau virtuel Azure AD DS

Comme indiqué dans la section précédente, vous pouvez uniquement créer un domaine managé dans un réseau virtuel unique dans Azure et un seul domaine managé par locataire Azure AD. Si l’on se base sur cette architecture, vous devrez peut-être connecter un ou plusieurs réseaux virtuels qui hébergent les charges de travail de votre application sur votre réseau virtuel de domaine managé.

Vous pouvez connecter des charges de travail d’application hébergées sur d’autres réseaux virtuels Azure en utilisant l’une des méthodes suivantes :

* Peering de réseau virtuel
* Réseau privé virtuel (VPN)

### <a name="virtual-network-peering"></a>Peering de réseau virtuel

VNET Peering est un mécanisme permettant de connecter deux réseaux virtuels situés dans la même région via le réseau principal Azure. Vous pouvez connecter des réseaux virtuels dans différentes régions à l’aide du peering de réseaux virtuels mondiaux. Une fois homologués, les deux réseaux virtuels permettent aux ressources, telles que les machines virtuelles, de communiquer directement entre elles à l’aide d’adresses IP privées. Le peering de réseaux virtuels vous permet de déployer un domaine managé géré avec vos charges de travail d’applications déployées dans d’autres réseaux virtuels.

![Connexion entre des réseaux virtuels à l’aide d’un peering](./media/active-directory-domain-services-design-guide/vnet-peering.png)

Pour plus d’informations, consultez la page [Vue d’ensemble du peering du réseau virtuel Azure](../virtual-network/virtual-network-peering-overview.md).

### <a name="virtual-private-networking-vpn"></a>Réseau privé virtuel (VPN)

Vous pouvez connecter deux réseaux virtuels (connexion de réseau virtuel à réseau virtuel) de la même manière que vous pouvez configurer un réseau virtuel à un emplacement de site local. Ces deux connexions font appel à une passerelle VPN pour créer un tunnel sécurisé utilisant Ipsec/IKE. Ce modèle de connexion vous permet de déployer le domaine managé dans un réseau virtuel Azure, puis de connecter des emplacements locaux ou d’autres clouds.

![Connexion entre des réseaux virtuels à l’aide d’une passerelle VPN](./media/active-directory-domain-services-design-guide/vnet-connection-vpn-gateway.jpg)

Pour plus d’informations sur l’utilisation de réseaux privés virtuels, consultez la page [Configurer une connexion de passerelle VPN de réseau virtuel à réseau virtuel à l’aide du portail Azure](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md).

## <a name="name-resolution-when-connecting-virtual-networks"></a>Résolution de noms lors de la connexion de réseaux virtuels

Les réseaux virtuels connectés au réseau virtuel du domaine managé disposent généralement de leurs propres paramètres DNS. Lorsque vous connectez des réseaux virtuels, cela ne permet pas de configurer automatiquement la résolution de noms du réseau virtuel en cours de connectés afin de résoudre les services fournis par le domaine managé. La résolution de noms de réseaux virtuels en cours de connexion doit être configurée pour permettre aux charges de travail d’application de localiser le domaine managé.

Vous pouvez activer la résolution de noms à l’aide de redirecteurs DNS conditionnels sur le serveur DNS qui prend en charge les réseaux virtuels connectés, ou en utilisant les adresses IP DNS du réseau virtuel du domaine managé.

## <a name="network-resources-used-by-azure-ad-ds"></a>Ressources réseau utilisées par Azure AD DS

Un domaine managé crée des ressources réseau au cours du déploiement. Ces ressources sont nécessaires au bon fonctionnement et à la bonne gestion du domaine managé et ne doivent pas être configurées manuellement.

| Ressource Azure                          | Description |
|:----------------------------------------|:---|
| Cartes d'interface réseau                  | Azure AD DS héberge le domaine managé sur deux contrôleurs de domaine (DC) qui s’exécutent sur Windows Server en tant que machines virtuelles Azure. Chaque machine virtuelle dispose d’une interface réseau virtuelle qui se connecte à votre sous-réseau de réseau virtuel. |
| Adresse IP publique standard dynamique      | Azure AD DS communique avec le service de synchronisation et de gestion à l’aide d’une adresse IP publique de référence SKU standard. Pour plus d’informations sur les adresse IP publique, consultez la page [Types d’adresses IP et méthodes d’allocation dans Azure](../virtual-network/public-ip-addresses.md). |
| Azure Standard Load Balancer            | Azure AD DS utilise un équilibreur de charge SKU standard pour la traduction d’adresses réseau (NAT) et l’équilibrage de charge (en cas d’utilisation avec le protocole LDAP sécurisé). Pour plus d’informations sur les équilibreurs de charge Azure, consultez [Qu’est-ce que Azure Load Balancer ?](../load-balancer/load-balancer-overview.md) |
| Règles de traduction d’adresses réseau (NAT) | Azure AD DS crée et utilise trois règles NAT sur l’équilibreur de charge : une règle pour le trafic HTTP sécurisé et deux règles pour une communication à distance PowerShell sécurisé. |
| Règles d'équilibrage de charge                     | Quand un domaine managé est configuré pour le LDAP sécurisé sur le port TCP 636, trois règles sont créées et utilisées sur un équilibreur de charge pour répartir le trafic. |

> [!WARNING]
> Ne supprimez ni ne modifiez aucune des ressources réseau créées par Azure AD DS, telles que la configuration manuelle de l’équilibreur de charge ou des règles. Si vous supprimez ou modifiez l’une des ressources réseau, le service Azure AD DS peut être interrompu.

## <a name="network-security-groups-and-required-ports"></a>Groupes de sécurité réseau et ports requis

Un [groupe de sécurité réseau (NSG)](../virtual-network/network-security-groups-overview.md) contient la liste des règles qui autorisent ou rejettent le trafic réseau vers le trafic d’un réseau virtuel Azure. Un groupe de sécurité réseau est créé lorsque vous déployez un domaine managé contenant un ensemble de règles permettant au service de fournir des fonctions d’authentification et de gestion. Ce groupe de sécurité réseau par défaut est associé au sous-réseau de réseau virtuel dans lequel votre domaine managé est déployé.

Les règles de groupe de sécurité réseau suivantes sont requises pour permettre au domaine managé de fournir des services d’authentification et de gestion. Ne modifiez pas et ne supprimez pas ces règles de groupe de sécurité réseau pour le sous-réseau de réseau virtuel dans lequel votre domaine managé est déployé.

| Numéro de port | Protocol | Source                             | Destination | Action | Obligatoire | Objectif |
|:-----------:|:--------:|:----------------------------------:|:-----------:|:------:|:--------:|:--------|
| 443         | TCP      | AzureActiveDirectoryDomainServices | Quelconque         | Allow  | Oui      | Synchronisation avec votre locataire Azure AD. |
| 3389        | TCP      | CorpNetSaw                         | Quelconque         | Allow  | Oui      | Gestion de votre domaine. |
| 5986        | TCP      | AzureActiveDirectoryDomainServices | Quelconque         | Allow  | Oui      | Gestion de votre domaine. |

Un équilibreur Azure Standard Load Balancer, pour lequel ces règles doivent être en place, est créé. Ce groupe de sécurité réseau, qui sécurise Azure AD DS, est nécessaire pour que le domaine managé fonctionne correctement. Ne supprimez pas ce groupe de sécurité réseau. L’équilibreur de charge ne fonctionnera pas correctement sans ce groupe.

Si nécessaire, vous pouvez [créer le groupe de sécurité réseau et les règles requis à l’aide d’Azure PowerShell](powershell-create-instance.md#create-a-network-security-group).

> [!WARNING]
> Ne modifiez pas manuellement ces configurations et ressources réseau. Lorsque vous associez un groupe de sécurité réseau mal configuré ou une table d’itinéraire définie par l’utilisateur au sous-réseau dans lequel le domaine managé est déployé, vous risquez de perturber la capacité de Microsoft à traiter et à gérer le domaine. La synchronisation entre votre locataire Azure AD et votre domaine managé est également interrompue.
>
> Si vous utilisez le protocole LDAP sécurisé, vous pouvez ajouter la règle de port TCP 636 requise pour autoriser le trafic externe, si nécessaire. L’ajout de cette règle ne place pas vos règles de groupe de sécurité réseau dans un état non pris en charge. Pour plus d’informations, consultez [Verrouiller l’accès LDAP sécurisé via Internet](tutorial-configure-ldaps.md#lock-down-secure-ldap-access-over-the-internet).
>
> Règles par défaut pour *AllowVnetInBound*, *AllowAzureLoadBalancerInBound*, *DenyAllInBound*, *AllowVnetOutBound*, *AllowInternetOutBound* et *DenyAllOutBound* existent également pour le groupe de sécurité réseau. Ne modifiez pas et ne supprimez pas ces règles par défaut.
>
> Le SLA Azure ne s’applique pas aux déploiements dans lesquels un groupe de sécurité réseau mal configuré et/ou des tables d’itinéraire définies par l’utilisateur ont été appliqués et empêchent Azure AD DS de mettre à jour et de gérer votre domaine.

### <a name="port-443---synchronization-with-azure-ad"></a>Port 443 – synchronisation avec Azure AD

* Utilisé pour synchroniser votre locataire Azure AD avec votre domaine managé.
* Sans accès à ce port, votre domaine managé ne peut pas être synchronisé avec votre locataire Azure AD. Les utilisateurs risquent de ne pas pouvoir se connecter, car les modifications apportées à leurs mots de passe ne sont pas synchronisées avec votre domaine managé.
* L’accès entrant à ce port aux adresses IP est limité par défaut à l'aide de la balise de service **AzureActiveDirectoryDomainServices**.
* Ne limitez pas l’accès sortant à partir de ce port.

### <a name="port-3389---management-using-remote-desktop"></a>Port 3389 – gestion à l’aide du bureau à distance

* Utilisé pour les connexions Bureau à distance aux contrôleurs de domaine de votre domaine managé.
* La règle de groupe de sécurité réseau par défaut utilise la balise de service *CorpNetSaw* pour restreindre davantage le trafic.
    * Cette balise de service autorise uniquement les stations de travail à accès sécurisé sur le réseau d’entreprise Microsoft à utiliser le bureau à distance pour le domaine managé.
    * L’accès est autorisé uniquement avec la justification métier, par exemple pour les scénarios de gestion ou de résolution des problèmes.
* Cette règle peut être définie sur *Refuser* et définie uniquement sur *Autoriser* quand cela est nécessaire. La plupart des tâches de gestion et de surveillance sont effectuées à l’aide de la communication à distance PowerShell. Le RDP n'est utilisé que dans les rares cas où Microsoft a besoin de se connecter à distance à votre domaine managé pour une résolution des problèmes avancée.

> [!NOTE]
> Vous ne pouvez pas sélectionner manuellement la balise de service *CorpNetSaw* dans le portail si vous essayez de modifier cette règle de groupe de sécurité réseau. Vous devez utiliser Azure PowerShell ou Azure CLI pour configurer manuellement une règle qui utilise la balise de service *CorpNetSaw*.
>
> Par exemple, vous pouvez utiliser le script suivant pour créer une règle autorisant le protocole RDP : 
>
> `Get-AzureRmNetworkSecurityGroup -Name "nsg-name" -ResourceGroupName "resource-group-name" | Add-AzureRmNetworkSecurityRuleConfig -Name "new-rule-name" -Access "Allow" -Protocol "TCP" -Direction "Inbound" -Priority "priority-number" -SourceAddressPrefix "CorpNetSaw" -SourcePortRange "" -DestinationPortRange "3389" -DestinationAddressPrefix "" | Set-AzureRmNetworkSecurityGroup`

### <a name="port-5986---management-using-powershell-remoting"></a>Port 5986 – -gestion à l’aide de la communication à distance PowerShell

* Utilisé pour effectuer des tâches de gestion à l’aide de la communication à distance PowerShell sur votre domaine managé.
* Sans accès à ce port, votre domaine managé ne peut pas être mis à jour, configuré, sauvegardé ou surveillé.
* Pour les domaines managés qui utilisent un réseau virtuel basé sur Resource Manager, vous pouvez restreindre l’accès entrant à ce port à la balise de service *AzureActiveDirectoryDomainServices*.
    * Pour les domaines managés hérités utilisant un réseau virtuel classique, vous pouvez restreindre l’accès entrant à ce port aux adresses IP sources suivantes : *52.180.183.8*, *23.101.0.70*, *52.225.184.198*, *52.179.126.223*, *13.74.249.156*, *52.187.117.83*, *52.161.13.95*, *104.40.156.18*, et *104.40.87.209*.

    > [!NOTE]
    > En 2017, le service Azure AD Domain Services est devenu disponible pour l’hébergement dans un réseau Azure Resource Manager. Depuis lors, nous avons pu créer un service plus sécurisé utilisant les fonctionnalités modernes d’Azure Resource Manager. Étant donné que les déploiements d’Azure Resource Manager remplacent entièrement les déploiements d’Azure Classic, les déploiements de réseau virtuel Classic d’Azure AD DS seront mis hors service le 1er mars 2023.
    >
    > Pour plus d’informations, voir l’[avis de désapprobation officiel](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/)

## <a name="user-defined-routes"></a>Itinéraires définis par l’utilisateur

Les itinéraires définis par l’utilisateur ne sont pas créés par défaut et ne sont pas nécessaires au bon fonctionnement d’Azure AD DS. Si vous devez utiliser des tables d’itinéraire, évitez de modifier l’itinéraire *0.0.0.0*. Les modifications apportées à cet itinéraire perturbent Azure AD Domain Services et place le domaine managé dans un état non pris en charge.

Vous devez également acheminer le trafic entrant à partir des adresses IP incluses dans les balises de service Azure respectives vers le sous-réseau du domaine managé. Pour plus d’informations sur les balises de service et leur adresse IP associée , consultez la page [Plages et balises de service Azure IP – Cloud public](https://www.microsoft.com/en-us/download/details.aspx?id=56519).

> [!CAUTION]
> Ces plages IP de centre de données Azure peuvent être modifiées sans préavis. Vérifiez que vous disposez des processus vous permettant de valider que vous disposez des dernières adresses IP.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur certaines ressources réseau et options de connexion utilisées par Azure AD DS, consultez les articles suivants :

* [Peering de réseaux virtuels Azure](../virtual-network/virtual-network-peering-overview.md)
* [Passerelles VPN Azure](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md)
* [Groupes de sécurité réseau Azure](../virtual-network/network-security-groups-overview.md)