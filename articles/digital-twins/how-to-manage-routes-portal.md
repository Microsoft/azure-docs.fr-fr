---
title: Gérer les points de terminaison et les itinéraires (portail)
titleSuffix: Azure Digital Twins
description: Découvrez comment configurer et gérer des points de terminaison et des itinéraires d’événements pour les données d’Azure Digital Twins à l’aide du portail Azure.
author: baanders
ms.author: baanders
ms.date: 7/22/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 083d868f2d2652be9480227c29dfb289564056d6
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94533784"
---
# <a name="manage-endpoints-and-routes-in-azure-digital-twins-portal"></a>Gérer les points de terminaison et les itinéraires dans Azure Digital Twins (portail)

[!INCLUDE [digital-twins-route-selector.md](../../includes/digital-twins-route-selector.md)]

Dans Azure Digital Twins, vous pouvez acheminer les [notifications d’événements](how-to-interpret-event-data.md) vers des services en aval ou des ressources de calcul connectées. Pour ce faire, vous devez d’abord configurer des **points de terminaison** qui peuvent recevoir les événements. Vous pouvez ensuite créer [**des itinéraires d’événements**](concepts-route-events.md) qui spécifient quels événements générés par Azure Digital Twins sont remis aux points de terminaison.

Cet article vous guide tout au long du processus de création de points de terminaison et d’itinéraires à l’aide du [portail Azure](https://portal.azure.com).

Vous pouvez également gérer les points de terminaison et les routes à l’aide des [API de routes d’événements](/rest/api/digital-twins/dataplane/eventroutes), des kits [SDK](how-to-use-apis-sdks.md#overview-data-plane-apis) ou de l’[interface CLI Azure Digital Twins](how-to-use-cli.md). Pour obtenir une version de cet article qui utilise ces mécanismes plutôt que le portail, consultez [*Guide pratique : Gérer les points de terminaison et les itinéraires (API et CLI)*](how-to-manage-routes-apis-cli.md).

## <a name="prerequisites"></a>Prérequis

* Vous avez besoin d’un **compte Azure** (vous pouvez en définir un gratuitement [ici](https://azure.microsoft.com/free/?WT.mc_id=A261C142F))
* Vous aurez besoin d’une **instance Azure Digital Twins** dans votre abonnement Azure. Si vous n’avez pas d’instance, vous pouvez en créer une en suivant les étapes décrites du [*Tutoriel : Configurer une instance et l’authentification*](how-to-set-up-instance-portal.md). Utilisez les valeurs suivantes du programme d’installation pour les utiliser plus loin dans cet article :
    - Nom de l’instance
    - Resource group

Vous trouverez ces informations dans le [portail Azure](https://portal.azure.com) après avoir configuré votre instance. Connectez-vous au portail et recherchez le nom de votre instance dans la barre de recherche du portail.
 
:::image type="content" source="media/how-to-manage-routes-portal/search-field-portal.png" alt-text="Capture d’écran de la barre de recherche du portail Azure.":::

Sélectionnez votre instance parmi les résultats pour afficher la page de détails la concernant :

:::image type="content" source="media/how-to-manage-routes-portal/instance-details.png" alt-text="Capture d’écran des détails de l’instance ADT." border="false":::

## <a name="create-an-endpoint-for-azure-digital-twins"></a>Créer un point de terminaison pour Azure Digital Twins

Voici les types de points de terminaison pris en charge que vous pouvez créer pour votre instance :
* [Event Grid](../event-grid/overview.md) 
* [Hubs d'événements](../event-hubs/event-hubs-about.md)
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)

Pour plus d’informations sur les différents points de terminaison, consultez [*Choisir entre les différents services de messagerie Azure*](../event-grid/compare-messaging-services.md).

Pour lier un point de terminaison à Azure Digital Twins, l’Event Hub, une rubrique Event Grid ou le Service Bus que vous utilisez pour le point de terminaison doivent pré-exister. 

### <a name="create-an-event-grid-endpoint"></a>Créer un point de terminaison Event Grid

**Condition préalable** : Créez une rubrique Event Grid en suivant les étapes décrites dans [la section *Créer une rubrique personnalisée*](../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic) du démarrage rapide *Événements personnalisés* Event Grid.

Une fois que vous avez créé la rubrique, vous pouvez la lier à Azure Digital Twins à partir de la page de votre instance Azure Digital Twins dans le [portail Azure](https://portal.azure.com) (vous pouvez trouver l’instance en entrant son nom dans la barre de recherche du portail).

Dans le menu de l’instance, sélectionnez _Points de terminaison_. Ensuite, à partir de la page *Points de terminaison* qui suit, sélectionnez *+ Créer un point de terminaison*. 

Dans la page *Créer un point de terminaison* qui s’ouvre, vous pouvez créer un point de terminaison de type _Event Grid_ en sélectionnant la case d’option correspondante. Renseignez les autres détails : entrez un nom pour votre point de terminaison dans le champ _Nom_, choisissez votre _abonnement_ dans la liste déroulante, puis choisissez votre _rubrique Event Grid_ précréée dans la troisième liste déroulante.

Ensuite, créez votre point de terminaison en appuyant sur _Enregistrer_.

:::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-event-grid.png" alt-text="Capture d’écran de la création d’un point de terminaison de type Event Grid.":::

Vous pouvez vérifier que le point de terminaison est correctement créé en sélectionnant l’icône de notification dans la barre supérieure du portail Azure : 

:::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-notifications.png" alt-text="Capture d’écran de la notification pour vérifier la création du point de terminaison." border="false":::

Vous pouvez également afficher le point de terminaison qui a été créé dans la page *Points de terminaison* de votre instance Azure Digital Twins.

Si la création du point de terminaison échoue, observez le message d’erreur et réessayez après quelques minutes.

La rubrique Event Grid est alors disponible en tant que point de terminaison dans Azure Digital Twins, sous le nom spécifié dans le champ _Nom_. Vous utiliserez généralement ce nom en tant que cible d’un **itinéraire d’événement** que vous allez créer [plus loin dans cet article](#create-an-event-route).

### <a name="create-an-event-hubs-endpoint"></a>Créer un point de terminaison Event Hubs

**Prérequis** : 
* Vous avez besoin d’un _espace de noms Event Hubs_ et d’un _Event Hub_. Pour créer ces deux éléments, suivez les étapes du démarrage rapide [*Créer un Event Hub*](../event-hubs/event-hubs-create.md) d’Event Hubs.
* Vous aurez besoin d’une _règle d’autorisation_. Pour ce faire, reportez-vous à l’article [*Autorisation de l’accès aux ressources Event Hubs à l’aide de signatures d’accès partagé*](../event-hubs/authorize-access-shared-access-signature.md) d’Event Hubs.

Accédez à la page de détails de votre instance Azure Digital Twins dans le [portail Azure](https://portal.azure.com) (vous pouvez la trouver en entrant son nom dans la barre de recherche du portail).

Dans le menu de l’instance, sélectionnez _Points de terminaison_. Ensuite, à partir de la page *Points de terminaison* qui suit, sélectionnez *+ Créer un point de terminaison*. 

Dans la page *Créer un point de terminaison* qui s’ouvre, vous pouvez créer un point de terminaison de type _Event Hub_ en sélectionnant la case d’option correspondante. Entrez un nom pour votre point de terminaison dans le champ _Nom_. Sélectionnez ensuite votre _abonnement_, ainsi que l’_espace de noms Event Hub_, l’_Event Hub_ et la _règle d’autorisation_ précréés dans les listes déroulantes respectives.

Ensuite, créez votre point de terminaison en appuyant sur _Enregistrer_.

:::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-event-hub.png" alt-text="Capture d’écran de la création d’un point de terminaison de type Event Hubs.":::

Vous pouvez vérifier que le point de terminaison est correctement créé en sélectionnant l’icône de notification dans la barre supérieure du portail Azure. 

Si la création du point de terminaison échoue, observez le message d’erreur et réessayez après quelques minutes.

L’Event Hub est alors disponible en tant que point de terminaison dans Azure Digital Twins, sous le nom spécifié dans le champ _Nom_. Vous utiliserez généralement ce nom en tant que cible d’un **itinéraire d’événement** que vous allez créer [plus loin dans cet article](#create-an-event-route).

### <a name="create-a-service-bus-endpoint"></a>Créer un point de terminaison Service Bus

**Prérequis** : 
* Vous avez besoin d’un _espace de noms Service Bus_ et d’une _rubrique Service Bus_. Pour créer ces deux éléments, suivez les étapes décrites dans le démarrage rapide [*Créer des rubriques et des abonnements*](../service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md) de Service Bus. Vous n’avez pas besoin de terminer la section [*Créer des abonnements à la rubrique*](../service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md#create-subscriptions-to-the-topic).
* Vous aurez besoin d’une _règle d’autorisation_. Pour ce faire, reportez-vous à l’article [*Authentification et autorisation*](../service-bus-messaging/service-bus-authentication-and-authorization.md#shared-access-signature) de Service Bus.

Accédez à la page de détails de votre instance Azure Digital Twins dans le [portail Azure](https://portal.azure.com) (vous pouvez la trouver en entrant son nom dans la barre de recherche du portail).

Dans le menu de l’instance, sélectionnez _Points de terminaison_. Ensuite, à partir de la page *Points de terminaison* qui suit, sélectionnez *+ Créer un point de terminaison*. 

Dans la page *Créer un point de terminaison* qui s’ouvre, vous pouvez créer un point de terminaison de type _Service Bus_ en sélectionnant la case d’option correspondante. Entrez un nom pour votre point de terminaison dans le champ _Nom_. Sélectionnez ensuite votre _abonnement_, ainsi que l’_espace de noms Service Bus_, la _rubrique Service Bus_ et la _règle d’autorisation_ précréés dans les listes déroulantes respectives.

Ensuite, créez votre point de terminaison en appuyant sur _Enregistrer_.

:::image type="content" source="media/how-to-manage-routes-portal/create-endpoint-service-bus.png" alt-text="Capture d’écran de la création d’un point de terminaison de type Service Bus.":::

Vous pouvez vérifier que le point de terminaison est correctement créé en sélectionnant l’icône de notification dans la barre supérieure du portail Azure. 

Si la création du point de terminaison échoue, observez le message d’erreur et réessayez après quelques minutes.

La rubrique Service Bus est ensuite disponible en tant que point de terminaison dans Azure Digital Twins, sous le nom spécifié dans le champ _Nom_. Vous utiliserez généralement ce nom en tant que cible d’un **itinéraire d’événement** que vous allez créer [plus loin dans cet article](#create-an-event-route).

### <a name="create-an-endpoint-with-dead-lettering"></a>Créer un point de terminaison avec mise en file d’attente de lettres mortes

Lorsqu’un point de terminaison ne peut pas remettre un événement dans un laps de temps donné ou après avoir essayé de remettre l’événement un certain nombre de fois, il peut envoyer l’événement non remis à un compte de stockage. Ce processus est appelé **mise en file d’attente de lettres mortes**.

Pour créer un point de terminaison avec mise en file d’attente de lettres mortes, il est préférable d’utiliser les [API ARM](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate) pour créer votre point de terminaison, plutôt que le portail Azure.

Pour obtenir des instructions sur la façon de procéder avec les API, consultez la version basée sur [*l’interface CLI et les API*](how-to-manage-routes-apis-cli.md#create-an-endpoint-with-dead-lettering) de cet article.

## <a name="create-an-event-route"></a>Création d’un itinéraire d’événements

Pour envoyer concrètement des données d’Azure Digital Twins à un point de terminaison, vous devez définir un **itinéraire d’événement**. Ces itinéraires permettent aux développeurs d’associer le flux d’événements au sein du système et aux services en aval. Pour en savoir plus sur les itinéraires d’événements, consultez [*Concepts : routage des événements Azure Digital Twins*](concepts-route-events.md).

**Condition préalable** : Vous devez créer des points de terminaison comme décrit précédemment dans cet article avant de pouvoir passer à la création d’un itinéraire. Une fois que vos points de terminaison sont configurés, vous pouvez passer à la création d’un itinéraire d’événements.

>[!NOTE]
>Si vous avez récemment déployé vos points de terminaison, vérifiez que le déploiement est terminé **avant** de tenter de les utiliser pour un nouvel itinéraire d’événements. Si vous ne parvenez pas à configurer l’itinéraire parce que les points de terminaison ne sont pas prêts, patientez quelques minutes, puis réessayez.

### <a name="creation-steps-with-the-azure-portal"></a>Étapes de création avec le portail Azure

Une définition d’itinéraire d’événement contient les éléments suivants :
* nom de l’itinéraire que vous souhaitez utiliser ;
* nom du point de terminaison que vous souhaitez utiliser ;
* filtre définissant les événements envoyés au point de terminaison.
    - Pour désactiver l’itinéraire afin qu’aucun événement ne soit envoyé, utilisez une valeur de filtre `false`.
    - Pour activer un itinéraire qui n’a pas de filtrage spécifique, utilisez une valeur de filtre `true`.
    - Pour plus d’informations sur les autres types de filtre, consultez la section [*Filtrer les événements*](#filter-events) ci-dessous.

Un seul itinéraire peut permettre la sélection de plusieurs notifications et types d’événements.

Pour créer un itinéraire d’événement, accédez à la page de détails de votre instance Azure Digital Twins dans le [portail Azure](https://portal.azure.com) (vous pouvez rechercher l’instance en entrant son nom dans la barre de recherche du portail).

Dans le menu de l’instance, sélectionnez _Itinéraires d’événements_. Ensuite, à partir de la page *Itinéraires d’événements* qui suit, sélectionnez *+ Créer un itinéraire d’événements*. 

Dans la page *Créer un itinéraire d’événement* qui s’ouvre, choisissez au minimum :
* Un nom pour votre itinéraire dans le champ _Nom_
* Le _point de terminaison_ que vous souhaitez utiliser pour créer l’itinéraire 

Pour que l’itinéraire soit activé, vous devez également **ajouter un filtre d’itinéraires d’événements** `true` au minimum. (Laisser la valeur `false` par défaut créera l’itinéraire, mais aucun événement ne lui sera envoyé.) Pour ce faire, appuyez sur le bouton bascule de l’_éditeur avancé_ pour l’activer, puis écrivez `true` dans la zone *Filtre*.

:::image type="content" source="media/how-to-manage-routes-portal/create-event-route-no-filter.png" alt-text="Capture d’écran de la création de l’itinéraire d’événement pour votre instance." lightbox="media/how-to-manage-routes-portal/create-event-route-no-filter.png":::

Lorsque vous avez terminé, cliquez sur le bouton _Enregistrer_ pour créer votre itinéraire d’événement.

## <a name="filter-events"></a>Filtrer les événements

Comme décrit ci-dessus, les itinéraires ont un champ **Filtre**. Si la valeur de filtre sur votre itinéraire est `false`, aucun événement n’est envoyé à votre point de terminaison. 

Après avoir activé le filtre minimal `true`, les points de terminaison recevront une variété d’événements d’Azure Digital Twins :
* télémétrie déclenchée par des [jumeaux numériques](concepts-twins-graph.md) à l’aide de l’API de service d’Azure Digital Twins ;
* notifications de changement de propriété de jumeau, déclenchées par des modifications des propriétés d’un jumeau dans l’instance Azure Digital Twins ;
* événements de cycle de vie déclenchés lors de la création ou de la suppression de jumeaux ou de relations ;

Vous pouvez restreindre les types d’événements envoyés en définissant un filtre plus spécifique.

Pour ajouter un filtre d’événements lors de la création d’un itinéraire d’événement, utilisez la section _Ajouter un filtre d’itinéraires d’événements_ de la page *Créer un itinéraire d’événement*. 

Vous pouvez sélectionner l’une des options de filtre courantes de base ou utiliser les options de filtre avancées pour écrire vos propres filtres personnalisés.

#### <a name="use-the-basic-filters"></a>Utiliser les filtres de base

Pour utiliser les filtres de base, développez l’option _Types d’événements_, puis cochez les cases correspondant aux événements que vous souhaitez envoyer à votre point de terminaison. 

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-basic-1.png" alt-text="Capture d’écran de la création d’un itinéraire d’événement avec un filtre de base. Sélection des cases à cocher correspondant aux événements.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Cette opération remplit automatiquement la zone de texte de filtre avec le texte du filtre que vous avez sélectionné :

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-basic-2.png" alt-text="Capture d’écran de la création d’un itinéraire d’événement avec un filtre de base. Indication du texte de filtre rempli automatiquement après la sélection des événements.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

#### <a name="use-the-advanced-filters"></a>Utiliser les filtres avancés

Vous pouvez également utiliser l’option de filtre avancée pour écrire vos propres filtres personnalisés.

Pour créer un itinéraire d’événement avec des options de filtre avancées, appuyez sur le bouton bascule de l’_éditeur avancé_ pour l’activer. Vous pouvez ensuite écrire vos propres filtres d’événement dans la zone *Filtre* :

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes-portal/create-event-route-filter-advanced.png" alt-text="Capture d’écran de la création d’un itinéraire d’événement avec un filtre avancé.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Voici les filtres d’itinéraire pris en charge. Les détails de la colonne *Schéma de texte du filtre* sont le texte qui peut être entré dans la zone de filtre.

[!INCLUDE [digital-twins-route-filters](../../includes/digital-twins-route-filters.md)]

[!INCLUDE [digital-twins-route-metrics](../../includes/digital-twins-route-metrics.md)]

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les différents types de messages d’événements que vous pouvez recevoir :
* [*Guide pratique pour interpréter des données d’événement*](how-to-interpret-event-data.md)
