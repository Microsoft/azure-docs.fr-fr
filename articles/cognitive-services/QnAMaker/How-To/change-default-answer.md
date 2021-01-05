---
title: Obtenir la réponse par défaut - QnA Maker
description: La réponse par défaut est retournée quand il n’existe aucune correspondance avec la question. Vous souhaiterez peut-être remplacer la réponse par défaut standard.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: how-to
ms.date: 11/09/2020
ms.openlocfilehash: 5aab021ab5194b4af18e3ff1b2c154ed74710353
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96346118"
---
# <a name="change-default-answer-for-a-qna-maker-resource"></a>Modifier la réponse par défaut pour une ressource QnA Maker

La réponse par défaut d’une base de connaissances doit être retournée quand aucune réponse n’a été trouvée. Si vous utilisez une application cliente, telle que le [service Azure Bot](/azure/bot-service/bot-builder-howto-qna), celle-ci peut également avoir sa propre réponse par défaut, indiquant qu’aucune réponse n’a atteint le seuil de score.

## <a name="types-of-default-answer"></a>Types de réponses par défaut

Il existe deux types de réponses par défaut dans votre base de connaissances. Il est important de comprendre comment et quand chacune d’entre elles est retournée par une requête de prédiction :

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

|Types de réponses par défaut|Description de la réponse|
|--|--|
|Réponse de la base de connaissances lorsqu’aucune réponse n’est déterminée|`No good match found in KB.` : lorsque l’[API GenerateAnswer](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) ne trouve aucune réponse correspondante à la question, le paramètre `DefaultAnswer` du service d’application est retourné. Toutes les bases de connaissances de la même ressource QnA Maker partagent le même texte de réponse par défaut.<br>Vous pouvez gérer le paramètre dans le portail Azure, via le service d’application ou à l’aide des API REST pour [obtenir](/rest/api/appservice/webapps/listapplicationsettings) ou [mettre à jour](/rest/api/appservice/webapps/updateapplicationsettings) le paramètre.|
|Texte d’instruction de l’invite de suivi|Lorsque vous utilisez une invite de suivi dans un flux de conversation, il se peut que vous n’ayez pas besoin d’une réponse dans la paire question/réponse, car vous souhaitez que l’utilisateur choisisse parmi les invites de suivi. Dans ce cas, paramétrez un texte spécifique en définissant le texte de réponse par défaut, qui est retourné avec chaque prédiction pour les invites de suivi. Le texte est destiné à s’afficher sous la forme d’un texte d’instruction pour la sélection des invites de suivi. Un exemple de ce texte de réponse par défaut est `Please select from the following choices`. Cette configuration est expliquée dans les sections suivantes de ce document. Peut également être défini dans le cadre de la définition de la base de connaissances de `defaultAnswerUsedForExtraction` à l’aide de l’[API REST](/rest/api/cognitiveservices/qnamaker/knowledgebase/create).|

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

|Types de réponses par défaut|Description de la réponse|
|--|--|
|Réponse de la base de connaissances lorsqu’aucune réponse n’est déterminée|`No good match found in KB.` - Lorsque l’[API GenerateAnswer](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) ne trouve aucune réponse correspondante à la question, elle affiche une réponse texte par défaut. Dans QnA Maker managé (préversion), vous pouvez définir ce texte dans les **paramètres** de votre base de connaissances. <br><br> ![Définir la réponse par défaut dans QnA Maker managé (préversion)](../media/qnamaker-how-change-default-answer/qnamaker-v2-change-default-answer.png)|
|Texte d’instruction de l’invite de suivi|Lorsque vous utilisez une invite de suivi dans un flux de conversation, il se peut que vous n’ayez pas besoin d’une réponse dans la paire question/réponse, car vous souhaitez que l’utilisateur choisisse parmi les invites de suivi. Dans ce cas, paramétrez un texte spécifique en définissant le texte de réponse par défaut, qui est retourné avec chaque prédiction pour les invites de suivi. Le texte est destiné à s’afficher sous la forme d’un texte d’instruction pour la sélection des invites de suivi. Un exemple de ce texte de réponse par défaut est `Please select from the following choices`. Cette configuration est expliquée dans les sections suivantes de ce document. Vous pouvez aussi le définir dans le cadre d’une définition de base de connaissances avec `defaultAnswerUsedForExtraction` à l’aide de l’[API REST](/rest/api/cognitiveservices/qnamaker/knowledgebase/create).|

---

### <a name="client-application-integration"></a>Intégration d’applications clientes

Pour une application cliente, telle qu’un bot avec le **service bot Azure**, vous pouvez choisir parmi les scénarios courants suivants :

* Utiliser le paramètre de la base de connaissances.
* Utiliser un autre texte dans l’application cliente pour distinguer les cas où une réponse est retournée, mais n’atteint pas le seuil de score. Ce texte peut être du texte statique stocké dans le code ou il peut être stocké dans la liste des paramètres de l’application cliente.

## <a name="set-follow-up-prompts-default-answer-when-you-create-knowledge-base"></a>Définir la réponse par défaut de l’invite de suivi lors de la création de la base de connaissances

Lorsque vous créez une base de connaissances, le texte de la réponse par défaut correspond à un paramètre. Si vous choisissez de ne pas la définir pendant le processus de création, vous pourrez la modifier ultérieurement à l’aide de la procédure suivante.

## <a name="change-follow-up-prompts-default-answer-in-qna-maker-portal"></a>Modifier la réponse par défaut de l’invite de suivi dans le portail QnA Maker

La réponse par défaut de la base de connaissances est retournée quand aucune réponse n’est retournée par le service QnA Maker.

1. Connectez-vous au [portail QnA Maker](https://www.qnamaker.ai/), puis sélectionnez votre base de connaissances dans la liste.
1. Dans la barre de navigation, sélectionnez **Paramètres**.
1. Changez la valeur de **Texte de réponse par défaut** dans la section **Gérer une base de connaissances**.

    :::image type="content" source="../media/qnamaker-concepts-confidencescore/change-default-answer.png" alt-text="Capture d’écran du portail QnA Maker, page Paramètres, avec la zone de texte Réponse par défaut mise en évidence.":::

1. Sélectionnez **Enregistrer et entraîner** pour enregistrer la modification.

## <a name="next-steps"></a>Étapes suivantes

* [Créer une base de connaissances](../How-to/manage-knowledge-bases.md)