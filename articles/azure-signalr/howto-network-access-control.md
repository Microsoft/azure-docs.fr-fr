---
title: Configurer le contrôle d’accès réseau
titleSuffix: Azure SignalR Service
description: Configurer le contrôle d’accès réseau pour votre service SignalR Azure.
services: signalr
author: ArchangelSDY
ms.service: signalr
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: dayshen
ms.openlocfilehash: 24e56736e0d033420f9aaf976b0fb7d9727c1a5b
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143216"
---
# <a name="configure-network-access-control"></a>Configurer le contrôle d’accès réseau

Azure SignalR Service vous permet de sécuriser et de contrôler le niveau d’accès à votre point de terminaison de service, en fonction du type de requête et du sous-ensemble de réseaux utilisés. Quand des règles de réseau sont configurées, seules les applications demandant des données sur l’ensemble de réseaux spécifié peuvent accéder à votre service SignalR Azure.

Azure SignalR Service possède un point de terminaison public accessible via Internet. Vous pouvez également créer [des points de terminaison privés pour votre service SignalR Azure](howto-private-endpoints.md). Le point de terminaison privé attribue une adresse IP privée de votre réseau virtuel au service SignalR Azure et sécurise tout le trafic entre votre réseau virtuel et le service SignalR Azure via une liaison privée. Le contrôle d’accès réseau d’Azure SignalR Service fournit un contrôle d’accès pour le point de terminaison public et les points de terminaison privés.

Si vous le souhaitez, vous pouvez choisir d’autoriser ou de refuser certains types de requêtes pour le point de terminaison public et pour chaque point de terminaison privé. Par exemple, vous pouvez bloquer toutes les [connexions de serveur](signalr-concept-internals.md#server-connections) provenant du point de terminaison public et vous assurer qu’elles proviennent uniquement d’un réseau virtuel spécifique.

Une application qui accède à un service SignalR Azure lorsque des règles de contrôle d’accès réseau sont en vigueur requiert toujours une autorisation appropriée pour la requête.

## <a name="scenario-a---no-public-traffic"></a>Scénario A : aucun trafic public

Pour refuser complètement tout le trafic public, vous devez d’abord configurer la règle de réseau public pour n’autoriser aucun type de requête. Ensuite, vous devez configurer des règles qui autorisent l’accès au trafic en provenance de réseaux virtuels spécifiques. Cette configuration vous permet de créer une limite de réseau sécurisée pour vos applications.

## <a name="scenario-b---only-client-connections-from-public-network"></a>Scénario B : uniquement les connexions client d’un réseau public

Dans ce scénario, vous pouvez configurer la règle de réseau public pour autoriser uniquement les [connexions client](signalr-concept-internals.md#client-connections) du réseau public. Vous pouvez ensuite configurer des règles de réseau privé pour autoriser d’autres types de requêtes provenant d’un réseau virtuel spécifique. Cette configuration masque vos serveurs d’applications du réseau public et établit des connexions sécurisées entre vos serveurs d’applications et Azure SignalR Service.

## <a name="managing-network-access-control"></a>Gestion du contrôle d’accès réseau

Vous pouvez gérer le contrôle d’accès réseau pour Azure SignalR Service via le portail Azure.

### <a name="azure-portal"></a>Portail Azure

1. Accédez au service SignalR Azure que vous souhaitez sécuriser.

1. Cliquez sur le menu des paramètres appelé **Contrôle d’accès réseau**.

    ![Liste de contrôle d’accès réseau sur le portail](media/howto-network-access-control/portal.png)

1. Pour modifier l’action par défaut, faites basculer le bouton **Autoriser/refuser**.

    > [!TIP]
    > L’action par défaut est l’action que nous entreprenons lorsqu’il n’existe aucune correspondance de règle avec la liste de contrôle d’accès (ACL, access-control list). Par exemple, si l’action par défaut est **Refuser**, les types de demandes non explicitement approuvés ci-dessous sont refusés.

1. Pour modifier une règle de réseau public, sélectionnez les types de requêtes autorisés sous **Réseau public**.

    ![Modifier l’ACL du réseau public sur le portail ](media/howto-network-access-control/portal-public-network.png)

1. Pour modifier des règles de réseau des points de terminaison privés, sélectionnez les types de requêtes autorisés dans chaque ligne sous **Connexions de points de terminaison privés**.

    ![Modifier l’ACL des points de terminaison privés sur le portail ](media/howto-network-access-control/portal-private-endpoint.png)

1. Cliquez sur **Enregistrer** pour enregistrer les changements.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [Azure Private Link](../private-link/private-link-overview.md).