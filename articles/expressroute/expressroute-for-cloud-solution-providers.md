---
title: ExpressRoute pour les fournisseurs de solutions cloud - Azure | Microsoft Docs
description: Cet article fournit des informations pour les fournisseurs de services cloud qui souhaitent intégrer dans leurs offres ExpressRoute et des services Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 10/10/2016
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 09fee610ccc15874481ecfd4693e4b89379caa7a
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92330034"
---
# <a name="expressroute-for-cloud-solution-providers-csp"></a>ExpressRoute pour les fournisseurs de solutions Cloud (CSP)
Microsoft fournit des services à très grande échelle afin que les revendeurs et les distributeurs traditionnels puissent configurer rapidement de nouveaux services et solutions pour vos clients sans avoir à investir dans le développement de ces nouveaux services. Pour permettre au fournisseur de solutions Cloud (CSP) de gérer directement ces nouveaux services, Microsoft fournit des programmes et des API lui permettant de gérer les ressources Microsoft Azure pour le compte de vos clients. L’une de ces ressources est ExpressRoute. ExpressRoute permet au fournisseur de solutions Cloud de connecter les ressources client existantes aux services Azure. ExpressRoute est une liaison de communication privée haut débit vers les services d’Azure. 

ExpressRoute se compose d’une paire de circuits assurant la haute disponibilité qui sont associés à un ou des abonnements client uniques et ne peuvent pas être partagés par plusieurs clients. Chaque circuit doit se terminer dans un autre routeur afin de maintenir la haute disponibilité.

> [!NOTE]
> La bande passante et le nombre de connexions possibles sur chaque circuit ExpressRoute sont limités. Si les besoins d’un seul client dépassent ces limites, ils auront besoin de plusieurs circuits ExpressRoute pour leur implémentation réseau hybride.
> 
> 

Microsoft Azure fournit un nombre croissant de services que vous pouvez proposer à vos clients. ExpressRoute vous aide, vous et vos clients, à tirer profit de ces services en fournissant un accès à grande vitesse et faible latence à l’environnement Microsoft Azure.

## <a name="microsoft-azure-management"></a>Gestion de Microsoft Azure
Microsoft fournit des API aux fournisseurs de solutions Cloud pour gérer les abonnements client Azure en permettant l’intégration de la programmation à vos propres systèmes de management des services. Les fonctionnalités de gestion prises en charge sont indiquées [ici](/previous-versions/windows/mt844538(v=win.10)).

## <a name="microsoft-azure-resource-management"></a>Gestion des ressources Microsoft Azure
Le contrat signé avec votre client détermine le mode de gestion de l’abonnement. Le fournisseur de solutions Cloud peut gérer directement la création et la maintenance des ressources ou bien le client peut garder le contrôle de l’abonnement Microsoft Azure et créer les ressources Azure requises. Si vos clients gèrent la création des ressources dans leur abonnement Microsoft Azure, ils doivent utiliser l’un des deux modèles disponibles : le modèle «  *Connect-Through*  » ou le modèle «  *Direct-To*  ». Ces modèles sont décrits en détail dans les sections suivantes.  

### <a name="connect-through-model"></a>Modèle « Connect-through »
![Diagramme montrant le modèle « Connect-through ».](./media/expressroute-for-cloud-solution-providers/connect-through.png)  

Dans le modèle « Connect-through », le fournisseur de solutions Cloud crée une connexion directe entre votre centre de données et l’abonnement Azure de votre client. La connexion directe est établie à l’aide d’ExpressRoute et connecte votre réseau à Azure. Votre client se connecte ensuite à votre réseau. Ce scénario nécessite que le client traverse le réseau du fournisseur de solutions Cloud pour accéder aux services Azure. 

Si vos clients possèdent d’autres abonnements Azure non gérés par vous, ils doivent utiliser le réseau Internet public ou leur propre connexion privée pour se connecter à ces services approvisionnés dans le cadre de l’abonnement non-fournisseur de solutions Cloud. 

