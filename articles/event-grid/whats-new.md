---
title: Nouveautés Notes de publication - Azure Event Grid
description: Découvrez les nouveautés d’Azure Event Grid, notamment les dernières notes de publication, les problèmes connus, les corrections de bogues, les fonctionnalités dépréciées et les modifications à venir.
ms.topic: overview
ms.date: 07/23/2020
ms.openlocfilehash: 7528cef7007704bbf434f16622707da8920eec2b
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350005"
---
# <a name="whats-new-in-azure-event-grid"></a>Nouveautés d’Azure Event Grid

>Soyez notifié de la disponibilité des mises à jour sur cette page en faisant un copier-coller de cette URL : `https://docs.microsoft.com/api/search/rss?search=%22Release+notes+-+Azure+Event+Grid%22&locale=en-us` dans votre lecteur de flux ![icône RSS](./media/whats-new/feed-icon-16x16.png).

Azure Event Grid bénéficie d’améliorations en continu. Pour vous informer des développements les plus récents, cet article détaille les thèmes suivants :

- Versions les plus récentes
- Problèmes connus
- Résolution des bogues
- Fonctionnalités dépréciées
- Modifications planifiées

## <a name="600-2020-06"></a>6.0.0 (2020-06)
- Ajout de la prise en charge de la nouvelle API de service en disponibilité générale (GA) version 2020-06-01.
- Les nouvelles fonctionnalités désormais en disponibilité générale sont les suivantes :
    - [Mappages d’entrée](input-mappings.md)
    - [Schéma d’entrée personnalisé](input-mappings.md)
    - [Schéma d’événement cloud V10](cloud-event-schema.md)
    - [Rubrique Service Bus en tant que destination](handler-service-bus.md)
    - [Fonction Azure en tant que destination](handler-functions.md)
    - [Traitement par lot de webhooks](./edge/delivery-output-batching.md)
    - [Webhook sécurisé (prise en charge d’Azure Active Directory)](secure-webhook-delivery.md)
    - [Filtrage IP](configure-firewall.md)
    - [Prise en charge du service de liaison privée](configure-private-endpoints.md)
    - [Schéma de remise des événements](event-schema.md)

## <a name="532-preview-2020-05"></a>5.3.2-preview (2020-05)
- Cette version comprend des correctifs de bogues supplémentaires pour améliorer la qualité.
- Comme la version 5.3.1-preview, cette version correspond à la version d’API 2020-04-01-Preview, qui comprend les nouvelles fonctionnalités suivantes : 
    - [Prise en charge du filtrage IP lors de la publication d’événements dans des domaines et des rubriques](configure-firewall.md)
    - [Rubriques de partenaire](./partner-events-overview.md)
    - [Rubriques système en tant que ressources suivies dans le portail Azure](system-topics.md)
    - [Remise d’événement avec une identité de service managé](managed-service-identity.md) 
    - [Prise en charge du service de liaison privée](configure-private-endpoints.md)

## <a name="531-preview-2020-04"></a>5.3.1-preview (2020-04)
- Cette version comprend divers correctifs de bogues pour améliorer la qualité.
- Comme la version 5.3.0-preview, cette version correspond à la version d’API 2020-04-01-Preview, qui comprend les nouvelles fonctionnalités suivantes : 
    - [Prise en charge du filtrage IP lors de la publication d’événements dans des domaines et des rubriques](configure-firewall.md)
    - [Rubriques de partenaire](./partner-events-overview.md)
    - [Rubriques système en tant que ressources suivies dans le portail Azure](system-topics.md)
    - [Remise d’événement avec une identité de service managé](managed-service-identity.md) 
    - [Prise en charge du service de liaison privée](configure-private-endpoints.md)

