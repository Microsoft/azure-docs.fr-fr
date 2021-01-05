---
title: Azure Event Grid - Événements de partenaire
description: Envoyez des événements de partenaires SaaS et PaaS Event Grid tiers directement à des services Azure avec Azure Event Grid.
ms.topic: conceptual
ms.date: 11/10/2020
ms.openlocfilehash: 31a5fe611871eb4734b6a68e3818592028ebc75c
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94506144"
---
# <a name="partner-events-in-azure-event-grid-preview"></a>Événements de partenaire dans Azure Event Grid (préversion)
La fonctionnalité **Événements de partenaire** permet à un fournisseur SaaS tiers de publier des événements depuis ses services afin que les consommateurs puissent s’abonner à ces événements. Cette fonctionnalité offre une expérience interne aux sources d’événements tierces, en exposant un type de [rubrique](concepts.md#topics) et une **rubrique de partenaire**. Les abonnés créent des abonnements à cette rubrique pour consommer des événements. Elle fournit également un nouveau modèle clair de publication-abonnement, en séparant les préoccupations et la propriété des ressources qui sont utilisées par les éditeurs d’événements et les abonnés.

> [!NOTE]
> Si vous ne connaissez pas Event Grid, consultez la [vue d’ensemble](overview.md), les [concepts](concepts.md) et les [gestionnaires d’événements](event-handlers.md).

## <a name="what-is-partner-events-to-a-publisher"></a>Que représente Événements de partenaire pour un éditeur ?
Pour un éditeur d’événements, la fonctionnalité Événements de partenaire permet d’effectuer les tâches suivantes :

- Intégrer ses sources d’événements à Event Grid
- Créer un espace de noms (point de terminaison) sur lequel il peut publier des événements
- Créer des rubriques de partenaire dans Azure, que les abonnés possèdent et utilisent pour consommer des événements

## <a name="what-is-partner-events-to-a-subscriber"></a>Que représente Événements de partenaire pour un abonné ?
Pour un abonné, la fonctionnalité Événements de partenaire permet de créer des rubriques de partenaire dans Azure, afin de consommer des événements émanant de sources d’événements tierces. La consommation d’événements est obtenue en créant des abonnements d’événements qui envoient (push) des événements au gestionnaire d’événements d’un abonné.

## <a name="why-should-i-use-partner-events"></a>Pourquoi devrais-je utiliser Événements de partenaire ?
Il peut être intéressant pour vous d’utiliser Événements de partenaire si une ou plusieurs des exigences suivantes sont les vôtres.

### <a name="for-publishers"></a>Pour les éditeurs

- Vous souhaitez disposer d’un mécanisme permettant de rendre vos événements disponibles sur Azure. Vos utilisateurs peuvent filtrer et acheminer ces événements par l’intermédiaire des rubriques de partenaire et des abonnements aux événements qu’ils possèdent et gèrent. Vous pouvez utiliser d’autres moyens d’intégration, comme les [rubriques](custom-topics.md) et les [domaines](event-domains.md). Par contre, ils ne permettent pas la séparation franche de la propriété, de la gestion et de la facturation (rubriques de partenaire) des ressources entre les éditeurs et les abonnés. Cette approche offre également une expérience utilisateur plus intuitive qui facilite la découverte et l’utilisation des rubriques de partenaire.
- Vous désirez publier des événements sur un seul point de terminaison, le point de terminaison de l’espace de noms. Et pouvoir également filtrer ces événements afin qu’un seul sous-ensemble soit disponible. 
- Vous voulez bénéficier d’une visibilité sur les métriques liées aux événements publiés.
- Vous souhaitez utiliser le schéma [Cloud Events 1.0](https://cloudevents.io/) pour vos événements.

### <a name="for-subscribers"></a>Pour les abonnés

- Vous souhaitez vous abonner aux événements d’[éditeurs tiers](#available-third-party-event-publishers) et gérer ces événements à l’aide de gestionnaires d’événements sur Azure ou ailleurs.
- Vous désirez tirer parti de l’ensemble complet des fonctionnalités de routage et des [gestionnaires d’événements/destinations](overview.md#event-handlers) permettant de traiter les événements provenant de sources tierces. 
- Vous voulez implémenter des architectures faiblement couplées, dans lesquelles votre abonné/gestionnaire d’événements ignore l’existence du courtier de messages utilisé. 
- Vous avez besoin d’un mécanisme de remise par envoi (push) résilient, doté d’une prise en charge des nouvelles tentatives d’envoi et d’une sémantique de type au moins une fois.
- Vous souhaitez utiliser le schéma [Cloud Events 1.0](https://cloudevents.io/) pour vos événements. 


## <a name="available-third-party-event-publishers"></a>Éditeurs d’événements tiers disponibles
Un éditeur d’événements tiers doit suivre un [processus d’intégration](partner-onboarding-overview.md) avant qu’un abonné puisse commencer à consommer ses événements. 

Si vous êtes un abonné et que vous souhaitez qu’un service tiers expose ses événements via Event Grid, 

### <a name="auth0"></a>Auth0
**Auth0** est le premier éditeur partenaire disponible. Vous pouvez créer une [rubrique de partenaire Auth0](auth0-overview.md) pour connecter vos comptes Auth0 et Azure. Cette intégration vous permet de traiter, journaliser et superviser les événements Auth0 en temps réel. Pour l’essayer, consultez [Intégrer Azure Event Grid à Auto0](auth0-how-to.md)

Si vous souhaitez qu’un service tiers expose ses événements par le biais d’Event Grid, soumettez cette idée sur le [portail User Voice](https://feedback.azure.com/forums/909934-azure-event-grid).
 
## <a name="resources-managed-by-event-publishers"></a>Ressources gérées par les éditeurs d’événements
Les éditeurs d’événements créent et gèrent les ressources suivantes :

### <a name="partner-registration"></a>Inscription de partenaire
Une inscription contient des informations générales liées à un éditeur. Elle définit un type de rubrique de partenaire qui s’affiche dans le portail Azure sous forme d’option lorsque les utilisateurs essaient de créer une rubrique de partenaire. Un éditeur peut exposer plusieurs types de rubriques partenaires pour répondre aux besoins de ses abonnés. Autrement dit, un éditeur peut créer des inscriptions distinctes (types de rubriques partenaires) pour des événements provenant de différents services. Par exemple, pour le service des ressources humaines (RH), l’éditeur peut définir une rubrique de partenaire pour des événements, tels que employé intégré, employé promu et employé parti de l’entreprise. 

Gardez à l’esprit les points suivants :

- Seuls les enregistrements de partenaires approuvés par Azure sont visibles. 
- Les inscriptions sont globales. En d’autres termes, elles ne sont pas associées à une région Azure particulière.
- Une inscription est une ressource facultative. Toutefois, nous vous recommandons (en tant qu’éditeur) de créer une inscription. Elle permet aux utilisateurs de découvrir vos rubriques sur la page **Créer une rubrique de partenaire** dans le [portail Azure](https://portal.azure.com/#create/Microsoft.EventGridPartnerTopic). Ensuite, l’utilisateur peut sélectionner des types d’événements (par exemple, employé intégré, employé parti, etc.) lors de la création d’abonnements aux événements.

### <a name="namespace"></a>Espace de noms
À l’instar des [rubriques personnalisées](custom-topics.md) et des [domaines](event-domains.md), un espace de noms de partenaire constitue un point de terminaison régional pour publier des événements. C’est par l’intermédiaire des espaces de noms que les éditeurs créent et gèrent des canaux d’événements. Un espace de noms fonctionne également en tant que ressource de conteneur pour les canaux d’événements.

### <a name="event-channels"></a>Canaux d’événements
Un canal d’événement est une ressource mise en miroir sur une rubrique de partenaire. Lorsqu’un éditeur crée un canal d’événement dans l’abonnement Azure de l’éditeur, il crée également une rubrique de partenaire sous l’abonnement Azure d’un abonné. Les opérations effectuées sur un canal d’événement (à l’exception de GET) sont appliquées à la rubrique de partenaire correspondante de l’abonné, y compris la suppression. Toutefois, seules les rubriques de partenaire sont les types de ressources sur lesquels les abonnements et la remise d’événement peuvent être configurés.

## <a name="resources-managed-by-subscribers"></a>Ressources gérées par les abonnés 
Les abonnés peuvent utiliser les rubriques de partenaire définies par un éditeur, c’est le seul type de ressource qu’ils voient et gèrent. Dès lors qu’une rubrique de partenaire est créée, un utilisateur abonné peut créer des abonnements à des événements en définissant des règles de filtre pour des [gestionnaires d’événements/destinations](overview.md#event-handlers). Pour les abonnés, une rubrique de partenaire et ses abonnements aux événements associés fournissent les mêmes fonctionnalités complètes que les [rubriques personnalisées](custom-topics.md) ; leur(s) abonnement(s) associé(s) présentent une différence notable : les rubriques de partenaire prennent uniquement en charge le schéma [Cloud Events 1.0](cloudevents-schema.md) qui fournit un ensemble plus riche de fonctionnalités que les autres schémas pris en charge.

L’illustration suivante montre le flux des opérations du plan de contrôle.

:::image type="content" source="./media/partner-events-overview/partner-control-plane-flow.png" alt-text="Événements de partenaire – Flux du plan de contrôle":::

1. L’éditeur crée une **inscription de partenaire**. Les inscriptions de partenaires sont mondiales. Autrement dit, elles ne sont pas associées à une région Azure particulière. Cette étape est facultative.
1. L’éditeur crée un **espace de noms de partenaire** dans une région spécifique.
1. Lorsque « Abonné 1 » tente de créer une rubrique de partenaire, un **canal d’événement**, « Canal d’événement 1 », est d’abord créé dans l’abonnement Azure de l’éditeur.
1. Ensuite, une **rubrique de partenaire**, « Rubrique de partenaire 1 », est créée dans l’abonnement Azure de l’abonné. L’abonné doit activer la rubrique de partenaire. 
1. Abonné 1 crée un **abonnement Azure Logic Apps** à Rubrique de partenaire 1.
1. Abonné 1 crée un **abonnement Stockage Blob Azure** à Rubrique de partenaire 1. 
1. Lorsque « Abonné 2 » tente de créer une rubrique de partenaire, un autre **canal d’événement**, « Canal d’événement 2 », est d’abord créé dans l’abonnement Azure de l’éditeur. 
1. Ensuite, la **rubrique de partenaire**, « Rubrique de partenaire 2 », est créée dans l’abonnement Azure de l’abonné. L’abonné doit activer la rubrique de partenaire. 
1. Abonné 2 crée un **abonnement Azure Functions** à Rubrique de partenaire 2. 

## <a name="pricing"></a>Tarifs
Les rubriques de partenaire sont facturées au nombre d’opérations effectuées lors de l’utilisation d’Event Grid. Pour plus d’informations sur tous les types d’opérations utilisés comme base de facturation, et les tarifs détaillés, consultez [Tarifs Event Grid](https://azure.microsoft.com/pricing/details/event-grid/).

## <a name="limits"></a>limites
Pour plus d’informations sur les limites en place appliquées aux rubriques de partenaire, consultez [Limites du service Event Grid](../azure-resource-manager/management/azure-subscription-service-limits.md#event-grid-limits).


## <a name="next-steps"></a>Étapes suivantes

- [Rubrique de partenaire Auth0](auth0-overview.md)
- [Comment utiliser la rubrique de partenaire Auth0](auth0-how-to.md)
- [Devenir partenaire Event Grid](partner-onboarding-overview.md)