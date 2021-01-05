---
title: Résoudre les problèmes d’appairage de réseaux virtuels
description: Étapes permettant de résoudre la plupart des problèmes d’appairage de réseaux virtuels.
services: virtual-network
documentationcenter: na
author: v-miegge
manager: dcscontentpm
editor: ''
tags: virtual-network
ms.assetid: 1a3d1e84-f793-41b4-aa04-774a7e8f7719
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/28/2019
ms.author: kaushika
ms.openlocfilehash: 9685c1739a00788a974c200ddabb8cc975696b62
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83587729"
---
# <a name="troubleshoot-virtual-network-peering-issues"></a>Résoudre les problèmes d’appairage de réseaux virtuels

Ce guide de dépannage décrit les étapes de résolution de la plupart des problèmes d’[appairage de réseaux virtuels](virtual-network-peering-overview.md).

![Diagramme de peering de réseaux virtuels](./media/virtual-network-troubleshoot-peering-issues/4489538_en_1.png)

## <a name="configure-virtual-network-peering-between-two-virtual-networks"></a>Configurer l’appairage entre deux réseaux virtuels

Les réseaux virtuels se trouvent-ils dans le même abonnement ou dans des abonnements différents ?

### <a name="the-virtual-networks-are-in-the-same-subscription"></a>Les réseaux virtuels se trouvent dans le même abonnement

Pour configurer le peering des réseaux virtuels qui se trouvent dans le même abonnement, utilisez les méthodes décrites dans les articles suivants :

