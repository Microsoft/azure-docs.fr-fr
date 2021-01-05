---
title: 'Démarrage rapide : Ajouter un système de conversation à votre application'
titleSuffix: An Azure Communication Services quickstart
description: Ce guide de démarrage rapide vous montre comment ajouter un système de conversation Communication Services à votre application.
author: fanche
manager: phans
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: 2d0d80be367dda3689566dec2ade6fd7fc7c01fc
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91665359"
---
# <a name="quickstart-add-chat-to-your-app"></a>Démarrage rapide : Ajouter un système de conversation à votre application

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

Commencez avec Azure Communication Services en utilisant la bibliothèque de client Communication Services Chat pour ajouter un système de conversation en temps réel à votre application. Dans ce guide de démarrage rapide, nous allons utiliser la bibliothèque de client Chat pour créer des fils de conversation qui permettent aux utilisateurs de discuter entre eux. Pour en savoir plus sur les concepts de Chat, consultez la [documentation conceptuelle de Chat](../../concepts/chat/concepts.md).

::: zone pivot="programming-language-javascript"
[!INCLUDE [Chat with JavaScript client library](./includes/chat-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Chat with Python client library](./includes/chat-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Chat with Java client library](./includes/chat-java.md)]
::: zone-end

::: zone pivot="programming-language-csharp"
[!INCLUDE [Chat with C# client library](./includes/chat-csharp.md)]
::: zone-end

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer un abonnement Communication Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées. Découvrez-en plus sur le [nettoyage des ressources](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez découvert comment :

> [!div class="checklist"]
> * Créer un client de conversation
> * Créer un fil avec deux utilisateurs
> * Envoyer un message au fil
> * Recevoir des messages d’un fil
> * Supprimer des utilisateurs d’un fil

> [!div class="nextstepaction"]
> [Essayez l’application Chat Hero](../../samples/chat-hero-sample.md)

Vous voudrez peut-être aussi :

 - Découvrir les [concepts de Chat](../../concepts/chat/concepts.md)
 - Vous familiariser avec la [bibliothèque de client Chat](../../concepts/chat/sdk-features.md)
