---
title: Envoi de notifications VoIP APNS avec Azure Notification Hubs
description: Découvrez comment envoyer des notifications VoIP APNS via Azure Notification Hubs (non officiellement pris en charge).
author: sethmanheim
ms.author: sethm
ms.date: 3/23/2020
ms.topic: how-to
ms.service: notification-hubs
ms.openlocfilehash: c99af881b8f93b75633741c2352dc5df17dd2963
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80137649"
---
# <a name="use-apns-voip-through-notification-hubs-not-officially-supported"></a>Utiliser VoIP APNS par le biais de Notification Hubs (non officiellement pris en charge)

Il est possible d’utiliser les notifications VoIP APNS par le biais d’Azure Notification Hubs ; toutefois, il n’existe pas de prise en charge officielle pour ce scénario.

## <a name="considerations"></a>Considérations

Si vous choisissez tout de même d’envoyer des notifications VoIP APNS par le biais de Notification Hubs, tenez compte des limitations suivantes :

- L’envoi d’une notification VoIP requiert que l’en-tête `apns-topic` soit défini sur l’ID de bundle de l’application et sur le suffixe `.voip`. Par exemple, pour un exemple d’application avec l’ID de bundle `com.microsoft.nhubsample`, l’en-tête `apns-topic` doit être défini sur `com.microsoft.nhubsample.voip.`.

   Cette méthode ne fonctionne pas correctement avec Azure Notification Hubs, car l’ID de bundle de l’application doit être configuré dans le cadre des informations d’identification APNS du hub, et la valeur ne peut pas être modifiée. En outre, Notification Hubs n’autorise pas la substitution de la valeur de l’en-tête `apns-topic` au moment de l’exécution.

   Pour envoyer des notifications VoIP, vous devez configurer un hub de notification distinct avec l’ID de bundle d’application `.voip`.

- L’envoi d’une notification VoIP requiert que l’en-tête `apns-push-type` soit défini sur la valeur `voip`.

   Pour aider les clients à passer à iOS 13, Notification Hubs tente de déduire la valeur correcte pour l’en-tête `apns-push-type`. La logique d’inférence est intentionnellement simple, afin d’éviter d’interrompre les notifications standard. Malheureusement, cette méthode provoque des problèmes avec les notifications VoIP, car Apple traite ces dernières comme un cas spécial qui ne suit pas les mêmes règles que les notifications standard.

   Pour envoyer des notifications VoIP, vous devez spécifier une valeur explicite pour l’en-tête `apns-push-type`.

- Notification Hubs limite les charges utiles APNS à 4 Ko, comme indiqué par Apple. Pour les notifications VoIP, Apple autorise les charges utiles jusqu’à 5 Ko. Notification Hubs ne fait pas la différence entre les notifications standard et les notifications VoIP ; par conséquent, toutes les notifications sont limitées à 4 Ko.

   Pour envoyer des notifications VoIP, vous ne devez pas dépasser la limite de taille de charge utile de 4 Ko.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, consultez les liens suivants :

- [Documentation relative aux en-têtes et valeurs `apns-topic` et `apns-push-type`, y compris les cas particuliers pour les notifications VoIP](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns).

- [Documentation relative à la limite de taille de charge utile](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification).

- [Mises à jour de Notification Hubs pour iOS 13](push-notification-updates-ios-13.md#apns-push-type).
