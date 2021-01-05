---
title: Utilisation de points de terminaison privés pour Azure App Configuration
description: Sécuriser votre magasin App Configuration avec des points de terminaison privés
services: azure-app-configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 07/15/2020
ms.openlocfilehash: 6cadadfb3623d05dd3ae3851acd5eaca13860023
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929841"
---
# <a name="using-private-endpoints-for-azure-app-configuration"></a>Utilisation de points de terminaison privés pour Azure App Configuration

Vous pouvez utiliser des [points de terminaison privés](../private-link/private-endpoint-overview.md) pour Azure App Configuration afin de permettre aux clients d’un réseau virtuel (VNet) d’accéder en toute sécurité aux données via une [liaison privée](../private-link/private-link-overview.md). Le point de terminaison privé utilise une adresse IP de l’espace d’adressage du réseau virtuel pour votre magasin App Configuration. Le trafic réseau entre les clients sur le réseau virtuel et le magasin App Configuration traverse le réseau virtuel en utilisant une liaison privée sur le réseau principal de Microsoft, ce qui élimine l’exposition à l’Internet public.

L’utilisation de points de terminaison privés pour votre magasin App Configuration vous permet d’effectuer les opérations suivantes :
- Sécuriser les détails de configuration de votre application en configurant le pare-feu pour bloquer toutes les connexions à App Configuration sur le point de terminaison public.
- Améliorer la sécurité du réseau virtuel (VNet), en veillant à ce que les données n’échappent pas su réseau virtuel.
- Vous connecter en toute sécurité au magasin App Configuration à partir de réseaux locaux qui se connectent au réseau virtuel à l’aide de [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou [d’ExpressRoutes](../expressroute/expressroute-locations.md) avec un peering privé.

## <a name="conceptual-overview"></a>Vue d'ensemble conceptuelle

Un point de terminaison privé est une interface réseau spéciale pour un service Azure dans votre [réseau virtuel](../virtual-network/virtual-networks-overview.md). Lorsque vous créez un point de terminaison privé pour votre magasin App Configuration, il offre une connectivité sécurisée entre les clients sur votre réseau virtuel et votre magasin de configuration. Une adresse IP est attribuée au point de terminaison privé à partir de la plage d’adresses IP de votre réseau virtuel. La connexion entre le point de terminaison privé et le magasin de configuration utilise une liaison privée sécurisée.

Les applications du réseau virtuel peuvent se connecter au magasin de configuration sur le point de terminaison privé, **en utilisant les mêmes chaînes de connexion et mécanismes d’autorisation que d’habitude**. Les points de terminaison privés peuvent être utilisés avec tous les protocoles pris en charge par le magasin App Configuration.

Bien qu’App Configuration ne prenne pas en charge les points de terminaison de service, des points de terminaison privés peuvent être créés dans des sous-réseaux qui utilisent des [points de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md). Des clients dans un sous-réseau peuvent connecter en toute sécurité à un magasin App Configuration en utilisant le point de terminaison privé, tout en utilisant des points de terminaison de service pour accéder à d’autres.  

Quand vous créez un point de terminaison privé pour un service dans votre réseau virtuel, une demande de consentement est envoyée pour approbation au propriétaire du compte de service. Si l’utilisateur qui demande la création du point de terminaison privé est également propriétaire du compte, cette demande de consentement est automatiquement approuvée.

Les propriétaires de comptes de service peuvent gérer les demandes de consentement et les points de terminaison privés via l’onglet `Private Endpoints` du magasin de configuration dans le [portail Azure](https://portal.azure.com).

### <a name="private-endpoints-for-app-configuration"></a>Points de terminaison privés pour App Configuration 

Lorsque vous créez un point de terminaison privé, vous devez spécifier le magasin App Configuration auquel il se connecte. Si vous avez plusieurs instances App Configuration dans un compte, vous avez besoin d’un point de terminaison privé distinct pour chaque magasin.

### <a name="connecting-to-private-endpoints"></a>Connexion à des points de terminaison privés

Azure s’appuie sur la résolution DNS pour router les connexions du réseau virtuel au magasin de configuration via une liaison privée. Vous pouvez rechercher rapidement des chaînes de connexion dans le portail Azure en sélectionnant votre magasin App Configuration, puis **Paramètres** > **Clés d’accès**.  

> [!IMPORTANT]
> Pour vous connecter à votre magasin App Configuration à l’aide de points de terminaison privés, utilisez la chaîne de connexion que vous utiliseriez pour un point de terminaison public. Ne vous connectez pas au magasin à l’aide de son URL de sous-domaine `privatelink`.

## <a name="dns-changes-for-private-endpoints"></a>Modifications DNS pour les points de terminaison privés

Quand vous créez un point de terminaison privé, l’enregistrement de ressource CNAME DNS pour le magasin de configuration est remplacé par un alias dans un sous-domaine avec le préfixe `privatelink`. Azure crée également une [zone DNS privée](../dns/private-dns-overview.md) correspondant au sous-domaine `privatelink`, avec les enregistrements de ressources DNS A pour les points de terminaison privés.

Quand vous résolvez l’URL du point de terminaison à partir du réseau virtuel hébergeant le point de terminaison privé, elle correspond au point de terminaison privé du magasin. En cas de résolution depuis l’extérieur du réseau virtuel, l’URL du point de terminaison correspond au point de terminaison public. Quand vous créez un point de terminaison privé, le point de terminaison public est désactivé.

Si vous utilisez un serveur DNS personnalisé sur votre réseau, les clients doivent pouvoir résoudre le nom de domaine complet (FQDN) du point de terminaison de service en l’adresse IP du point de terminaison privé. Configurez votre serveur DNS pour déléguer votre sous-domaine de liaison privée à la zone DNS privée du réseau virtuel, ou configurer les enregistrements A pour `AppConfigInstanceA.privatelink.azconfig.io` avec l’adresse IP du point de terminaison privé.

> [!TIP]
> Lorsque vous utilisez un serveur DNS personnalisé ou local, vous devez configurer votre serveur DNS pour résoudre le nom du magasin dans le sous-domaine `privatelink` en l’adresse IP du point de terminaison privé. Pour ce faire, vous pouvez déléguer le sous-domaine `privatelink` à la zone DNS privée du réseau virtuel, ou configurer la zone DNS sur votre serveur DNS et ajouter les enregistrements A DNS.

## <a name="pricing"></a>Tarifs

L’activation des points de terminaison privés requiert un magasin App Configuration de [niveau standard](https://azure.microsoft.com/pricing/details/app-configuration/).  Pour des détails sur la tarification des liaisons privées, consultez [Tarification des liaisons privées Azure](https://azure.microsoft.com/pricing/details/private-link).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la création d’un point de terminaison privé pour votre magasin App Configuration en consultant les articles suivants :

- [Créer un point de terminaison privé à l’aide du centre Private Link dans le portail Azure](../private-link/create-private-endpoint-portal.md)
- [Créer un point de terminaison privé à l’aide d’Azure CLI](../private-link/create-private-endpoint-cli.md)
- [Créer un point de terminaison privé à l’aide d’Azure PowerShell](../private-link/create-private-endpoint-powershell.md)

Apprenez à configurer votre serveur DNS avec des points de terminaison privés :

- [Résolution de noms pour des ressources dans les réseaux virtuels Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)
- [Configuration DNS pour les points de terminaison privés](../private-link/private-endpoint-overview.md#dns-configuration)