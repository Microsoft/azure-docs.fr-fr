---
title: Concepts relatifs aux SMS dans Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Découvrez les concepts relatifs aux SMS dans Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 35398d60008ac52ba16dca0a0201f8c2f2101a0f
ms.sourcegitcommit: 6a4687b86b7aabaeb6aacdfa6c2a1229073254de
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/06/2020
ms.locfileid: "91758555"
---
# <a name="sms-concepts"></a>Concepts relatifs aux SMS

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

Azure Communication Services vous permet d’envoyer et de recevoir des messages texte SMS à l’aide des bibliothèques clientes SMS Communication Services. Ces bibliothèques clientes peuvent être utilisées pour prendre en charge les scénarios de service client, les rappels de rendez-vous, l’authentification à 2 facteurs et d’autres besoins de communication en temps réel. Le service SMS de Communication Services vous permet d’envoyer des messages de manière fiable tout en affichant leur remise et des insights sur le taux de réponse dans le cadre de vos campagnes.

Les principales fonctionnalités des bibliothèques clientes SMS d’Azure Communication Services incluent :

-  Une expérience de configuration **simple** pour l’ajout de la fonctionnalité SMS dans vos applications.
- Une prise en charge des messages à **haute vitesse** via des numéros de téléphone gratuits pour les cas d’utilisation A2P (application à personne) aux États-Unis.
- Des conversations **bidirectionnelles** pour prendre en charge des scénarios tels que le support client, les alertes et les rappels de rendez-vous.
- Une **remise fiable** avec des rapports de remise en temps réel pour les messages envoyés à partir de votre application.
- Des **analyses** pour effectuer le suivi de vos modèles d’utilisation et de l’engagement client.
- Une prise en charge de la gestion des **désactivations** pour détecter et respecter automatiquement les désactivations pour les numéros gratuits. Les désinscriptions des numéros gratuits aux États-Unis sont régies et appliquées par les opérateurs nationaux.
  - STOP : si le destinataire d’un SMS souhaite se désinscrire, il peut envoyer « STOP » au numéro gratuit. L’opérateur envoie la réponse par défaut suivante pour STOP : *« MESSAGE RÉSEAU : Vous avez répondu avec le mot « STOP » qui bloque tout texte envoyé depuis ce numéro. Répondez « unstop » par SMS pour recevoir les messages de nouveau. »*
  - START/UNSTOP : si le destinataire souhaite se réinscrire aux SMS envoyés depuis un numéro gratuit, il peut envoyer « START » ou « UNSTOP » à ce numéro. L’opérateur envoie la réponse par défaut suivante pour START/UNSTOP : *«MESSAGE RÉSEAU : Vous avez répondu « unstop » et allez recommencer à recevoir les messages envoyés depuis ce numéro. »*
  - Azure Communication Services détecte le message STOP et bloque tous les futurs messages adressés au destinataire. Le rapport de remise indique un échec de la remise avec un message d’état selon lequel l’expéditeur est bloqué pour le destinataire concerné.
  - Les messages STOP, UNSTOP et START sont relayés jusqu’à vous. Azure Communication Services vous encourage à superviser et à implémenter ces désinscriptions pour garantir qu’aucune autre tentative d’envoi de message n’est effectuée en direction des destinataires qui ont refusé vos communications.


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [ l’envoi de SMS](../../quickstarts/telephony-sms/send.md)

Les documents suivants peuvent vous intéresser :

- Se familiariser avec la [bibliothèque cliente SMS](../telephony-sms/sdk-features.md)
- Obtenir un [numéro de téléphone](../../quickstarts/telephony-sms/get-phone-number.md) compatible SMS
- [Planifier votre solution SMS](../telephony-sms/plan-solution.md)