Les fournisseurs de solutions Cloud qui gèrent des services Azure doivent disposer d’une banque d’identités client déjà établie qui est ensuite répliquée dans Azure Active Directory pour permettre la gestion de leur abonnement de fournisseur de solutions Cloud par le biais d’AOBO (Administrate-On-Behalf-Of). Les principaux facteurs motivant l’adoption de ce scénario sont les suivants : un partenaire ou un fournisseur de services donné a une relation bien établie avec le client, le client utilise actuellement les services d’un fournisseur ou bien le partenaire souhaite proposer une combinaison de solutions hébergées par le fournisseur et Azure afin d’apporter davantage de flexibilité et de répondre aux défis du client qui ne peuvent pas être résolus uniquement par le fournisseur de solutions Cloud. Ce modèle est présenté dans la **figure** ci-dessous.

![Diagramme montrant un scénario détaillé du modèle « Connect-through ».](./media/expressroute-for-cloud-solution-providers/connect-through-model.png)

### <a name="connect-to-model"></a>Modèle « Connect-to »
![Diagramme montrant le modèle « Connect-To ».](./media/expressroute-for-cloud-solution-providers/connect-to.png)

Dans le modèle « Connect-to », le fournisseur de services crée une connexion directe entre le centre de données de ses clients et l’abonnement Azure approvisionné par le fournisseur de solutions Cloud à l’aide d’ExpressRoute sur le réseau des clients.

> [!NOTE]
> Pour ExpressRoute, le client doit créer et gérer le circuit ExpressRoute.  
> 
> 

Ce scénario de connectivité nécessite que le client se connecte directement via un réseau client pour accéder à l’abonnement Azure géré par le fournisseur de solutions Cloud, à l’aide d’une connexion réseau directe créée, détenue et gérée entièrement ou en partie par le client. Pour ces clients, cela suppose que le fournisseur ne dispose pas actuellement d’une banque d’identités client établie et que le fournisseur aide le client à répliquer sa banque d’identités en cours dans Azure Active Directory pour la gestion de son abonnement via AOBO. Les principaux facteurs motivant l’adoption de ce scénario sont les suivants : un partenaire ou un fournisseur de services donné a une relation bien établie avec le client, le client utilise actuellement les services d’un fournisseur ou bien le partenaire souhaite proposer des services basés uniquement sur des solutions hébergées par Azure sans avoir besoin de disposer d’un centre de données ou d’une infrastructure fournisseur.

![Diagramme montrant un scénario détaillé pour le modèle « Connect-To ».](./media/expressroute-for-cloud-solution-providers/connect-to-model.png)

Le choix de l’une de ces deux options dépend des besoins de vos clients et de votre besoin actuel de fournir des services Azure. Ces modèles et les modèles associés de conception d’identité, de mise en réseau et de contrôle d’accès basé sur les rôles sont présentés en détail aux liens suivants :

