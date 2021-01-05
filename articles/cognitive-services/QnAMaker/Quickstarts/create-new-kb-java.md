---
title: 'Démarrage rapide : Créer une base de connaissances - REST, Java - QnA Maker'
description: Ce guide de démarrage rapide basé sur Java REST vous guide dans la création, par programmation, d’un exemple de base de connaissances QnA Maker qui apparaîtra dans le tableau de bord Azure de votre compte d’API Cognitive Services.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.date: 12/16/2019
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27, devx-track-java
ms.topic: how-to
ms.openlocfilehash: 50433ee1e5a575ab671d562bfc3fe549b26fe00c
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352301"
---
# <a name="quickstart-create-a-knowledge-base-in-qna-maker-using-java"></a>Démarrage rapide : Créer une base de connaissances dans QnA Maker à l’aide de Java

Ce démarrage rapide vous aide à créer par programmation un exemple de base de connaissances QnA Maker. QnA Maker extrait automatiquement les questions et les réponses à partir du contenu semi-structuré, telles que les FAQ, de [sources de données](../index.yml). Le modèle de la base de connaissances est défini dans le code JSON envoyé dans le corps de la requête d’API.

Ce démarrage rapide fait appel aux API QnA Maker :
* [Créer la base de connaissances](/rest/api/cognitiveservices/qnamaker/knowledgebase/create)
* [Obtenir les détails de l’opération](/rest/api/cognitiveservices/qnamaker/operations/getdetails)

[Documentation de référence](/rest/api/cognitiveservices/qnamaker/knowledgebase) | [Exemple Java](https://github.com/Azure-Samples/cognitive-services-qnamaker-java/blob/master/documentation-samples/quickstarts/create-knowledge-base/CreateKB.java)

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Conditions préalables requises

* [Go 1.10.1](https://golang.org/dl/)
* Vous devez disposer d’un [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Pour récupérer votre clé et votre point de terminaison (qui incluent le nom de la ressource), sélectionnez **Démarrage rapide** pour votre ressource dans le portail Azure.

L’[exemple de code](https://github.com/Azure-Samples/cognitive-services-qnamaker-java/blob/master/documentation-samples/quickstarts/create-knowledge-base/CreateKB.java) est disponible sur le dépôt GitHub pour QnA Maker avec Java.

## <a name="create-a-knowledge-base-file"></a>Créer un fichier de base de connaissances

Créer un fichier appelé `CreateKB.java`

## <a name="add-the-required-dependencies"></a>Ajouter les dépendances nécessaires

En haut de `CreateKB.java`, ajoutez les lignes suivantes pour ajouter les dépendances nécessaires au projet :

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="dependencies":::

## <a name="add-the-required-constants"></a>Ajouter les constantes nécessaires
Après les dépendances nécessaires précédentes, ajoutez les constantes nécessaires à la classe `CreateKB` pour accéder à QnA Maker.

Vous devez disposer d’un [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Pour récupérer votre clé et le nom de votre ressource, sélectionnez **Démarrage rapide** dans le portail Azure pour votre ressource QnA Maker.

Définissez les valeurs suivantes :

* `<your-qna-maker-subscription-key>` – La clé (**key**) est une chaîne de 32 caractères qui est disponible sur le portail Azure, dans la ressource QnA Maker, dans la page Démarrage rapide. Il ne s’agit pas de la clé du point de terminaison de prédiction.
* `<your-resource-name>` – Votre nom de ressource (**resource name**) est utilisé pour construire l’URL du point de terminaison de création au format `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Ce nom de ressource n’est pas le même que celui utilisé pour interroger le point de terminaison de prédiction.

Vous n’avez pas besoin d’ajouter la dernière accolade à la fin de la classe ; elle est déjà dans l’extrait de code final à la fin de ce guide de démarrage rapide.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="constants":::


## <a name="add-the-kb-model-definition-classes"></a>Ajouter les classes de la définition du modèle de base de connaissances
Après les constantes, ajoutez les classes et fonctions suivantes dans la classe `CreateKB` pour sérialiser l’objet de définition du modèle dans JSON.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="model":::

## <a name="add-supporting-functions"></a>Ajouter des fonctions de prise en charge

Ajoutez ensuite les fonctions de prise en charge suivantes dans la classe `CreateKB`.

1. Ajoutez la fonction suivante pour imprimer JSON dans un format lisible :

    :::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="pretty":::

2. Ajoutez la classe suivante pour gérer la réponse HTTP :

    :::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="response":::

3. Ajoutez la méthode suivante pour effectuer une requête POST dans les API QnA Maker. `Ocp-Apim-Subscription-Key` correspond à la clé de service QnA Maker, utilisée pour l’authentification.

    :::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="post":::

4. Ajoutez la méthode suivante pour effectuer une requête GET dans les API QnA Maker.

    :::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="get":::

## <a name="add-a-method-to-create-the-kb"></a>Ajouter une méthode pour créer la base de connaissances
Ajoutez la méthode suivante pour créer la base de données en l’appelant dans la méthode POST.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="create_kb":::

Cet appel d’API renvoie une réponse JSON qui inclut l’ID d’opération. Utilisez l’ID d’opération pour déterminer si la base de connaissances a bien été créée.

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-09-26T05:19:01Z",
  "lastActionTimestamp": "2018-09-26T05:19:01Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "8dfb6a82-ae58-4bcb-95b7-d1239ae25681"
}
```

## <a name="add-a-method-to-get-status"></a>Ajouter une méthode pour obtenir le statut
Ajoutez la méthode suivante pour vérification le statut de création.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="get_status":::

Répétez l’appel jusqu’à ce que l’opération réussisse ou échoue :

```JSON
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:23:08Z",
  "resourceLocation": "/knowledgebases/XXX7892b-10cf-47e2-a3ae-e40683adb714",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "177e12ff-5d04-4b73-b594-8575f9787963"
}
```

## <a name="add-a-main-method"></a>Ajouter une méthode Main
La méthode Main crée la base de connaissances, puis demande le statut. L’ID d’opération est renvoyé dans le champ **Location** de l’en-tête de réponse POST. Il est ensuite utilisé dans le routage de la requête GET. Les boucles `while` testent à nouveau le statut s’il n’est pas terminé.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/CreateKB.java" id="main":::

## <a name="compile-and-run-the-program"></a>Compiler et exécuter le programme

1. Assurez-vous que la bibliothèque gson se trouve dans le répertoire `./libs`. Sur la ligne de commande, compilez le fichier `CreateKB.java` :

    ```bash
    javac -cp ".;libs/*" CreateKB.java
    ```

2. Entrez la commande suivante sur une ligne de commande pour exécuter le programme. Il envoie la requête à l’API QnA Maker pour créer la base de connaissances, puis demande des résultats toutes les 30 secondes. Chaque réponse est imprimée sur la fenêtre de console.

    ```bash
    java -cp ",;libs/*" CreateKB
    ```

Une fois votre base de connaissances créée, vous pouvez l’afficher dans votre portail QnA Maker, sur la page [Mes bases de connaissances](https://www.qnamaker.ai/Home/MyServices).

[!INCLUDE [Clean up files and KB](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Documentation de référence pour l’API REST QnA Maker (V4)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)