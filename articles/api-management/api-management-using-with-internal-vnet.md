---
title: Utiliser la gestion des API Azure avec des réseaux virtuels internes
titleSuffix: Azure API Management
description: Découvrez comment installer et configurer la Gestion des API Azure sur un réseau virtuel interne
services: api-management
documentationcenter: ''
author: vladvino
manager: kjoshi
editor: ''
ms.assetid: dac28ccf-2550-45a5-89cf-192d87369bc3
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/31/2019
ms.author: apimpm
ms.openlocfilehash: 0832c975ecb410b97a24c975f9fc0f4799120abd
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93145512"
---
# <a name="using-azure-api-management-service-with-an-internal-virtual-network"></a>Utiliser le service Gestion des API Azure avec un réseau virtuel interne
Avec les réseaux virtuels Azure, la Gestion des API Azure peut gérer des API inaccessibles sur Internet. Plusieurs technologies VPN sont disponibles pour établir la connexion. La Gestion des API peut être déployée selon deux modes principaux à l’intérieur d’un réseau virtuel :
* Externe
* Interne

En cas de déploiement de Gestion des API dans un mode réseau virtuel interne, tous les points de terminaison de service (passerelle proxy, portail des développeurs, gestion directe et Git) ne sont visibles que dans un réseau virtuel auquel vous contrôlez l’accès. Aucun point de terminaison de service n’est inscrit sur le serveur DNS Public.

