---
title: Utiliser des points de terminaison privés
titleSuffix: Azure SignalR Service
description: Vue d'ensemble des points de terminaison privés pour un accès sécurisé à Azure SignalR Service à partir de réseaux virtuels.
services: signalr
author: ArchangelSDY
ms.service: signalr
ms.topic: article
ms.date: 05/06/2020
ms.author: dayshen
ms.openlocfilehash: 80369883b84ca30cae475235d41addcfba7e52e1
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92152328"
---
# <a name="use-private-endpoints-for-azure-signalr-service"></a>Utiliser des points de terminaison privés pour Azure SignalR Service

Vous pouvez utiliser des [points de terminaison privés](../private-link/private-endpoint-overview.md) pour votre instance d'Azure SignalR Service afin de permettre aux clients d'un réseau virtuel (VNet) d'accéder en toute sécurité aux données via une liaison [Private Link](../private-link/private-link-overview.md). Le point de terminaison privé utilise une adresse IP de l'espace d'adressage du réseau virtuel pour votre instance d'Azure SignalR Service. Le trafic réseau entre les clients du réseau virtuel et Azure SignalR Service passe par une liaison privée du réseau principal Microsoft, ce qui élimine l'exposition à l'Internet public.

L'utilisation de points de terminaison privés pour votre instance d'Azure SignalR Service vous permet de :

- Sécuriser votre instance d'Azure SignalR Service à l'aide du contrôle d'accès réseau afin de bloquer toutes les connexions sur le point de terminaison public pour Azure SignalR Service
- Améliorer la sécurité du réseau virtuel en vous permettant de bloquer l'exfiltration des données à partir du réseau virtuel
- Vous connecter en toute sécurité à Azure SignalR Service à partir de réseaux locaux qui se connectent au réseau virtuel à l'aide d'un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou d'[ExpressRoutes](../expressroute/expressroute-locations.md) avec un peering privé

## <a name="conceptual-overview"></a>Vue d'ensemble conceptuelle

