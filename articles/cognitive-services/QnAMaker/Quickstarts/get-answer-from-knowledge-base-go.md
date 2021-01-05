---
title: 'Démarrage rapide : Obtenir des réponses d’une base de connaissances - REST, Go - QnA Maker'
description: Ce guide de démarrage rapide basé sur REST Go vous aide à obtenir par programmation une réponse à partir d’une base de connaissances.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.date: 02/08/2020
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27
ms.topic: how-to
ms.openlocfilehash: f0c3eda7172cd871273d558d452b32ac2d40ef5e
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96346063"
---
# <a name="quickstart-get-answers-to-a-question-from-a-knowledge-base-with-go"></a>Démarrage rapide : Obtenir des réponses à une question à partir d’une base de connaissance avec Go

Ce guide de démarrage rapide vous aide à obtenir programmatiquement une réponse à partir d’une base de connaissances QnA Maker publiée. La base de connaissances contient des questions et réponses de [sources de données](../index.yml) telles que des FAQ. La [question](../how-to/metadata-generateanswer-usage.md#generateanswer-request-configuration) est envoyée au service QnA Maker. La [réponse](../how-to/metadata-generateanswer-usage.md#generateanswer-response-properties) inclut la réponse la plus prévisible.

[Documentation de référence](/rest/api/cognitiveservices/qnamaker4.0/Runtime) | [Exemple](https://github.com/Azure-Samples/cognitive-services-qnamaker-go/blob/master/documentation-samples/quickstarts/get-answer/get-answer.go)

## <a name="prerequisites"></a>Prérequis

* [Go 1.10.1](https://golang.org/dl/)
* [Visual Studio Code](https://code.visualstudio.com/)
* Vous devez disposer d’un [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Pour récupérer votre clé, sélectionnez **Clés** sous **Gestion des ressources** dans votre tableau de bord Azure pour votre ressource QnA Maker.
* Paramètres de la page **Publier**. Si vous ne disposez pas d’une base de connaissances publiée, créez une base de connaissances vide, importez une base de connaissances sur la page **Paramètres**, puis publiez. Vous pouvez télécharger et utiliser [cette base de connaissances](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/knowledge-bases/basic-kb.tsv).

    Les paramètres de la page de publication incluent la valeur de route POST, la valeur d’hôte et la valeur EndpointKey.

    ![Paramètres de publication](../media/qnamaker-quickstart-get-answer/publish-settings.png)

## <a name="create-a-go-file"></a>Créer un fichier Go

Ouvrez VSCode et créez un fichier nommé `get-answer.go`, puis ajoutez la classe suivante :

```Go
package main

func main() {

}
```

## <a name="add-the-required-dependencies"></a>Ajouter les dépendances nécessaires

Au-dessus de la fonction `main`, en haut du fichier `get-answer.go`, ajoutez les dépendances nécessaires au projet :

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/query-kb.go" id="dependencies":::

## <a name="add-a-post-request-to-send-question-and-get-answer"></a>Ajouter une requête POST pour envoyer une question et obtenir une réponse

Le code suivant adresse une requête HTTPS à l’API QnA Maker afin d’envoyer la question à la base de connaissances et reçoit la réponse :

:::code language="go" source="~/cognitive-services-quickstart-code/go/QnAMaker/rest/query-kb.go" id="main":::

La valeur de l’en-tête `Authorization` inclut la chaîne `EndpointKey`.

En savoir plus sur la [requête](../how-to/metadata-generateanswer-usage.md#generateanswer-request) et la [réponse](../how-to/metadata-generateanswer-usage.md#generateanswer-response).

## <a name="build-and-run-the-program"></a>Créez et exécutez le projet.

Générez et exécutez le programme à partir de la ligne de commande. Il envoie automatiquement la requête à l’API QnA Maker, puis affiche la réponse dans la fenêtre de console.

1. Générez le fichier :

    ```bash
    go build get-answer.go
    ```

1. Exécutez le fichier :

    ```bash
    ./get-answer
    ```

[!INCLUDE [JSON request and response](../../../../includes/cognitive-services-qnamaker-quickstart-get-answer-json.md)]


[!INCLUDE [Clean up files and knowledge base](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Documentation de référence pour l’API REST QnA Maker (V4)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)