> [!NOTE]
> En l'absence d'entrée DNS pour les points de terminaison de service, ces points de terminaison ne sont pas accessibles tant que [DNS est configuré](#apim-dns-configuration) pour le réseau virtuel.

Avec la Gestion des API en mode interne, vous pouvez effectuer les scénarios suivants :

* Rendre les API hébergées dans votre centre de données privé accessibles de l’extérieur en toute sécurité à des tiers à l’aide de connexions site à site ou de connexions Azure VPN ExpressRoute.
* Activer les scénarios de cloud hybride en exposant vos API cloud et sur site par le biais d’une passerelle commune.
* Gérer vos API hébergées dans plusieurs emplacements géographiques à l’aide d’un seul point de terminaison de passerelle.

[!INCLUDE [premium-dev.md](../../includes/api-management-availability-premium-dev.md)]

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes décrites dans cet article, vous devez disposer des éléments suivants :

+ **Un abonnement Azure actif**.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

+ **Une instance du service Gestion des API Azure**. Pour en savoir plus, voir [Créer une instance de gestion des API Azure](get-started-create-service-instance.md).
+ Lorsqu’un service Gestion des API est déployé dans un réseau virtuel, une [liste de ports](./api-management-using-with-vnet.md#required-ports) est utilisée et ces derniers doivent être ouverts. 

## <a name="creating-an-api-management-in-an-internal-virtual-network"></a><a name="enable-vpn"> </a>Créer une Gestion des API dans un réseau virtuel interne
Le service Gestion des API dans un réseau virtuel interne est hébergé derrière un [équilibreur de charge interne (classique)](/previous-versions/azure/load-balancer/load-balancer-get-started-ilb-classic-cloud). Il s'agit là de la seule option disponible et elle ne peut pas être modifiée.

### <a name="enable-a-virtual-network-connection-using-the-azure-portal"></a>Activer une connexion de réseau virtuel à l’aide du portail Azure

1. Accédez à votre instance Gestion des API Azure dans le [portail Azure](https://portal.azure.com/).
2. Sélectionnez **Réseau virtuel**.
3. Configurez l’instance Gestion des API pour la déployer à l’intérieur du réseau virtuel.

    ![Menu pour configurer une Gestion des API Azure dans un réseau virtuel interne][api-management-using-internal-vnet-menu]

4. Sélectionnez **Enregistrer**.

Une fois le déploiement réussi, l'adresse IP virtuelle **privée** et l'adresse IP virtuelle **publique** de votre service Gestion des API doivent s'afficher dans le panneau Vue d’ensemble. L'adresse IP virtuelle **privée** est une adresse IP à charge équilibrée dans le sous-réseau délégué Gestion des API via lequel les points de terminaison `gateway`, `portal`, `management` et `scm` sont accessibles. L’adresse IP virtuelle **publique** est **uniquement** utilisée pour le trafic du plan de contrôle vers le point de terminaison `management` par le biais du port 3443, et peut être verrouillée sur la balise de service [ApiManagement][ServiceTags].

![Tableau de bord Gestion des API avec réseau virtuel interne configuré][api-management-internal-vnet-dashboard]

> [!NOTE]
> La Console de test disponible sur le portail Azure ne fonctionnera pas pour le service de réseau VNET **Interne** déployé, car l’URL de la passerelle n’est pas inscrite sur le DNS Public. À la place, vous devez utiliser la Console de test fournie sur le **Portail des développeurs**.

### <a name="deploy-api-management-into-virtual-network"></a><a name="deploy-apim-internal-vnet"> </a>Déployer Gestion des API dans Réseau virtuel

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-api-management-create-with-internal-vnet%2Fazuredeploy.json)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez également activer une connectivité de réseau virtuel à l’aide d’applets de commande PowerShell.

* Créer un service Gestion des API au sein d’un réseau virtuel : utilisez la cmdlet [New-AzApiManagement](/powershell/module/az.apimanagement/new-azapimanagement) pour créer un service Gestion des API Azure au sein d’un réseau virtuel et le configurer de sorte qu’il utilise le type réseau virtuel interne.

* Mettre à jour le déploiement existant d'un service Gestion des API au sein d’un réseau virtuel : utilisez la cmdlet [Update-AzApiManagementRegion](/powershell/module/az.apimanagement/update-azapimanagementregion) pour déplacer un service Gestion des API existant au sein d’un réseau virtuel et le configurer de sorte qu’il utilise le type de réseau virtuel interne.

## <a name="dns-configuration"></a><a name="apim-dns-configuration"></a>Configuration DNS
Lorsque la Gestion des API se trouve en mode réseau virtuel externe, le DNS est géré par Azure. En mode réseau virtuel interne, vous devez gérer votre propre serveur DNS.

> [!NOTE]
> Le service Gestion des API n’écoute pas les demandes provenant des adresses IP. Il répond uniquement aux demandes pour le nom d’hôte configuré sur ses points de terminaison de service. Ces points de terminaison incluent une passerelle, le portail Azure, le portail des développeurs, un point de terminaison de gestion directe et Git.

### <a name="access-on-default-host-names"></a>Accès sur les noms d’hôtes par défaut
Lorsque vous créez un service Gestion des API, nommé « contosointernalvnet » par exemple, les points de terminaison de service suivants sont configurés par défaut :

   * Passerelle ou proxy : contosointernalvnet.azure-api.net

   * Portail des développeurs : contosointernalvnet.portal.azure-api.net

   * Nouveau portail des développeurs : contosointernalvnet.developer.azure-api.net

   * Point de terminaison de gestion directe : contosointernalvnet.management.azure-api.net

   * Git : contosointernalvnet.scm.azure-api.net

Pour accéder à ces points de terminaison de service Gestion des API, vous pouvez créer une machine virtuelle dans un sous-réseau connecté au réseau virtuel dans lequel la Gestion des API est déployée. En supposant que l’adresse IP virtuelle interne de votre service est 10.1.0.5, vous pouvez effectuer le mappage des fichiers hôtes, %SystemDrive%\drivers\etc\hosts, de la façon suivante :

   * 10.1.0.5     contosointernalvnet.azure-api.net

   * 10.1.0.5     contosointernalvnet.portal.azure-api.net

   * 10.1.0.5     contosointernalvnet.developer.azure-api.net

   * 10.1.0.5     contosointernalvnet.management.azure-api.net

   * 10.1.0.5     contosointernalvnet.scm.azure-api.net

Vous pouvez alors accéder à tous les points de terminaison de service à partir de la machine virtuelle que vous avez créée.
Si vous utilisez un serveur DNS personnalisé dans un réseau virtuel, vous pouvez également créer des enregistrements DNS A et accéder à ces points de terminaison à partir de l’endroit de votre choix dans votre réseau virtuel.

### <a name="access-on-custom-domain-names"></a>Accès sur des noms de domaines personnalisés

1. Si vous ne souhaitez pas accéder au service Gestion des API avec les noms d’hôtes par défaut, vous pouvez configurer des noms de domaines personnalisés pour tous vos points de terminaison de service, comme indiqué dans l’image suivante :

   ![Configuration d’un domaine personnalisé pour la Gestion des API][api-management-custom-domain-name]

2. Vous pouvez ensuite créer des enregistrements dans votre serveur DNS de façon à accéder aux points de terminaison qui ne sont accessibles qu’à partir de votre réseau virtuel.

## <a name="routing"></a><a name="routing"> </a> Routage

* Une adresse IP virtuelle *privée* à charge équilibrée de la plage de sous-réseau est réservée et utilisée pour accéder aux points de terminaison du service Gestion des API à partir du réseau virtuel. Cette adresse IP *privée* se trouve dans le panneau Vue d’ensemble du service sur le portail Azure. Cette adresse doit être inscrite sur les serveurs DNS utilisés par le réseau virtuel.
* Une adresse IP *publique* à charge équilibrée (adresse IP virtuelle) est également réservée pour fournir l’accès au point de terminaison de service de gestion sur le port 3443. Cette adresse IP *publique* se trouve dans le panneau Vue d’ensemble du service sur le portail Azure. L’adresse IP *publique* est uniquement utilisée pour le trafic du plan de contrôle vers le point de terminaison `management` par le biais du port 3443, et peut être verrouillée sur la balise de service [ApiManagement][ServiceTags].
* Les adresses IP de la plage d’adresses IP de sous-réseau (DIP) sont attribuées à chaque machine virtuelle du service et utilisées pour accéder aux ressources au sein du réseau virtuel. Une adresse IP publique est utilisée pour accéder aux ressources extérieures au réseau virtuel. Si des listes de restriction d’adresses IP sont utilisées pour sécuriser les ressources au sein du réseau virtuel, la plage entière du sous-réseau où le service Gestion des API est déployé doit être spécifiée pour accorder ou restreindre l’accès à partir du service.
* Les adresses IP privée et publique à charge équilibrée se trouvent dans le panneau Vue d’ensemble sur le portail Azure.
* Les adresses IP attribuées pour un accès public et privé peuvent changer si le service est supprimé, puis rajouté au réseau virtuel. Dans ce cas, il peut être nécessaire de mettre à jour les enregistrements DNS, règles d'acheminement et listes de restriction d'adresses IP au sein du réseau virtuel.

## <a name="related-content"></a><a name="related-content"> </a>Contenu associé
Pour en savoir plus, consultez les articles suivants :
* [Problèmes courants de configuration réseau lors de la configuration de la Gestion des API Azure dans un réseau virtuel][Common network configuration problems]
* [FAQ des réseaux virtuels](../virtual-network/virtual-networks-faq.md)
* [Création d’un enregistrement A dans DNS](/previous-versions/windows/it-pro/windows-2000-server/bb727018(v=technet.10))

[api-management-using-internal-vnet-menu]: ./media/api-management-using-with-internal-vnet/api-management-using-with-internal-vnet.png
[api-management-internal-vnet-dashboard]: ./media/api-management-using-with-internal-vnet/api-management-internal-vnet-dashboard.png
[api-management-custom-domain-name]: ./media/api-management-using-with-internal-vnet/api-management-custom-domain-name.png

[Create API Management service]: get-started-create-service-instance.md
[Common network configuration problems]: api-management-using-with-vnet.md#network-configuration-issues

[ServiceTags]: ../virtual-network/network-security-groups-overview.md#service-tags