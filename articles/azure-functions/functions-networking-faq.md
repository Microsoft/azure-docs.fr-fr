---
title: Questions fréquentes concernant les réseaux dans Azure Functions
description: Répond à certaines questions courantes concernant les réseaux dans Azure Functions.
ms.topic: troubleshooting
ms.date: 4/11/2019
ms.reviewer: glenga
ms.openlocfilehash: 3e8a992aac95b6c2688cb45aa980bf0b01883a53
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94578227"
---
# <a name="frequently-asked-questions-about-networking-in-azure-functions"></a>Questions fréquentes concernant les réseaux dans Azure Functions

Cet article liste les questions fréquentes concernant les réseaux dans Azure Functions. Pour obtenir une présentation plus complète, consultez [Options de réseau dans Azure Functions](functions-networking-options.md).

## <a name="how-do-i-set-a-static-ip-in-functions"></a>Comment définir une adresse IP statique dans Azure Functions ?

Le seul moyen d’obtenir une adresse IP statique entrante et sortante pour votre fonction est de déployer celle-ci dans App Service Environment. Pour plus d’informations sur l’utilisation d’App Service Environment, commencez par lire l’article [Créer et utiliser un équilibreur de charge interne avec App Service Environment](../app-service/environment/create-ilb-ase.md).

## <a name="how-do-i-restrict-internet-access-to-my-function"></a>Comment limiter l’accès Internet à ma fonction ?

Vous pouvez limiter l’accès Internet de deux façons :

* [Restrictions d’adresse IP](../app-service/app-service-ip-restrictions.md) : limite le trafic entrant vers votre application de fonction en se basant sur la plage d’adresses IP.
    * Lorsque des restrictions d’adresse IP sont appliquées, vous pouvez également configurer des [points de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md),afin de contraindre votre fonction à n’accepter que le trafic entrant provenant d’un réseau virtuel particulier.
* Suppression de tous les déclencheurs HTTP. Pour certaines applications, il suffit simplement d’éviter les déclencheurs HTTP et d’utiliser toute autre source d’événement pour déclencher votre fonction.

N’oubliez pas que l’éditeur du portail Azure nécessite un accès direct à la fonction actuellement exécutée. Pour que les modifications apportées au code via le portail Azure soient appliquées, l'ID de l'appareil que vous utilisez pour accéder au portail doit être ajouté à la liste approuvée. Toutefois, vous pouvez aussi utiliser l’une des options qui se trouvent sous l’onglet Fonctionnalités de la plateforme lorsque des restrictions de réseau sont appliquées.

## <a name="how-do-i-restrict-my-function-app-to-a-virtual-network"></a>Comment limiter mon application de fonction à un réseau virtuel ?

Vous pouvez limiter le trafic **entrant** d’une application de fonction à un seul réseau virtuel à l’aide de [points de terminaison de service](./functions-networking-options.md#use-service-endpoints). Cette configuration permet toujours à l’application de fonction d’effectuer des appels sortants vers Internet.

Pour limiter complètement une fonction de sorte que tout le trafic transite par un réseau virtuel, vous pouvez utiliser des [points de terminaison privés](./functions-networking-options.md#private-endpoint-connections) avec l’intégration du réseau virtuel sortant ou un App Service Environment.

## <a name="how-can-i-access-resources-in-a-virtual-network-from-a-function-app"></a>Comment accéder aux ressources d’un réseau virtuel à partir d’une application de fonction ?

Vous pouvez accéder aux ressources d’un réseau virtuel à partir d’une fonction en cours d’exécution, à l’aide de l’intégration de réseau virtuel. Pour plus d’informations, consultez [Intégration de réseau virtuel](functions-networking-options.md#virtual-network-integration).

## <a name="how-do-i-access-resources-protected-by-service-endpoints"></a>Comment accéder aux ressources protégées par des points de terminaison de service ?

Avec l’intégration de réseau virtuel, vous pouvez accéder aux ressources sécurisées par des points de terminaison de service à partir d’une fonction en cours d’exécution. Pour plus d’informations, consultez [Intégration de réseau virtuel](functions-networking-options.md#virtual-network-integration).

## <a name="how-can-i-trigger-a-function-from-a-resource-in-a-virtual-network"></a>Comment déclencher une fonction à partir d’une ressource d’un réseau virtuel ?

Vous pouvez permettre aux déclencheurs HTTP d’être appelés à partir d’un réseau virtuel en utilisant des [points de terminaison de service](./functions-networking-options.md#use-service-endpoints) ou des [connexions de point de terminaison privé](./functions-networking-options.md#private-endpoint-connections). 

Vous pouvez également déclencher une fonction à partir d’autres ressources d’un réseau virtuel en déployant votre application de fonction dans un plan Premium, un plan App Service ou App Service Environment. Pour plus d’informations, consultez [Déclencheurs de réseau virtuel non-HTTP](./functions-networking-options.md#virtual-network-triggers-non-http).

## <a name="how-can-i-deploy-my-function-app-in-a-virtual-network"></a>Comment déployer mon application de fonction dans un réseau virtuel ?

Un déploiement dans App Service Environment est le seul moyen de créer une application de fonction qui se trouve entièrement dans un réseau virtuel. Pour plus d’informations sur l’utilisation d’un équilibreur de charge interne avec App Service Environment, commencez par lire l’article [Créer et utiliser un équilibreur de charge interne avec App Service Environment](../app-service/environment/create-ilb-ase.md).

Si vous avez seulement besoin d’un accès unidirectionnel aux ressources d’un réseau virtuel ou d’un isolement réseau moins complet, consultez [Vue d’ensemble des réseaux Azure Functions](functions-networking-options.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les réseaux et les fonctions : 

* [Tutoriel pour bien démarrer avec l’intégration de réseau virtuel](./functions-create-vnet.md)
* [En savoir plus sur les options de réseau dans Azure Functions](./functions-networking-options.md)
* [En savoir plus sur l’intégration de réseau virtuel avec App Service et Functions](../app-service/web-sites-integrate-with-vnet.md)
* [En savoir plus sur les réseaux virtuels dans Azure](../virtual-network/virtual-networks-overview.md)
* [Activer davantage de fonctionnalités de réseau et plus de contrôle avec Azure App Service Environment](../app-service/environment/intro.md)
