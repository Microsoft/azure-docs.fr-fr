---
title: Guide pratique pour modifier votre modèle LUIS à l’aide de l’API REST
titleSuffix: Azure Cognitive Services
description: Dans cet article, ajoutez des exemples d’énoncés pour changer un modèle et entraîner l’application.
services: cognitive-services
manager: nitinme
ms.custom: seodec18, devx-track-python, devx-track-js, devx-track-csharp
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 11/30/2020
zone_pivot_groups: programming-languages-set-one
ms.openlocfilehash: 7346037ad804da4f68a2bc6512494c461d7fc5bd
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436179"
---
# <a name="how-to-change-the-luis-model-with-rest-apis"></a>Guide pratique pour modifier le modèle LUIS avec des API REST

Dans cet article, vous allez ajouter des exemples d’énoncés à une application de pizza pour entraîner l’application. Les exemples d’énoncés sont du texte utilisateur conversationnel mappé à une intention. En fournissant des exemples d’énoncés pour les intentions, vous apprenez à l’application LUIS quels types de texte fourni par l’utilisateur appartiennent à quelle intention.

::: zone pivot="programming-language-csharp"
[!INCLUDE [Get intent with C# and REST](./includes/get-started-get-model-rest-csharp.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Get intent with Java and REST](./includes/get-started-get-model-rest-java.md)]
::: zone-end

::: zone pivot="programming-language-go"
[!INCLUDE [Get intent with Go and REST](./includes/get-started-get-model-rest-go.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [Get intent with Node.js and REST](./includes/get-started-get-model-rest-nodejs.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Get intent with Python and REST](./includes/get-started-get-model-rest-python.md)]
::: zone-end