![Vue d'ensemble des points de terminaison privés pour Azure SignalR Service](media/howto-private-endpoints/private-endpoint-overview.png)

Un point de terminaison privé est une interface réseau spéciale pour un service Azure dans votre [réseau virtuel](../virtual-network/virtual-networks-overview.md). Lorsque vous créez un point de terminaison privé pour votre instance d'Azure SignalR Service, il offre une connectivité sécurisée entre les clients de votre réseau virtuel et votre service. Une adresse IP est attribuée au point de terminaison privé à partir de la plage d’adresses IP de votre réseau virtuel. La connexion entre le point de terminaison privé et Azure SignalR Service utilise une liaison privée.

Les applications du réseau virtuel peuvent se connecter en toute transparence à Azure SignalR Service sur le point de terminaison privé, **avec les chaînes de connexion et mécanismes d'autorisation habituels**. Les points de terminaison privés peuvent être utilisés avec tous les protocoles pris en charge par Azure SignalR Service, notamment l'API REST.

Lorsque vous créez un point de terminaison privé pour une instance Azure SignalR Service de votre réseau virtuel, une demande de consentement est envoyée pour approbation au propriétaire de l'instance d'Azure SignalR Service. Si l'utilisateur qui demande la création du point de terminaison privé est également propriétaire de l'instance d'Azure SignalR Service, cette demande de consentement est automatiquement approuvée.

Les propriétaires d'instances d'Azure SignalR Service peuvent gérer les demandes de consentement et les points de terminaison privés via l'onglet « *Points de terminaison privés* » de l'instance d'Azure SignalR Service sur le [portail Azure](https://portal.azure.com).

> [!TIP]
> Si vous souhaitez restreindre l'accès à votre instance d'Azure SignalR Service uniquement par le biais du point de terminaison privé, [configurez le contrôle d'accès réseau](howto-network-access-control.md#managing-network-access-control) de manière à refuser ou contrôler l'accès via le point de terminaison public.

### <a name="connecting-to-private-endpoints"></a>Connexion à des points de terminaison privés

Les clients d'un réseau virtuel qui utilisent le point de terminaison privé doivent utiliser la même chaîne de connexion pour l'instance d'Azure SignalR Service que les clients qui se connectent au point de terminaison public. Nous nous appuyons sur la résolution DNS pour acheminer automatiquement les connexions entre le réseau virtuel et Azure SignalR Service via une liaison privée.

> [!IMPORTANT]
> Utilisez la même chaîne de connexion pour vous connecter à Azure SignalR Service avec des points de terminaison privés. Ne vous connectez pas à Azure SignalR Service à l'aide de son URL de sous-domaine `privatelink`.

Nous créons une [zone DNS privée](../dns/private-dns-overview.md) attachée au réseau virtuel avec les mises à jour nécessaires pour les points de terminaison privés, par défaut. Toutefois, si vous utilisez votre propre serveur DNS, vous devrez peut-être apporter des modifications supplémentaires à votre configuration DNS. La section sur les [modifications DNS](#dns-changes-for-private-endpoints) ci-dessous décrit les mises à jour requises pour les points de terminaison privés.

## <a name="dns-changes-for-private-endpoints"></a>Modifications DNS pour les points de terminaison privés

Lorsque vous créez un point de terminaison privé, l'enregistrement de ressource CNAME DNS de votre instance d'Azure SignalR Service est remplacé dans un sous-domaine par un alias doté du préfixe `privatelink`. Par défaut, nous créons également une [zone DNS privée](../dns/private-dns-overview.md) correspondant au sous-domaine `privatelink`, avec les enregistrements de ressource DNS A pour les points de terminaison privés.

Lorsque vous résolvez votre nom de domaine Azure SignalR Service à l'extérieur du réseau virtuel avec le point de terminaison privé, il correspond au point de terminaison public de l'instance d'Azure SignalR Service. En cas de résolution à partir du réseau virtuel hébergeant le point de terminaison privé, le nom de domaine correspond à l'adresse IP du point de terminaison privé.

Pour l'exemple illustré ci-dessus, les enregistrements de ressources DNS correspondant à Azure SignalR Service « foobar », lorsqu'ils sont résolus à l'extérieur du réseau virtuel hébergeant le point de terminaison privé, sont les suivants :

| Nom                                                  | Type  | Valeur                                                 |
| :---------------------------------------------------- | :---: | :---------------------------------------------------- |
| ``foobar.service.signalr.net``                        | CNAME | ``foobar.privatelink.service.signalr.net``            |
| ``foobar.privatelink.service.signalr.net``            | Un     | \<Azure SignalR Service public IP address\>           |

Comme mentionné précédemment, vous pouvez refuser ou contrôler l'accès pour les clients situés en dehors du réseau virtuel via le point de terminaison public à l'aide du contrôle d'accès réseau.

Les enregistrements de ressources DNS correspondant à « foobar », lorsqu'ils sont résolus par un client du réseau virtuel hébergeant le point de terminaison privé, sont les suivants :

| Nom                                                  | Type  | Valeur                                                 |
| :---------------------------------------------------- | :---: | :---------------------------------------------------- |
| ``foobar.service.signalr.net``                        | CNAME | ``foobar.privatelink.service.signalr.net``            |
| ``foobar.privatelink.service.signalr.net``            | Un     | 10.1.1.5                                              |

Cette approche permet d'accéder à Azure SignalR Service **avec la même chaîne de connexion** pour les clients du réseau virtuel hébergeant les points de terminaison privés, ainsi que pour les clients situés en dehors du réseau virtuel.

Si vous utilisez un serveur DNS personnalisé sur votre réseau, les clients doivent pouvoir résoudre le nom de domaine complet du point de terminaison de l'instance d'Azure SignalR Service sur l'adresse IP du point de terminaison privé. Vous devez configurer votre serveur DNS de manière à déléguer votre sous-domaine de liaison privée à la zone DNS privée du réseau virtuel, ou configurer les enregistrements A pour `foobar.privatelink.service.signalr.net` avec l'adresse IP du point de terminaison privé.

> [!TIP]
> Lorsque vous utilisez un serveur DNS personnalisé ou local, vous devez configurer votre serveur DNS de manière à résoudre le nom de l'instance d'Azure SignalR Service dans le sous-domaine `privatelink` sur l'adresse IP du point de terminaison privé. Pour ce faire, vous pouvez déléguer le sous-domaine `privatelink` à la zone DNS privée du réseau virtuel, ou configurer la zone DNS sur votre serveur DNS et ajouter les enregistrements A DNS.

Le nom de zone DNS recommandé pour les points de terminaison privés de l'instance d'Azure SignalR Service est le suivant : `privatelink.service.signalr.net`.

Pour plus d’informations sur la configuration de votre propre serveur DNS pour la prise en charge des points de terminaison privés, reportez-vous aux articles suivants :

- [Résolution de noms pour des ressources dans les réseaux virtuels Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)
- [Configuration DNS pour les points de terminaison privés](../private-link/private-endpoint-overview.md#dns-configuration)

## <a name="create-a-private-endpoint"></a>Créer un Private Endpoint

### <a name="create-a-private-endpoint-along-with-a-new-azure-signalr-service-in-the-azure-portal"></a>Créer un point de terminaison privé avec une nouvelle instance d'Azure SignalR Service sur le portail Azure

1. Lors de la création d'une instance d'Azure SignalR Service, sélectionnez l'onglet **Mise en réseau**. Choisissez **Point de terminaison privé** comme méthode de connectivité.

    ![Créer une instance d'Azure SignalR Service - Onglet Mise en réseau](media/howto-private-endpoints/portal-create-blade-networking-tab.png)

1. Cliquez sur **Add**. Renseignez les champs Abonnement, Groupe de ressources, Emplacement et Nom du nouveau point de terminaison privé. Choisissez un réseau virtuel et un sous-réseau.

    ![Créer une instance d'Azure SignalR Service - Ajouter un point de terminaison privé](media/howto-private-endpoints/portal-create-blade-add-private-endpoint.png)

1. Cliquez sur **Vérifier + créer**.

### <a name="create-a-private-endpoint-for-an-existing-azure-signalr-service-in-the-azure-portal"></a>Créer un point de terminaison privé pour une instance d'Azure SignalR Service existante sur le portail Azure

1. Accédez à l'instance d'Azure SignalR Service.

1. Cliquez sur le menu de paramètres **Connexions des points de terminaison privés**.

1. Cliquez sur le bouton **+ Point de terminaison privé** en haut.

    ![Panneau Connexions des points de terminaison privés](media/howto-private-endpoints/portal-private-endpoint-connections-blade.png)

1. Renseignez les champs Abonnement, Groupe de ressources, Nom de la ressource et Région du nouveau point de terminaison privé.
    
    ![Créer un point de terminaison privé - Notions de base](media/howto-private-endpoints/portal-create-private-endpoint-basics.png)

1. Choisissez l'instance Azure SignalR Service cible.

    ![Créer un point de terminaison privé - Ressource](media/howto-private-endpoints/portal-create-private-endpoint-resource.png)

1. Choisir le réseau virtuel cible

    ![Créer un point de terminaison privé - Configuration](media/howto-private-endpoints/portal-create-private-endpoint-configuration.png)

1. Cliquez sur **Vérifier + créer**.

### <a name="create-a-private-endpoint-using-azure-cli"></a>Créer une instance Private Endpoint à l’aide d’Azure CLI

1. Se connecter à l’interface Azure CLI
    ```console
    az login
    ```
1. Sélectionner votre abonnement Azure
    ```console
    az account set --subscription {AZURE SUBSCRIPTION ID}
    ```
1. Créer un groupe de ressources
    ```console
    az group create -n {RG} -l {AZURE REGION}
    ```
1. Inscrire Microsoft.SignalRService en tant que fournisseur
    ```console
    az provider register -n Microsoft.SignalRService
    ```
1. Créer une nouvelle instance Azure SignalR Service
    ```console
    az signalr create --name {NAME} --resource-group {RG} --location {AZURE REGION} --sku Standard_S1
    ```
1. Création d'un réseau virtuel
    ```console
    az network vnet create --resource-group {RG} --name {vNet NAME} --location {AZURE REGION}
    ```
1. Ajouter un sous-réseau
    ```console
    az network vnet subnet create --resource-group {RG} --vnet-name {vNet NAME} --name {subnet NAME} --address-prefixes {addressPrefix}
    ```
1. Désactiver les stratégies de réseau virtuel
    ```console
    az network vnet subnet update --name {subnet NAME} --resource-group {RG} --vnet-name {vNet NAME} --disable-private-endpoint-network-policies true
    ```
1. Ajouter une zone DNS privée
    ```console
    az network private-dns zone create --resource-group {RG} --name privatelink.service.signalr.net
    ```
1. Lier la zone DNS privée au réseau virtuel
    ```console
    az network private-dns link vnet create --resource-group {RG} --virtual-network {vNet NAME} --zone-name privatelink.service.signalr.net --name {dnsZoneLinkName} --registration-enabled true
    ```
1. Créer un point de terminaison privé (approuver automatiquement)
    ```console
    az network private-endpoint create --resource-group {RG} --vnet-name {vNet NAME} --subnet {subnet NAME} --name {Private Endpoint Name}  --private-connection-resource-id "/subscriptions/{AZURE SUBSCRIPTION ID}/resourceGroups/{RG}/providers/Microsoft.SignalRService/SignalR/{NAME}" --group-ids signalr --connection-name {Private Link Connection Name} --location {AZURE REGION}
    ```
1. Créer un point de terminaison privé (demander manuellement une approbation)
    ```console
    az network private-endpoint create --resource-group {RG} --vnet-name {vNet NAME} --subnet {subnet NAME} --name {Private Endpoint Name}  --private-connection-resource-id "/subscriptions/{AZURE SUBSCRIPTION ID}/resourceGroups/{RG}/providers/Microsoft.SignalRService/SignalR/{NAME}" --group-ids signalr --connection-name {Private Link Connection Name} --location {AZURE REGION} --manual-request
    ```
1. Afficher l’état de la connexion
    ```console
    az network private-endpoint show --resource-group {RG} --name {Private Endpoint Name}
    ```

## <a name="pricing"></a>Tarifs

Pour plus d’informations sur les tarifs, consultez [Tarification Liaison privée Azure](https://azure.microsoft.com/pricing/details/private-link).

## <a name="known-issues"></a>Problèmes connus

Gardez à l'esprit les problèmes connus suivants concernant les points de terminaison privés pour Azure SignalR Service

### <a name="free-tier"></a>Niveau Gratuit

Vous ne pouvez pas créer de point de terminaison privé pour le niveau gratuit d'Azure SignalR Service.

### <a name="access-constraints-for-clients-in-vnets-with-private-endpoints"></a>Contraintes d'accès pour les clients des réseaux virtuels dotés de points de terminaison privés

Les clients des réseaux virtuels dotés de points de terminaison privés sont soumis à des contraintes lors de l'accès à d'autres instances d'Azure SignalR Service qui disposent de points de terminaison privés. Par exemple, supposons qu'un réseau virtuel N1 possède un point de terminaison privé pour une instance d'Azure SignalR Service S1. Si Azure SignalR Service S2 possède un point de terminaison privé sur un réseau virtuel N2, les clients du réseau virtuel N1 doivent également accéder à Azure SignalR Service S2 à l'aide d'un point de terminaison privé. Si Azure SignalR Service S2 ne possède pas de points de terminaison privés, les clients du réseau virtuel N1 peuvent accéder à l'instance Azure SignalR Service de ce compte sans point de terminaison privé.

Cette contrainte résulte des modifications DNS effectuées lorsqu'Azure SignalR Service S2 crée un point de terminaison privé.

### <a name="network-security-group-rules-for-subnets-with-private-endpoints"></a>Règles de groupe de sécurité réseau pour les sous-réseaux avec des points de terminaison privés

Actuellement, vous ne pouvez pas configurer de règles de [groupe de sécurité réseau](../virtual-network/network-security-groups-overview.md) (NSG, Network Security Group) ni de routes définies par l’utilisateur pour des points de terminaison privés. Les règles NSG appliquées au sous-réseau hébergeant le point de terminaison privé sont appliquées au point de terminaison privé. Une solution de contournement limitée pour ce problème consiste à implémenter vos règles d’accès pour les points de terminaison privés sur les sous-réseaux sources, bien que cette approche puisse nécessiter une charge de gestion supérieure.

## <a name="next-steps"></a>Étapes suivantes

- [Configurer le contrôle d'accès réseau](howto-network-access-control.md)