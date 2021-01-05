---
title: Bien démarrer avec l’interopérabilité de Teams sur Azure Communication Services
titleSuffix: An Azure Communication Services quickstart
description: Dans ce guide de démarrage rapide, vous allez apprendre à rejoindre une réunion Teams avec la bibliothèque de client Azure Communication Chat.
author: askaur
ms.author: askaur
ms.date: 12/08/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: ea66e4295e8228aa382aa29a46fcca8147dcbc98
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97578016"
---
# <a name="quickstart-join-your-chat-app-to-a-teams-meeting"></a>Démarrage rapide : Joindre votre application de conversation à une réunion Teams

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

Démarrez avec Azure Communication Services en connectant votre solution de conversation à Microsoft Teams à l’aide de la bibliothèque de client JavaScript. 

## <a name="prerequisites"></a>Prérequis 

1.  [Déploiement Teams](https://docs.microsoft.com/deployoffice/teams-install). 
2. [Application de conversation](./get-started.md) opérationnelle. 

## <a name="enable-teams-interoperability"></a>Activer l’interopérabilité de Teams 

Un utilisateur Communication Services qui rejoint une réunion Teams en tant qu’utilisateur invité ne peut accéder à la conversation de la réunion qu’après avoir rejoint l’appel de réunion Teams. Consultez la documentation [Interopérabilité de Teams](../voice-video-calling/get-started-teams-interop.md) pour savoir comment ajouter un utilisateur Communication Services à un appel de réunion Teams.

La fonctionnalité d’interopérabilité de Teams est disponible en préversion privée. Afin d’activer cette fonctionnalité pour votre ressource Communication Services, envoyez un e-mail à acsfeedback@microsoft.com avec les éléments suivants : 
1. L’ID de l’abonnement Azure qui contient votre ressource Communication Services 
2. Votre ID de locataire Teams Le moyen le plus simple de le récupérer consiste à obtenir et à partager un lien vers l’équipe Teams. 

Vous devez être membre de l’organisation propriétaire des deux entités pour pouvoir utiliser cette fonctionnalité. 

[!INCLUDE [Join Teams meetings](./includes/meeting-interop-javascript.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer un abonnement Communication Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées. Apprenez-en davantage sur le [nettoyage des ressources](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- Consultez notre [exemple de bannière de conversation](../../samples/chat-hero-sample.md).
- Apprenez-en davantage sur le [fonctionnement de la conversation](../../concepts/chat/concepts.md).
