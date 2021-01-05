---
title: Restriction des SMS, e-mails et notifications Push
description: Découvrez comment Azure limite le nombre de notifications possibles par SMS, e-mail, Webhook ou Push Azure App à partir d’un groupe d’actions.
author: dkamstra
ms.author: dukek
ms.topic: conceptual
ms.date: 3/12/2018
ms.subservice: alerts
ms.openlocfilehash: 39b9fd8396601d777aa802a147bee3acc4a22cd0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87045290"
---
# <a name="rate-limiting-for-voice-sms-emails-azure-app-push-notifications-and-webhook-posts"></a>Limitation de la fréquence des appels vocaux, SMS, e-mails, notifications push Azure App et publications Webhook
La limitation de la fréquence est une suspension des notifications qui se produit lorsque trop de notifications sont envoyées à une adresse e-mail, à un numéro de téléphone ou un appareil donnés. Elle garantit que les alertes sont faciles à gérer et exploitables.

Voici les seuils de limitation de la fréquence :

- **SMS** : pas plus de 1 SMS toutes les 5 minutes.
- **Voix** : pas plus de 1 appel vocal toutes les 5 minutes.
- **E-mail** : pas plus de 100 e-mails par heure.
 
  Les autres actions n’ont pas de fréquence limitée.

## <a name="rate-limit-rules"></a>Règles de limitation de la fréquence
- Un numéro de téléphone ou une adresse e-mail donné est soumis à une limitation de la fréquence lorsqu’il reçoit plus de messages que ne l’autorise le seuil.
- Un numéro de téléphone ou une adresse de messagerie peut faire partie de groupes d’actions sur plusieurs abonnements. La limitation de la fréquence s’applique à tous les abonnements, dès lors que le seuil est atteint, même si les messages sont envoyés à partir de plusieurs abonnements.
- Lorsqu’une adresse e-mail fait l’objet d’une restriction, une notification supplémentaire est envoyée à ce sujet. L’e-mail indique la fin de la restriction.

## <a name="next-steps"></a>Étapes suivantes ##
* En savoir plus sur le [comportement des alertes SMS](alerts-sms-behavior.md).
* Obtenir une [vue d’ensemble des alertes du journal d’activité](alerts-overview.md) et découvrir comment recevoir des alertes.  
* Découvrir comment [configurer des alertes lorsqu’une notification d’intégrité de service est publiée](../../service-health/alerts-activity-log-service-notifications-portal.md).
