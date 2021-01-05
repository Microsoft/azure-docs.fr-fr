---
title: Concept d’Azure Event Grid
description: Détaille Azure Event Grid et ses concepts. Définit plusieurs composants clés de Event Grid.
ms.topic: conceptual
ms.date: 10/29/2020
ms.openlocfilehash: 6cfb8b3aaf16a0080b9864ce5198b8a7232e8bc8
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93075107"
---
# <a name="concepts-in-azure-event-grid"></a>Concepts utilisés dans Azure Event Grid

Cet article décrit les principaux concepts utilisés dans Azure Event Grid.

## <a name="events"></a>Événements

Un événement correspond à la plus petite quantité d’informations décrivant intégralement quelque chose qui s’est produit dans le système. Chaque événement possède des informations communes telles que la source de l’événement, l’heure à laquelle l’événement a eu lieu et un identificateur unique. Chaque événement comporte également des informations spécifiques qui se rapportent uniquement au type d’événement en question. Par exemple, un événement concernant un fichier en cours de création dans le Stockage Azure a des informations détaillées sur le fichier, notamment la valeur `lastTimeModified`. Ou bien, un événement Event Hubs contient l’URL du fichier Capture. 

Tout événement d’une taille jusqu’à 64 Ko est couvert par le contrat de niveau de service (SLA) en disponibilité générale. La prise en charge des événements d’une taille jusqu’à 1 Mo est actuellement en préversion. Les événements de plus de 64 Ko donnent lieu à une facturation par incréments de 64 Ko. 


Pour les propriétés qui sont envoyées dans un événement, consultez le [schéma d’événements Azure Event Grid](event-schema.md).

## <a name="publishers"></a>Serveurs de publication

Un éditeur est l’utilisateur ou organisation qui décide d’envoyer des événements à Event Grid. Microsoft publie des événements pour plusieurs services Azure. Vous pouvez publier des événements à partir de votre propre application. Les organisations qui hébergent des services en dehors d’Azure peuvent publier des événements par le biais d’Event Grid.

## <a name="event-sources"></a>Sources d’événement

La source d’un événement désigne l’endroit où l’événement se produit. Chaque source de l’événement est liée à un ou plusieurs types d’événements. Par exemple, le stockage Azure est la source d’événement des événements créés pour l’objet blob. IoT Hub est la source d’événements pour les événements créés par l’appareil. Votre application est la source d’événements pour les événements personnalisés que vous définissez. Les sources d’événements sont responsables de l’envoi des événements à Event Grid.

Pour plus d’informations sur l’implémentation d’une source Event Grid prise en charge, consultez [Sources d’événements dans Azure Event Grid](overview.md#event-sources).

## <a name="topics"></a>Rubriques

La rubrique Event Grid fournit un point de terminaison où la source envoie les événements. L’éditeur crée la rubrique Event Grid et décide si une source d’événements a besoin d’une ou plusieurs rubriques. Une rubrique est utilisée pour une collection d’événements connexes. Pour répondre à certains types d’événements, les abonnés décident des rubriques auxquelles ils souhaitent s’abonner.

Les **rubriques système** sont des rubriques intégrées fournies par des services Azure, tels que Stockage Azure, Azure Event Hubs et Azure Service Bus. Vous pouvez créer des rubriques système dans votre abonnement Azure et vous y abonner. Pour plus d’informations, consultez [Vue d’ensemble des rubriques système](system-topics.md). 

Les **rubriques personnalisées** sont des rubriques tierces et applicatives. Quand vous créez une rubrique personnalisée ou que l’accès à une rubrique personnalisée vous est accordé, celle-ci est visible dans votre abonnement. Pour plus d’informations, consultez [Rubriques personnalisées](custom-topics.md). Lorsque vous concevez votre application, vous pouvez choisir le nombre de rubriques à créer. Pour les solutions volumineuses, créez une rubrique personnalisée pour chaque catégorie d’événements associés. Par exemple, envisagez une application qui envoie des événements liés à la modification de comptes d’utilisateur et au traitement de commandes. Il est peu probable qu’un gestionnaire d’événements accepte les deux catégories d’événements. Créez deux rubriques personnalisées et laissez les gestionnaires d’événements s’abonner à celle qui les intéresse. Pour les solutions de petite taille, vous pouvez à la place envoyer tous les événements à une seule rubrique. Les abonnés à des événements peuvent filtrer les types d’événements qu’ils souhaitent.

Il existe un autre type de rubrique : **la rubrique de partenaire**. La fonctionnalité [Événements de partenaire](partner-events-overview.md) permet à un fournisseur SaaS tiers de publier des événements depuis ses services, pour les mettre à la disposition des consommateurs qui peuvent s’abonner à ces événements. Le fournisseur SaaS expose un type de rubrique, une **rubrique de partenaire** , que les abonnés utilisent pour consommer des événements. Elle fournit également un nouveau modèle de publication-abonnement clair, en séparant les préoccupations et la propriété des ressources qui sont utilisées par les éditeurs d’événements et les abonnés.

## <a name="event-subscriptions"></a>Abonnements à des événements

Un abonnement indique à Event Grid quels événements vous souhaitez recevoir sur une rubrique. Lors de la création de l’abonnement, vous spécifiez un point de terminaison pour la gestion de l’événement. Vous pouvez filtrer les événements qui sont envoyés au point de terminaison. Vous pouvez filtrer par type d’événement, ou modèle d’objet. Pour plus d’informations, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).

