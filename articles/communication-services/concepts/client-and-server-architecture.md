---
title: Architecture client et serveur
titleSuffix: An Azure Communication Services concept document
description: Découvrez l’architecture de Communication Services.
author: mikben
manager: mikben
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: 8b63ce25d6839e8672027240099859dc7aad96c5
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886071"
---
# <a name="client-and-server-architecture"></a>Architecture client et serveur

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

<!--
> [!WARNING]
> This document is under construction and needs the following items to be addressed: 
> - Need to add security best practices for token management here
> - Reference docs:
> - https://docs.microsoft.com/windows/security/threat-protection/security-policy-settings/create-a-token-object
> - https://docs.microsoft.com/azure/aks/operator-best-practices-identity
> - https://docs.microsoft.com/cloud-app-security/api-tokens?view=gestures-1.0-->

Chaque application de Azure Communication Services aura des **applications clientes** qui utilisent des **services**  pour faciliter la connectivité de personne à personne. Cette page illustre des éléments d’architecture courants dans divers scénarios.

## <a name="user-access-management"></a>Gestion de l'accès utilisateur

Les bibliothèques de client Azure Communication Services requièrent `user access tokens` pour accéder aux ressources de Communication Services en toute sécurité. `User access tokens` doivent être générés et gérés par un service approuvé en raison de la nature sensible du jeton et de la chaîne de connexion nécessaire pour les générer. L’impossibilité de gérer correctement les jetons d’accès peut entraîner des frais supplémentaires en raison d’une mauvaise utilisation des ressources. Il est fortement recommandé d’utiliser un service approuvé pour la gestion des utilisateurs. Le service approuvé va générer les jetons et les transmettre au client à l’aide d’un chiffrement approprié. Vous trouverez un exemple de flux d’architecture ci-dessous :

:::image type="content" source="../media/scenarios/archdiagram-access.png" alt-text="Diagramme montrant l’architecture des jetons d’accès utilisateur.":::

Pour plus d’informations, consultez les [meilleures pratiques de gestion des identités](../../security/fundamentals/identity-management-best-practices.md)

## <a name="browser-communication"></a>Communication du navigateur

Les bibliothèques de client JavaScript Azure communications peuvent activer des applications Web avec des fonctionnalités de texte enrichi, de voix et d’interaction vidéo. L’application interagit directement avec Azure Communication Services via la bibliothèque de client pour accéder au plan de données et fournir une communication texte, vocale et vidéo en temps réel. Vous trouverez un exemple de flux d’architecture ci-dessous :

:::image type="content" source="../media/scenarios/archdiagram-browser.png" alt-text="Diagramme montrant l’architecture navigateur à navigateur pour Communication Services.":::

## <a name="native-app-communication"></a>Communication des applications natives

De nombreux scénarios sont mieux servis avec les applications natives. Azure Communication Services prend en charge à la fois les communications navigateur-application et application-application.  Lors de la création d’une application native, la présence de notifications Push permettra aux utilisateurs de recevoir des appels même lorsque l’application ne fonctionne pas. Azure Communication Services facilite cette tâche grâce à des notifications Push intégrées à Google Firebase, Apple Push Notification Service et aux notifications Push Windows. Vous trouverez un exemple de flux d’architecture ci-dessous :

:::image type="content" source="../media/scenarios/archdiagram-app.png" alt-text="Diagramme montrant l’architecture de Communication Services pour la communication des applications natives.":::

## <a name="voice-and-sms-over-the-public-switched-telephony-network-pstn"></a>Voix et SMS sur le réseau téléphonique public commuté (RTPC)

La communication sur le système téléphonique peut augmenter considérablement la portée de votre application. Pour prendre en charge les services vocaux et de SMS sur le RTPC, Azure Communication Services vous aide à [obtenir des numéros de téléphone](../quickstarts/telephony-sms/get-phone-number.md) directement à partir du portail Azure ou à l’aide des API REST et des bibliothèques de client. Une fois les numéros de téléphone acquis, ils peuvent être utilisés pour atteindre les clients à l’aide d’appels sur le RTPC et de SMS dans les scénarios entrants et sortants. Vous trouverez un exemple de flux d’architecture ci-dessous :

> [!Note]
> Pendant la préversion publique, l’obtention de numéros de téléphone américains est disponible pour les clients dont les adresses de facturation se trouvent aux États-Unis et au Canada. 

:::image type="content" source="../media/scenarios/archdiagram-pstn.png" alt-text="Diagramme montrant l’architecture du RTPC dans Communication Services.":::

Pour plus d’informations sur les solutions de RTPC et de SMS, consultez [Planifier votre solution RTPC et votre solution SMS](../concepts/telephony-sms/plan-solution.md)

## <a name="humans-communicating-with-bots-and-other-services"></a>Êtres humains communiquant avec des robots et d’autres services

Azure Communication Services prend en charge la communication entre humains et système via des chaînes de texte et de voix, avec des services qui accèdent directement au plan de données Azure Communication Services. Par exemple, vous pouvez demander à un bot de répondre à des appels téléphoniques entrants ou de participer à une conversation Web. Azure Communication Services fournit des bibliothèques de client qui permettent à ces scénarios d’appeler et de discuter. Vous trouverez un exemple de flux d’architecture ci-dessous :

:::image type="content" source="../media/scenarios/archdiagram-bot.png" alt-text="Diagramme montrant l’architecture des conversations dans Communication Services.":::

## <a name="networking"></a>Mise en réseau

Vous souhaiterez peut-être échanger des données arbitraires entre utilisateurs, par exemple pour synchroniser une réalité mixte partagée ou une expérience de gaming. Le plan de données en temps réel utilisé pour la communication avec le texte, la voix et la vidéo est directement disponible de deux façons :

- **Appel de la bibliothèque de client** - les appareils d’un appel ont accès aux API pour l’envoi et la réception de données via le canal d’appel. Il s’agit du moyen le plus simple d’ajouter des communications de données à une interaction existante.
- **STUN/TURN** : Azure Communication Services met à votre disposition des services STUN et TURN conformes aux normes. Cela vous permet de créer une couche de transport fortement personnalisée sur ces primitives standardisées. Vous pouvez créer votre propre client conforme aux normes ou utiliser des bibliothèques open source, telles que [WinRTC](https://github.com/microsoft/winrtc).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Création de jetons d’accès utilisateur](../quickstarts/access-tokens.md)

Pour plus d’informations, consultez les articles suivants :

- En savoir plus sur l’[authentification](../concepts/authentication.md)
- En savoir plus sur les [solutions RTPC et SMS](../concepts/telephony-sms/plan-solution.md)

- [Ajouter Chat à votre application](../quickstarts/chat/get-started.md)
- [Ajouter l’appel vocal à votre application](../quickstarts/voice-video-calling/getting-started-with-calling.md)