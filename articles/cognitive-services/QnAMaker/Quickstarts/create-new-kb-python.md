---
title: 'Démarrage rapide : Créer une base de connaissances - REST, Python - QnA Maker'
description: Ce guide de démarrage rapide basé sur REST en Python vous guide dans la création, par programmation, d’un exemple de base de connaissances QnA Maker qui apparaîtra dans le tableau de bord Azure de votre compte d’API Cognitive Services.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.date: 12/16/2019
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27, devx-track-python
ms.topic: how-to
ms.openlocfilehash: 9f3f433742ec25a1ee1abb2ede32a38e6b611f14
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352284"
---
# <a name="quickstart-create-a-knowledge-base-in-qna-maker-using-python"></a>Démarrage rapide : Créer une base de connaissances dans QnA Maker à l’aide de Python

Ce guide de démarrage rapide vous aide à créer et publier par programmation un exemple de base de connaissances QnA Maker. QnA Maker extrait automatiquement les questions et les réponses à partir du contenu semi-structuré, telles que les FAQ, de [sources de données](../index.yml). Le modèle de la base de connaissances est défini dans le code JSON envoyé dans le corps de la requête d’API.

Ce démarrage rapide fait appel aux API QnA Maker :
* [Créer la base de connaissances](/rest/api/cognitiveservices/qnamaker/knowledgebase/create)
* [Obtenir les détails de l’opération](/rest/api/cognitiveservices/qnamaker/operations/getdetails)

[Documentation de référence](/rest/api/cognitiveservices/qnamaker/knowledgebase) | [Exemple Python](https://github.com/Azure-Samples/cognitive-services-qnamaker-python/blob/master/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base-3x.py)

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Conditions préalables requises

* [Python 3.7](https://www.python.org/downloads/)
* Vous devez disposer d’un [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Pour récupérer votre clé et votre point de terminaison (qui incluent le nom de la ressource), sélectionnez **Démarrage rapide** pour votre ressource dans le portail Azure.

## <a name="create-a-knowledge-base-python-file"></a>Créer un fichier Python de base de connaissances

Créez un fichier appelé `create-new-knowledge-base-3x.py`.

## <a name="add-the-required-dependencies"></a>Ajouter les dépendances nécessaires

En haut de `create-new-knowledge-base-3x.py`, ajoutez les lignes suivantes pour ajouter les dépendances nécessaires au projet :

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="dependencies":::

## <a name="add-the-required-constants"></a>Ajouter les constantes nécessaires
Après les dépendances nécessaires précédentes, ajoutez les constantes nécessaires pour accéder à QnA Maker. Remplacez la valeur de `<your-qna-maker-subscription-key>` et `<your-resource-name>` par votre propre clé QnA Maker et votre propre nom de ressource.

En haut de la classe Program, ajoutez les constantes nécessaires pour accéder à QnA Maker.

Définissez les valeurs suivantes :

* `<your-qna-maker-subscription-key>` – La clé (**key**) est une chaîne de 32 caractères qui est disponible sur le portail Azure, dans la ressource QnA Maker, dans la page Démarrage rapide. Il ne s’agit pas de la clé du point de terminaison de prédiction.
* `<your-resource-name>` – Votre nom de ressource (**resource name**) est utilisé pour construire l’URL du point de terminaison de création au format `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Il ne s’agit pas de l’URL employée pour interroger le point de terminaison de prédiction.

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="constants":::

## <a name="add-the-kb-model-definition"></a>Ajouter la définition du modèle de base de connaissances

Après les constantes, ajoutez la définition du modèle de base de connaissances suivante. Après la définition, le modèle est converti en une chaîne.

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="model":::

## <a name="add-supporting-function"></a>Ajouter une fonction de prise en charge

Ajoutez la fonction suivante pour imprimer JSON dans un format lisible :

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="pretty":::

## <a name="add-function-to-create-kb"></a>Ajouter une fonction pour créer la base de connaissances

Ajoutez la fonction suivante pour exécuter une requête HTTP POST afin de créer la base de connaissances.
Cet appel d’API renvoie une réponse JSON qui inclut l’ID d’opération dans le champ d’en-tête **Emplacement**. Utilisez l’ID d’opération pour déterminer si la base de connaissances a bien été créée. `Ocp-Apim-Subscription-Key` correspond à la clé de service QnA Maker, utilisée pour l’authentification.

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="create_kb":::

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

## <a name="add-function-to-check-creation-status"></a>Ajouter une fonction pour vérifier l’état de la création

La fonction suivante vérifie l’état de la création en envoyant l’ID d’opération à la fin de l’itinéraire d’URL. L’appel à `check_status` se trouve à l’intérieur de la boucle _while_ principale.

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="get_status":::

Cet appel d’API renvoie une réponse JSON qui inclut l’état de l’opération :

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:22:53Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "177e12ff-5d04-4b73-b594-8575f9787963"
}
```

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

## <a name="add-main-code-block"></a>Ajouter un bloc de code principal
La boucle suivante interroge l’état de l’opération de création régulièrement jusqu’à la fin de l’opération.

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="main":::

## <a name="build-and-run-the-program"></a>Créez et exécutez le projet.

Entrez la commande suivante sur une ligne de commande pour exécuter le programme. Il envoie la requête à l’API QnA Maker pour créer la base de connaissances, puis demande des résultats toutes les 30 secondes. Chaque réponse est imprimée sur la fenêtre de console.

```bash
python create-new-knowledge-base-3x.py
```

Une fois votre base de connaissances créée, vous pouvez l’afficher dans votre portail QnA Maker, sur la page [Mes bases de connaissances](https://www.qnamaker.ai/Home/MyServices). Sélectionnez le nom de votre base de connaissances, par exemple QnA Maker FAQ, pour l’afficher.

[!INCLUDE [Clean up files and KB](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Documentation de référence pour l’API REST QnA Maker (V4)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)