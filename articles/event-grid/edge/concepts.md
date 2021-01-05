---
title: Concepts – Azure Event Grid IoT Edge | Microsoft Docs
description: Concepts d’Event Grid sur IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: b3babfe93012fae15e79362ba34f3f48856bc833
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171735"
---
# <a name="event-grid-concepts"></a>Concepts d’Event Grid

Cet article décrit les principaux concepts utilisés dans Azure Event Grid.

## <a name="events"></a>Événements

Un événement correspond à la plus petite quantité d’informations décrivant intégralement quelque chose qui s’est produit dans le système. Chaque événement possède des informations communes telles que la source de l’événement, l’heure à laquelle l’événement a eu lieu et un identificateur unique. Chaque événement comporte également des informations spécifiques qui se rapportent uniquement au type d’événement en question. La prise en charge des événements d’une taille jusqu’à 1 Mo est actuellement en préversion.

Pour connaître les propriétés incluses dans un événement, voir [Schéma d’événements Azure Event Grid](event-schemas.md).

## <a name="publishers"></a>Serveurs de publication

Un éditeur est l’utilisateur ou organisation qui décide d’envoyer des événements à Event Grid. Vous pouvez publier des événements à partir de votre propre application.

## <a name="event-sources"></a>Sources d’événement

La source d’un événement désigne l’endroit où l’événement se produit. Chaque source de l’événement est liée à un ou plusieurs types d’événements. Par exemple, le stockage Azure est la source d’événement des événements créés pour l’objet blob. Votre application est la source d’événements pour les événements personnalisés que vous définissez. Les sources d’événements sont responsables de l’envoi des événements à Event Grid.

## <a name="topics"></a>Rubriques

La rubrique Event Grid fournit un point de terminaison où la source envoie les événements. L’éditeur crée la rubrique Event Grid et décide si une source d’événements a besoin d’une ou plusieurs rubriques. Une rubrique est utilisée pour une collection d’événements connexes. Pour répondre à certains types d’événements, les abonnés décident des rubriques auxquelles ils souhaitent s’abonner.

Lorsque vous concevez votre application, vous pouvez choisir le nombre de rubriques à créer. Pour les solutions volumineuses, créez une rubrique personnalisée pour chaque catégorie d’événements associés. Par exemple, envisagez une application qui envoie des événements liés à la modification de comptes d’utilisateur et au traitement de commandes. Il est peu probable qu’un gestionnaire d’événements accepte les deux catégories d’événements. Créez deux rubriques personnalisées et laissez les gestionnaires d’événements s’abonner à celle qui les intéresse. Pour les solutions de petite taille, vous pouvez à la place envoyer tous les événements à une seule rubrique. Les abonnés à des événements peuvent filtrer les types d’événements qu’ils souhaitent.

Pour savoir comment gérer les rubriques dans Event Grid, voir [Documentation sur l’API REST](api.md).

## <a name="event-subscriptions"></a>Abonnements à des événements

Un abonnement indique à Event Grid quels événements vous souhaitez recevoir sur une rubrique. Lors de la création de l’abonnement, vous spécifiez un point de terminaison pour la gestion de l’événement. Vous pouvez filtrer les événements qui sont envoyés au point de terminaison. 

Pour savoir comment gérer les abonnements dans Event Grid, voir [Documentation sur l’API REST](api.md).

## <a name="event-handlers"></a>Gestionnaires d’événements

Pour Event Grid, un gestionnaire d’événements désigne l’endroit où l’événement est envoyé. Le gestionnaire effectue des actions supplémentaires pour traiter l’événement. Event Grid prend en charge plusieurs types de gestionnaires. Vous pouvez utiliser un service Azure pris en charge ou votre propre webhook comme gestionnaire. En fonction du type de gestionnaire, Event Grid suit différents mécanismes pour garantir la distribution de l’événement. Si le gestionnaire d’événements de destination est un webhook HTTP, l’événement est retenté lorsque le gestionnaire retourne le code de statut `200 – OK`. Dans le cas d’un hub Edge, un événement remis sans exception est considéré comme réussi.

## <a name="security"></a>Sécurité

Event Grid assure la sécurité lors de l’abonnement et de la publication de rubriques. Pour en savoir plus, consultez la page [Sécurité et authentification pour Event Grid](security-authentication.md).

## <a name="event-delivery"></a>Remise d’événement

Si Event Grid ne peut pas confirmer qu’un événement a été reçu par le point de terminaison de l’abonné, il procède à une nouvelle distribution de l’événement. Pour plus d’informations, consultez la page [Distribution et nouvelle tentative de distribution de messages avec Event Grid](delivery-retry.md).

## <a name="batching"></a>Traitement par lot

Quand vous utilisez une rubrique personnalisée, les événements doivent toujours être publiés dans un tableau. Dans les scénarios de faible débit, le tableau n’a qu’une seule valeur. Dans les cas d’utilisation à volume élevé, nous vous recommandons de regrouper plusieurs événements par publication pour augmenter l’efficacité. La taille maximale d’un lot est de 1 Mo. Un événement ne doit pas dépasser 1 Mo (préversion).