Pour obtenir des exemples de création d’abonnements, consultez :

* [Exemples d’interface de ligne de commande Azure pour Event Grid](cli-samples.md)
* [Exemples Azure PowerShell pour Event Grid](powershell-samples.md)
* [Modèles Azure Resource Manager pour Event Grid](template-samples.md)

Pour plus d’informations sur l’obtention de vos abonnements Event Grid actuels, consultez [Interroger les abonnements Event Grid](query-event-subscriptions.md).

## <a name="event-subscription-expiration"></a>Expiration de l’abonnement aux événements
L’abonnement aux événements expire automatiquement après cette date. Définissez une expiration pour les abonnements aux événements qui ne sont nécessaires que pour une durée limitée et que vous ne souhaitez pas avoir à nettoyer ; par exemple, si vous créez un abonnement aux événements pour tester un scénario. 

Pour un exemple de configuration d’une expiration, voir [S’abonner avec des filtres avancés](how-to-filter-events.md#subscribe-with-advanced-filters).

## <a name="event-handlers"></a>Gestionnaires d’événements

Pour Event Grid, un gestionnaire d’événements désigne l’endroit où l’événement est envoyé. Le gestionnaire effectue des actions supplémentaires pour traiter l’événement. Event Grid prend en charge plusieurs types de gestionnaires. Vous pouvez utiliser un service Azure pris en charge ou votre propre webhook comme gestionnaire. En fonction du type de gestionnaire, Event Grid suit différents mécanismes pour garantir la distribution de l’événement. Pour les gestionnaires d’événements HTTP Webhook, l’exécution de l’événement est retentée jusqu'à ce que le gestionnaire renvoie un code d’état de `200 – OK`. Dans le cas de la file d’attente de Stockage Azure, l’exécution des événements est retentée jusqu’à ce que le service de File d’attente traite correctement la transmission de type push du message dans la file d’attente.

Pour plus d’informations sur l’implémentation d’un gestionnaire Event Grid pris en charge, consultez [Gestionnaires d’événements dans Azure Event Grid](event-handlers.md).

## <a name="security"></a>Sécurité

Event Grid assure la sécurité lors de l’abonnement et de la publication de rubriques. Durant l’abonnement, vous devez disposer des autorisations appropriées sur la ressource ou la rubrique de grille d’événement. Lors de la publication, vous devez disposer d’un jeton SAS ou d’une clé d’authentification pour la rubrique. Pour en savoir plus, consultez la page [Sécurité et authentification pour Event Grid](security-authentication.md).

## <a name="event-delivery"></a>Remise d’événement

Si Event Grid ne peut pas confirmer qu’un événement a été reçu par le point de terminaison de l’abonné, il procède à une nouvelle distribution de l’événement. Pour plus d’informations, consultez la page [Distribution et nouvelle tentative de distribution de messages avec Event Grid](delivery-and-retry.md).

## <a name="batching"></a>Traitement par lot

Quand vous utilisez une rubrique personnalisée, les événements doivent toujours être publiés dans un tableau. Il peut s’agir d’un lot de un pour les scénarios à faible débit. Toutefois, dans les cas d’utilisation à volume élevé, il est recommandé de regrouper dans un lot plusieurs événements par publication afin d’améliorer l’efficacité. La taille maximale d’un lot est de 1 Mo. La taille maximale de chaque événement est toujours de 64 Ko (version en disponibilité générale) ou de 1 Mo (en préversion).

> [!NOTE]
> Tout événement d’une taille jusqu’à 64 Ko est couvert par le contrat de niveau de service (SLA) en disponibilité générale. La prise en charge des événements d’une taille jusqu’à 1 Mo est actuellement en préversion. Les événements de plus de 64 Ko donnent lieu à une facturation par incréments de 64 Ko. 

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Event Grid, consultez [À propos d’Event Grid](overview.md).
* Pour une prise en main rapide d’Event Grid, consultez [Créer et acheminer des événements personnalisés avec Azure Event Grid](custom-event-quickstart.md).
