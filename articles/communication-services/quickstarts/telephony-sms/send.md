---
title: Démarrage rapide - Envoyer un SMS
titleSuffix: An Azure Communication Services quickstart
description: Découvrez comment envoyer un SMS à l’aide d’Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
ms.custom: tracking-python, devx-track-js
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: 60c51de4e4549649c681c961c6ddc1acdb12e698
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659690"
---
# <a name="quickstart-send-an-sms-message"></a>Démarrage rapide : Envoi d’un SMS

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

> [!IMPORTANT]
> Les messages SMS peuvent être envoyés et reçus sur des numéros de téléphone situés aux États-Unis. Les numéros de téléphone appartenant à d’autres zones géographiques ne sont pas pris en charge pour les SMS Communication Services.
> Pour plus d’informations, consultez **[Planifier votre solution de téléphonie et SMS](../../concepts/telephony-sms/plan-solution.md)** .

::: zone pivot="programming-language-csharp"
[!INCLUDE [Send SMS with .NET client library](./includes/send-sms-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [Send SMS with JavaScript client library](./includes/send-sms-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Send SMS with Python client library](./includes/send-sms-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Send SMS with Java client library](./includes/send-sms-java.md)]
::: zone-end

## <a name="troubleshooting"></a>Dépannage

Pour résoudre les problèmes liés à la remise de SMS, vous pouvez [activer la création de rapports de remise avec Event Grid](./handle-sms-events.md) pour capturer les détails de remise.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer un abonnement Communication Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées. Apprenez-en davantage sur le [nettoyage des ressources](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à envoyer des SMS avec Azure Communication Services.

> [!div class="nextstepaction"]
> [S’abonner aux événements SMS](./handle-sms-events.md)

> [!div class="nextstepaction"]
> [Planifier votre solution RTC](../../concepts/telephony-sms/plan-solution.md)

> [!div class="nextstepaction"]
> [En savoir plus sur les SMS](../../concepts/telephony-sms/concepts.md)