## <a name="530-preview-2020-03"></a>5.3.0-preview (2020-03)
- Nous introduisons de nouvelles fonctionnalités en plus des fonctionnalités déjà ajoutées à la version 5.2.0-preview. 
- Comme la version 5.2.0-preview, cette version correspond à la version d’API 2020-04-01-Preview.
- Elle ajoute la prise en charge des nouvelles fonctionnalités suivantes : 
    - [Prise en charge du filtrage IP lors de la publication d’événements dans des domaines et des rubriques](configure-firewall.md)
    - [Rubriques de partenaire](./partner-events-overview.md)
    - [Rubriques système en tant que ressources suivies dans le portail Azure](system-topics.md)
    - [Remise d’événement avec une identité de service managé](managed-service-identity.md) 
    - [Prise en charge du service de liaison privée](configure-private-endpoints.md)

## <a name="520-preview-2020-01"></a>5.2.0-preview (2020-01)
- Cette version correspond à la version d’API 2020-04-01-Preview.
- Elle ajoute la prise en charge de la nouvelle fonctionnalité suivante :
    - [Prise en charge du filtrage IP lors de la publication d’événements dans des domaines et des rubriques](configure-firewall.md)

## <a name="500-2019-05"></a>5.0.0 (2019-05)
- Cette version correspond à la version d’API `2019-06-01`.
- Elle ajoute la prise en charge des nouvelles fonctionnalités suivantes :
    * [Domaines](event-domains.md)
    * Pagination et filtre de recherche pour les opérations sur les listes des ressources. Pour obtenir un exemple, consultez [Rubriques - Lister par abonnement](/rest/api/eventgrid/version2020-04-01-preview/topics/listbysubscription).
    * [File d’attente Service Bus en tant que destination](handler-service-bus.md)
    * [Filtrage avancé](event-filtering.md#advanced-filtering)

## <a name="410-preview-2019-03"></a>4.1.0-preview (2019-03)
- Cette version correspond à la version d’API 2019-02-01-preview.
- Elle ajoute la prise en charge des nouvelles fonctionnalités suivantes :
    * Pagination et filtre de recherche pour les opérations sur les listes des ressources. Pour obtenir un exemple, consultez [Rubriques - Lister par abonnement](/rest/api/eventgrid/version2020-04-01-preview/topics/listbysubscription).
    * [Création/suppression manuelle de rubriques de domaines](how-to-event-domains.md)
    * [File d’attente Service Bus en tant que destination](handler-service-bus.md)

## <a name="400-2018-12"></a>4.0.0 (2018-12)
- Cette version correspond à la version d’API stable `2019-01-01`.
- Elle prend en charge la disponibilité générale (GA) des fonctionnalités suivantes liées aux abonnements aux événements :
    * [Destination des lettres mortes](manage-event-delivery.md)
    * [File d’attente de stockage Azure en tant que destination](handler-storage-queues.md)
    * [Azure Relay- Connexion hybride en tant que destination](handler-relay-hybrid-connections.md)
    * [Validation manuelle des négociations](webhook-event-delivery.md)
    * [Prise en charge des stratégies de nouvelle tentative](delivery-and-retry.md)
- Les fonctionnalités qui sont toujours en préversion (par exemple, les [domaines Event Grid](event-domains.md) ou la [prise en charge des filtres avancés](event-filtering.md#advanced-filtering)) sont toujours accessibles à l’aide de la version 3.0.1-preview du SDK.

## <a name="301-preview-2018-10"></a>3.0.1-preview (2018-10)
- Dépendance vis-à-vis de la [version 10.0.3 du package NuGet Newtonsoft](https://www.nuget.org/packages/Newtonsoft.Json/10.0.3).

## <a name="300-preview-2018-10"></a>3.0.0-preview (2018-10)
- Cette version est un SDK en préversion pour les nouvelles fonctionnalités introduites dans la version d’API 2018-09-15-preview. - Cette version prend en charge les éléments suivants :
    - [Domaine et rubriques de domaine](event-domains.md)
    - Présentation de la [date d’expiration de l’abonnement aux événements](concepts.md#event-subscription-expiration)
    - Présentation du [filtrage avancé](event-filtering.md#advanced-filtering) pour les abonnements aux événements
    - La version stable du SDK ciblant la version d’API `2018-01-01`, qui continue d’exister en tant que version 1.3.0

## <a name="next-steps"></a>Étapes suivantes