* **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)** – Azure RBAC repose sur Azure Active Directory.  Pour plus d’informations concernant Azure RBAC, cliquez [ici](../role-based-access-control/role-assignments-portal.md).
* **Mise en réseau** : couvre les différentes rubriques de mise en réseau dans Microsoft Azure.
* **Azure Active Directory (Azure AD)**  : Azure AD assure la gestion des identités pour Microsoft Azure et les applications SaaS tierces. Pour plus d’informations sur Azure AD, cliquez [ici](https://azure.microsoft.com/documentation/services/active-directory/).  

## <a name="network-speeds"></a>Vitesses du réseau
ExpressRoute prend en charge des vitesses de réseau comprises entre 50 Mo/s et 10 Go/s. Les clients peuvent ainsi acheter la quantité de bande passante réseau dont ils ont besoin pour leur environnement.

> [!NOTE]
> La bande passante réseau peut être augmentée en fonction des besoins sans interrompre les communications mais, pour réduire la vitesse du réseau, il est nécessaire de démonter le circuit et de le reconstruire pour qu’il puisse prendre en charge une vitesse de réseau inférieure.  
> 
> 

ExpressRoute prend en charge la connexion de plusieurs réseaux virtuels à un même circuit ExpressRoute pour une meilleure utilisation des connexions à plus haut débit. Un circuit ExpressRoute peut être partagé entre plusieurs abonnements Azure appartenant au même client.

## <a name="configuring-expressroute"></a>Configuration d’ExpressRoute
ExpressRoute peut être configuré pour prendre en charge trois types de trafic ([domaines de routage](#expressroute-routing-domains)) sur un même circuit ExpressRoute. Ce trafic se répartit entre peering privé, peering Microsoft, peering public (déconseillé). Vous pouvez choisir d’envoyer un ou tous les types de trafic vers un circuit ExpressRoute ou utiliser plusieurs circuits ExpressRoute selon la taille du circuit ExpressRoute et l’isolement requis par votre client. La posture de sécurité de votre client peut ne pas autoriser le trafic public et privé à traverser le même circuit.

### <a name="connect-through-model"></a>Modèle « Connect-through »
Dans une configuration « Connect-through », vous êtes responsable de tous les fondements de la mise en réseau pour connecter les ressources de centre de données de vos clients aux abonnements hébergés dans Azure. Chacun de vos clients souhaitant utiliser les fonctionnalités Azure doit disposer de sa propre connexion ExpressRoute, qui est gérée par vous. Vous devez utiliser les mêmes méthodes que celles que le client utiliserait pour fournir le circuit ExpressRoute. Suivez les mêmes étapes répertoriées dans l’article [Flux de travail ExpressRoute](expressroute-workflows.md) d’approvisionnement du circuit et états du circuit. Vous devez ensuite configurer les itinéraires du protocole BGP (Border Gateway Protocol) pour contrôler le trafic entre le réseau local et le réseau virtuel Azure.

### <a name="connect-to-model"></a>Modèle « Connect-to »
Dans une configuration « Connect-to », votre client dispose déjà d’une connexion vers Azure ou va établir une connexion avec le fournisseur de services Internet en assurant la liaison entre ExpressRoute depuis leur centre de données directement vers Azure, au lieu de votre centre de données. Pour commencer le processus d’approvisionnement, votre client doit suivre les étapes présentées dans le modèle « Connect-to », ci-dessus. Une fois le circuit établi, votre client va devoir configurer les routeurs locaux pour pouvoir accéder à votre réseau et aux réseaux virtuels Azure.

Vous pouvez l’aider en configurant la connexion et les itinéraires afin que les ressources de votre ou vos centres de données puissent communiquer avec les ressources client dans votre centre de données ou avec les ressources hébergées dans Azure.

## <a name="expressroute-routing-domains"></a>Domaines de routage ExpressRoute
ExpressRoute propose deux domaines de routage pour les nouveaux circuits : le peering privé et le peering Microsoft. Chaque domaine de routage est configuré avec des routeurs identiques dans une configuration actif-actif pour la haute disponibilité. Pour plus d’informations sur les domaines de routage ExpressRoute, cliquez [ici](expressroute-circuit-peerings.md).

Vous pouvez définir des filtres d’itinéraires personnalisés pour autoriser uniquement le ou les itinéraires souhaités. Pour en savoir plus ou pour savoir comment effectuer ces modifications, consultez l’article : [Créer et modifier le routage pour un circuit ExpressRoute à l’aide de PowerShell](expressroute-howto-routing-classic.md) pour plus d’informations sur les filtres de routage.

> [!NOTE]
> Pour le peering Microsoft, la connectivité doit utiliser une adresse IP publique appartenant au client ou au fournisseur de services Cloud et doit respecter toutes les règles définies. Pour plus d’informations, consultez la page [Conditions préalables d’ExpressRoute](expressroute-prerequisites.md) .  
> 
> 

## <a name="routing"></a>Routage
ExpressRoute se connecte aux réseaux Azure via la passerelle de réseau virtuel Azure. Les passerelles de réseau assurent le routage pour les réseaux virtuels Azure.

La création de réseaux virtuels Azure crée également une table de routage par défaut permettant au réseau virtuel de diriger le trafic vers/depuis les sous-réseaux du réseau virtuel. Si la table de routage par défaut est insuffisante pour la solution, des itinéraires personnalisés peuvent être créés afin d’acheminer le trafic sortant vers des appareils personnalisés ou de bloquer des itinéraires vers des sous-réseaux ou des réseaux externes.

### <a name="default-routing"></a>Routage par défaut
La table de routage par défaut inclut les itinéraires suivants :

* Routage dans un sous-réseau
* Sous-réseau vers sous-réseau au sein du réseau virtuel
* Vers Internet
* Réseau virtuel vers réseau virtuel à l’aide de la passerelle VPN
* Réseau virtuel vers réseau local à l’aide d’une passerelle VPN ou ExpressRoute

![Diagramme montrant les options de routage par défaut.](./media/expressroute-for-cloud-solution-providers/default-routing.png)  

### <a name="user-defined-routing-udr"></a>Itinéraire défini par l’utilisateur (UDR)
Les itinéraires définis par l’utilisateur permettent de contrôler le trafic sortant du sous-réseau attribué à d’autres sous-réseaux dans le réseau virtuel ou sur l’une des autres passerelles prédéfinies (ExpressRoute, Internet ou VPN). La table de routage système par défaut peut être remplacée par une table de routage définie par l’utilisateur qui remplace alors la table de routage par défaut par des itinéraires personnalisés. Avec le routage défini par l’utilisateur, les clients peuvent créer des itinéraires vers des appareils tels que des pare-feu ou des dispositifs de détection des intrusions, ou bloquer l’accès à des sous-réseaux depuis le sous-réseau qui héberge l’itinéraire défini par l’utilisateur. Pour avoir une vue d’ensemble des itinéraires définis par l’utilisateur, cliquez [ici](../virtual-network/virtual-networks-udr-overview.md). 

## <a name="security"></a>Sécurité
Selon le modèle en cours d’utilisation (Connect-To ou Connect-Through), votre client définit les stratégies de sécurité dans son réseau virtuel ou communique les exigences en matière de stratégie de sécurité au fournisseur de services Cloud pour la configuration de ses réseaux virtuels. Vous pouvez définir les critères de sécurité suivants :

1. **Isolation du client** : la plateforme Azure assure l’isolation du client en stockant les informations d’ID client et de réseau virtuel dans une base de données sécurisée, utilisée pour encapsuler le trafic de chaque client dans un tunnel GRE.
2. **groupe de sécurité réseau (NSG)** permettent de définir le trafic autorisé vers et depuis les sous-réseaux au sein des réseaux virtuels dans Azure. Par défaut, le groupe de sécurité réseau contient des règles de blocage servant à bloquer le trafic d’Internet vers le réseau virtuel et des règles d’autorisation pour le trafic au sein d’un réseau virtuel. Pour plus d’informations concernant les groupes de sécurité réseau, cliquez [ici](https://azure.microsoft.com/blog/network-security-groups/).
3. **Tunneling forcé** : il s’agit d’une option permettant de rediriger le trafic Internet sortant provenant d’Azure vers la connexion ExpressRoute sur le centre de données local. Pour plus d’informations sur le tunneling forcé, cliquez [ici](expressroute-routing.md#advertising-default-routes).  
4. **Cryptage** : même si les circuits ExpressRoute sont dédiés à un client spécifique, il est possible que le fournisseur réseau subisse une attaque, permettant ainsi à un intrus d’examiner le trafic de paquets. Pour éviter ce problème, un client ou un fournisseur de services Cloud peut chiffrer le trafic sur la connexion en définissant des stratégies IPSec en mode tunnel pour tout le trafic circulant entre les ressources locales et les ressources Azure (reportez-vous à l’IPSec en mode tunnel facultatif pour le client 1 dans la Figure 5 : Sécurité ExpressRoute, ci-dessus). La deuxième option consiste à utiliser un dispositif pare-feu sur chaque point de terminaison du circuit ExpressRoute. Cela nécessite l’installation de machines virtuelles/d’appliances de pare-feu tierces supplémentaires à chaque extrémité afin de chiffrer le trafic sur le circuit ExpressRoute.

![texte de remplacement](./media/expressroute-for-cloud-solution-providers/expressroute-security.png)  

## <a name="next-steps"></a>Étapes suivantes
Le service du fournisseur de solutions Cloud vous permet d’augmenter la valeur ajoutée offerte à vos clients sans avoir à effectuer des investissements conséquents en matière d’infrastructure et de fonctionnalités, tout en conservant votre position de prestataire principal. L’intégration en toute transparence avec Microsoft Azure peut être effectuée via l’API du fournisseur de services Cloud, qui vous permet d’intégrer la gestion de Microsoft Azure au sein de vos infrastructures de gestion existantes.  

Vous trouverez des informations supplémentaires en cliquant sur les liens suivants :

[Azure et le programme des fournisseurs de solutions cloud](/azure/cloud-solution-provider).  
[Préparez-vous à opérer en tant que fournisseur de solutions Cloud](https://partner.microsoft.com/solutions/cloud-reseller-pre-launch).  
[Ressources des fournisseurs de solutions Cloud Microsoft](https://partner.microsoft.com/solutions/cloud-reseller-resources).