* Si les réseaux virtuels se trouvent dans la *même région*, consultez [Créer un peering](https://docs.microsoft.com/azure/virtual-network/virtual-network-manage-peering#create-a-peering).
* Si les réseaux virtuels se trouvent dans des *régions distinctes*, consultez [Peering de réseau virtuel](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview). 

> [!Note]
> La connectivité ne fonctionne pas via le peering de réseaux virtuels mondiaux pour les ressources suivantes : 
>
> * Machines virtuelles derrière le SKU ILB (équilibreur de charge interne) de base
> * Cache Redis (utilise le SKU ILB de base)
> * Application Gateway (utilise le SKU ILB de base)
> * Virtual Machine Scale Sets (utilise le SKU ILB de base)
> * Clusters Azure Service Fabric (utilise le SKU ILB de base)
> * SQL Server Always On (utilise le SKU ILB de base)
> * Azure App Service Environment pour PowerApps (utilise le SKU ILB de base)
> * Gestion des API Azure (utilise le SKU ILB de base)
> * Azure AD DS (Azure Active Directory Domain Services) (utilise le SKU ILB de base)

Pour plus d’informations, voir [la configuration requise et les contraintes](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview#requirements-and-constraints) d’un appairage mondial.

### <a name="the-virtual-networks-are-in-different-subscriptions-or-active-directory-tenants"></a>Les réseaux virtuels se trouvent dans des abonnements ou des locataires Active Directory distincts

Pour configurer le peering de réseaux virtuels se trouvant dans des abonnements ou des locataires Active Directory distincts, consultez les informations permettant de [créer un peering dans des abonnements distincts pour Azure CLI](https://docs.microsoft.com/azure/virtual-network/create-peering-different-subscriptions#cli).

> [!Note]
> Pour configurer l’appairage de réseaux, vous devez disposer des autorisations **Contributeur de réseaux** dans les deux abonnements. Pour plus d’informations, voir [Autorisations d’appairage](virtual-network-manage-peering.md#permissions).

## <a name="configure-virtual-network-peering-with-hub-spoke-topology-that-uses-on-premises-resources"></a>Configurer le peering de réseaux virtuels avec une topologie hub-and-spoke qui utilise des ressources locales

![Diagramme de peering de réseaux virtuels avec un spoke local](./media/virtual-network-troubleshoot-peering-issues/4488712_en_1a.png)

### <a name="for-a-site-to-site-connection-or-an-expressroute-connection"></a>Pour une connexion site à site ou une connexion ExpressRoute

Suivez les étapes décrites dans : [Configurer le transit par passerelle VPN pour l’appairage de réseaux virtuels](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-peering-gateway-transit?toc=/azure/virtual-network/toc.json).

### <a name="for-point-to-site-connections"></a>Pour des connexions point à site

1. Suivez les étapes décrites dans : [Configurer le transit par passerelle VPN pour l’appairage de réseaux virtuels](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-peering-gateway-transit?toc=/azure/virtual-network/toc.json).
2. Une fois le peering de réseaux virtuels établi ou changé, téléchargez et réinstallez le package point à site pour que les clients point à site obtiennent la mise à jour des routes vers le réseau virtuel spoke.

## <a name="configure-virtual-network-peering-with-hub-spoke-topology-virtual-network"></a>Configurer le peering de réseaux virtuels avec un réseau virtuel ayant une topologie hub-and-spoke

![Diagramme de peering de réseaux virtuels avec un spoke de réseau virtuel](./media/virtual-network-troubleshoot-peering-issues/4488712_en_1b.png)

### <a name="the-virtual-networks-are-in-the-same-region"></a>Les réseaux virtuels se trouvent dans la même région


1. Dans le réseau virtuel hub, configurez une appliance virtuelle réseau (NVA).
1. Dans les réseaux virtuels spoke, utilisez des routes définies par l’utilisateur ayant pour type de tronçon suivant « appliance virtuelle réseau ».

Pour plus d’informations, voir [Chaînage de services](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview#service-chaining).

> [!Note]
> Si vous avez besoin d’aide pour configurer un appliance virtuelle réseau, [contactez le fournisseur de l’appliance virtuelle réseau](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).

Pour plus d’informations sur la résolution des problèmes de configuration et de routage d’appliance virtuelle réseau, voir [Problèmes d’appliance virtuelle réseau dans Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-troubleshoot-nva).

### <a name="the-virtual-networks-are-in-different-regions"></a>Les réseaux virtuels se trouvent dans des régions différentes

Le transit via le peering de réseaux virtuels mondiaux est désormais pris en charge. La connectivité ne fonctionne pas via le peering de réseaux virtuels mondiaux pour les ressources suivantes :

* Machines virtuelles derrière une référence (SKU) ILB de base
* Cache Redis (utilise le SKU ILB de base)
* Application Gateway (utilise le SKU ILB de base)
* Groupes identiques (utilise le SKU ILB de base)
* Clusters Service Fabric (utilise la référence (SKU) ILB de base)
* SQL Server Always On (utilise le SKU ILB de base)
* App Service Environment (utilise le SKU ILB de base)
* Gestion des API (utilise la référence (SKU) ILB de base)
* Azure AD DS (utilise le SKU ILB de base)

Pour en savoir plus sur les conditions et les contraintes d’un peering mondial, consultez [Peering de réseaux virtuels](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview#requirements-and-constraints).

## <a name="troubleshoot-a-connectivity-issue-between-two-peered-virtual-networks"></a>Résoudre un problème de connectivité entre deux réseaux virtuels appairés

Connectez-vous au [portail Azure](https://portal.azure.com/) avec un compte disposant des [rôles et autorisations](virtual-network-manage-peering.md#permissions) nécessaires. Sélectionnez le réseau virtuel, sélectionnez **Peering**, puis consultez le champ **État**. Qu’est-ce que l’état ?

### <a name="the-peering-status-is-connected"></a>L’état du peering est « Connecté »

Pour résoudre ce problème :

1. Vérifiez les flux de trafic réseau :

   Utilisez les procédures [Résolution des problèmes de connexion](https://docs.microsoft.com/azure/network-watcher/network-watcher-connectivity-overview) et de [Vérification du flux IP](https://docs.microsoft.com/azure/network-watcher/network-watcher-ip-flow-verify-overview) de la machine virtuelle source à la machine virtuelle de destination pour déterminer s’il existe un groupe de sécurité réseau ou un UDR qui provoque des interférences dans les flux de trafic.

   Si vous utilisez un pare-feu ou une appliance virtuelle réseau : 
   1. Documentez les paramètres UDR pour pouvoir les restaurer une fois cette étape effectuée.
   2. Supprimez l’UDR du sous-réseau ou de la carte réseau de la machine virtuelle source qui pointe vers l’appliance virtuelle réseau en tant que tronçon suivant. Vérifiez la connectivité de la machine virtuelle source directement à la destination, qui contourne l’appliance virtuelle réseau. Si cette étape ne fonctionne pas, consultez l’[utilitaire de résolution des problèmes d’appliance virtuelle réseau](https://docs.microsoft.com/azure/virtual-network/virtual-network-troubleshoot-nva).

2. Obtenez une trace réseau : 
   1. Commencez à capturer une trace réseau sur la machine virtuelle de destination. Pour Windows, vous pouvez utiliser la commande **Netsh**. Pour Linux, utilisez la commande **TCPDump**.
   2. Exécutez la commande **TcpPing** ou **PsPing** de la source à la l’adresse IP de destination.

      Voici un exemple de commande **TcpPing** : `tcping64.exe -t <destination VM address> 3389`

   3. Une fois la commande **TcpPing** exécutée, arrêtez la capture de la trace réseau sur la destination.
   4. Si des paquets arrivent de la source, cela signifie qu’il n’y a aucun problème de réseau. Examinez le pare-feu de la machine virtuelle et l’application écoutant sur ce port pour localiser le problème de configuration.

   > [!Note]
   > Vous ne pouvez pas vous connecter aux types de ressources suivants via un appairage de réseaux virtuels globaux (réseaux virtuels se trouvant dans des régions différentes) :
   >
   > * Machines virtuelles derrière une référence (SKU) ILB de base
   > * Cache Redis (utilise le SKU ILB de base)
   > * Application Gateway (utilise le SKU ILB de base)
   > * Groupes identiques (utilise le SKU ILB de base)
   > * Clusters Service Fabric (utilise la référence (SKU) ILB de base)
   > * SQL Server Always On (utilise le SKU ILB de base)
   > * App Service Environment (utilise le SKU ILB de base)
   > * Gestion des API (utilise la référence (SKU) ILB de base)
   > * Azure AD DS (utilise le SKU ILB de base)

Pour plus d’informations, voir [la configuration requise et les contraintes](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview#requirements-and-constraints) d’un appairage mondial.

### <a name="the-peering-status-is-disconnected"></a>L’état du peering est « Déconnecté »

Pour résoudre ce problème, supprimez le peering des deux réseaux virtuels, puis recréez-le.

## <a name="troubleshoot-a-connectivity-issue-between-a-hub-spoke-virtual-network-and-an-on-premises-resource"></a>Résoudre un problème de connectivité entre un réseau virtuel hub-and-spoke et une ressource locale

Est-ce que votre réseau utilise une passerelle VPN ou une appliance virtuelle réseau tierce ?

### <a name="my-network-uses-a-third-party-nva-or-vpn-gateway"></a>Mon réseau utilise une passerelle VPN ou d’appliance virtuelle réseau tierce

Pour résoudre les problèmes de connectivité qui affectent une passerelle VPN ou d’appliance virtuelle réseau tierce, voir les articles suivants :

* [Utilitaire de résolution des problèmes d’appliance virtuelle réseau](https://docs.microsoft.com/azure/virtual-network/virtual-network-troubleshoot-nva)
* [Chaînage de services](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview#service-chaining)

### <a name="my-network-does-not-use-a-third-party-nva-or-vpn-gateway"></a>Mon réseau n’utilise pas de passerelle VPN ou d’appliance virtuelle réseau tierce

Le réseau virtuel hub et le réseau virtuel spoke ont-ils une passerelle VPN ?

#### <a name="both-the-hub-virtual-network-and-the-spoke-virtual-network-have-a-vpn-gateway"></a>Le réseau virtuel hub et le réseau virtuel spoke ont une passerelle VPN

L’utilisation d’une passerelle distante n’est pas prise en charge.

Si le réseau virtuel spoke dispose déjà d’une passerelle VPN, l’option **Utiliser des passerelles distantes** n’est pas prise en charge sur le réseau virtuel spoke. Cela est dû à une limitation du peering de réseaux virtuels.

#### <a name="both-the-hub-virtual-network-and-the-spoke-virtual-network-do-not-have-a-vpn-gateway"></a>Le réseau virtuel hub et le réseau virtuel spoke n’ont pas de passerelle VPN

Pour les connexions site à site ou Azure ExpressRoute, vérifiez les points suivants. Il s’agit des principales causes des problèmes de connectivité au réseau virtuel distant à partir de ressources locales :

* Sur le réseau virtuel doté d’une passerelle, vérifiez que la case **Autoriser le trafic transféré** est cochée.
* Sur le réseau virtuel non doté d’une passerelle, vérifiez que la case **Utiliser des passerelles distantes** est cochée.
* Demandez à votre administrateur réseau de vérifier vos appareils locaux pour s’assurer que l’espace d’adressage du réseau virtuel distant a bien été ajouté à tous.

Pour les connexions point à site :

* Sur le réseau virtuel doté d’une passerelle, vérifiez que la case **Autoriser le trafic transféré** est cochée.
* Sur le réseau virtuel non doté d’une passerelle, vérifiez que la case **Utiliser des passerelles distantes** est cochée.
* Téléchargez et réinstallez le package client point à site. Les routes de réseau virtuel qui viennent d’être appairées n’ajoutent pas automatiquement des routes aux clients point à site.

## <a name="troubleshoot-a-hub-spoke-network-connectivity-issue-between-spoke-virtual-networks-in-the-same-region"></a>Résoudre un problème de connectivité réseau hub-and-spoke entre des réseaux virtuels spoke de même région

Un réseau hub doit inclure une appliance virtuelle réseau. Configurez des UDR dans les spokes qui ont une appliance virtuelle réseau définie en tant que tronçon suivant, puis activez **Autoriser le trafic transféré** dans le réseau virtuel hub.

Pour plus d’informations, voir [Chaînage de services](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview#service-chaining) et discutez de ces exigences avec le [fournisseur de l’appliance virtuelle réseau](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines) de votre choix.

## <a name="troubleshoot-a-hub-spoke-network-connectivity-issue-between-spoke-virtual-networks-in-different-regions"></a>Résoudre un problème de connectivité réseau hub-and-spoke entre des réseaux virtuels spoke de régions distinctes

Le transit via le peering de réseaux virtuels mondiaux est désormais pris en charge. La connectivité ne fonctionne pas via le peering de réseaux virtuels mondiaux pour les ressources suivantes :

* Machines virtuelles derrière une référence (SKU) ILB de base
* Cache Redis (utilise le SKU ILB de base)
* Application Gateway (utilise le SKU ILB de base)
* Groupes identiques (utilise le SKU ILB de base)
* Clusters Service Fabric (utilise la référence (SKU) ILB de base)
* SQL Server Always On (utilise le SKU ILB de base)
* App Service Environment (utilise le SKU ILB de base)
* Gestion des API (utilise la référence (SKU) ILB de base)
* Azure AD DS (utilise le SKU ILB de base)

Pour plus d’informations, consultez les [exigences et contraintes](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview#requirements-and-constraints) d’un peering mondial et les [différentes topologies VPN](https://blogs.msdn.microsoft.com/igorpag/2016/02/11/hubspoke-daisy-chain-and-full-mesh-vnet-topologies-in-azure-arm-v2/).

## <a name="troubleshoot-a-hub-spoke-network-connectivity-issue-between-a-web-app-and-the-spoke-virtual-network"></a>Résoudre un problème de connectivité réseau hub-and-spoke entre une application web et le réseau virtuel spoke

Pour résoudre ce problème :

1. Connectez-vous au portail Azure. 
1. Dans l’application web, sélectionnez **réseau**, puis **Intégration de réseau virtuel**.
1. Vérifiez si vous pouvez voir le réseau virtuel distant. Entrez manuellement l’espace d’adressage du réseau virtuel distant (**Synchroniser le réseau** et **Ajouter des routes**).

Pour plus d’informations, consultez les articles suivants :

* [Intégrer votre application à un réseau virtuel Azure](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet)
* [À propos du routage VPN point à site](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-point-to-site-routing)

## <a name="troubleshoot-a-virtual-network-peering-configuration-error-message"></a>Résoudre les problèmes liés à un message d’erreur de configuration du peering de réseaux virtuels 

### <a name="current-tenant-tenant-id-isnt-authorized-to-access-linked-subscription"></a>Le locataire actuel `<TENANT ID>` n’est pas autorisé à accéder à l’abonnement lié

Pour résoudre ce problème, consultez [Créer un peering - Azure CLI](https://docs.microsoft.com/azure/virtual-network/create-peering-different-subscriptions#cli).

### <a name="not-connected"></a>Non connecté

Pour résoudre ce problème, supprimez le peering des deux réseaux virtuels, puis recréez-le.

### <a name="failed-to-peer-a-databricks-virtual-network"></a>Échec de l’appairage d’un réseau virtuel Databricks

Pour résoudre ce problème, configurez le peering de réseaux virtuels sous **Azure Databricks**, puis spécifiez le réseau virtuel cible à l’aide de l’**ID de la ressource**. Pour plus d’informations, voir [Appairer un réseau virtuel Databricks à un réseau virtuel distant](https://docs.azuredatabricks.net/administration-guide/cloud-configurations/azure/vnet-peering.html#id2).

### <a name="the-remote-virtual-network-lacks-a-gateway"></a>Le réseau virtuel distant n’a pas de passerelle

Ce problème se produit lorsque vous appairez des réseaux virtuels de différents locataires et que vous souhaitez ensuite configurer `Use Remote Gateways`. Une des limites du Portail Azure est qu’il ne peut pas valider la présence d’une passerelle de réseau virtuel dans le réseau virtuel d’un autre locataire.

Il existe deux façons de résoudre le problème :

 * Supprimez les Peerings et activez l’option `Use Remote Gateways` lorsque vous créez un nouveau Peering.
 * Utilisez PowerShell ou l’interface CLI au lieu du Portail Azure pour activer `Use Remote Gateways`.

## <a name="next-steps"></a>Étapes suivantes

* [Résolution des problèmes de connectivité entre machines virtuelles Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-troubleshoot-connectivity-problem-between-vms)
