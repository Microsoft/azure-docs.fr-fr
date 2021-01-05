---
title: Corriger les mots mal orthographiés – LUIS
titleSuffix: Azure Cognitive Services
description: Corrigez les mots mal orthographiés dans les énoncés en ajoutant l’API Vérification orthographique Bing v7 aux requêtes de point de terminaison LUIS.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 11/19/2019
ms.openlocfilehash: 9d2a1702ea131e9b1b4bf5e586f4290db3aff7ff
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018767"
---
# <a name="correct-misspelled-words-with-bing-spell-check"></a>Corriger les mots mal orthographiés avec la vérification orthographique Bing

Vous pouvez intégrer l’[API Vérification orthographique Bing v7](https://azure.microsoft.com/services/cognitive-services/spell-check/) dans votre application LUIS pour corriger les mots mal orthographiés dans les énoncés avant LUIS prédise le score et les entités de l’énoncé.

[!INCLUDE [Not supported in V3 API prediction endpoint](./includes/v2-support-only.md)]

## <a name="create-endpoint-key"></a>Créer une clé de point de terminaison

Pour créer une ressource Vérification orthographique Bing dans le portail Azure, suivez ces instructions :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le coin supérieur gauche, sélectionnez **Créer une ressource**.

3. Dans la zone de recherche, entrez `Bing Spell Check API V7`.

    ![Rechercher l’API Vérification orthographique Bing v7](./media/luis-tutorial-bing-spellcheck/portal-search.png)

4. Sélectionnez le service.

5. Un panneau d’informations apparaît à droite, avec notamment les mentions légales. Sélectionnez **Créer** pour commencer le processus de création d’abonnement.

6. Dans le panneau suivant, entrez les paramètres du service. Attendez la fin du processus de création de service.

    ![Entrer les paramètres du service](./media/luis-tutorial-bing-spellcheck/subscription-settings.png)

7. Sélectionnez **Toutes les ressources** sous le titre **Favoris** dans la barre de navigation à gauche.

8. Sélectionnez le nouveau service. Son type est **Cognitive Services** et l’emplacement est **global**.

9. Dans le panneau principal, sélectionnez **Clés** pour voir vos nouvelles clés.

    ![Obtenir les clés](./media/luis-tutorial-bing-spellcheck/grab-keys.png)

10. Copiez la première clé. Vous avez seulement besoin d’une des deux clés.

<!--
## Using the key in LUIS test panel
There are two places in LUIS to use the key. The first is in the [test panel](luis-interactive-test.md#view-bing-spell-check-corrections-in-test-panel). The key isn't saved into LUIS but instead is a session variable. You need to set the key every time you want the test panel to apply the Bing Spell Check API v7 service to the utterance. See [instructions](luis-interactive-test.md#view-bing-spell-check-corrections-in-test-panel) in the test panel for setting the key.
-->
## <a name="adding-the-key-to-the-endpoint-url"></a>Ajout de la clé à l’URL de point de terminaison
La requête de point de terminaison a besoin de la clé transmise dans les paramètres de chaîne de requête pour chaque requête à laquelle vous souhaitez appliquer la correction orthographique. Vous pouvez avoir un chatbot qui appelle LUIS, ou vous pouvez appeler l’API de point de terminaison LUIS directement. Quelle que soit la façon dont le point de terminaison est appelé, chaque appel doit inclure les informations requises pour que les corrections orthographiques fonctionnent correctement.

L’URL de point de terminaison a plusieurs valeurs qui doivent être transmises correctement. La clé d’API Vérification orthographique Bing v7 en fait partie. Vous devez définir le paramètre **spellCheck** sur la valeur True et configurer la valeur de **bing-spell-check-subscription-key** avec la valeur de clé :

`https://{region}.api.cognitive.microsoft.com/luis/v2.0/apps/{appID}?subscription-key={luisKey}&spellCheck=true&bing-spell-check-subscription-key={bingKey}&verbose=true&timezoneOffset=0&q={utterance}`

## <a name="send-misspelled-utterance-to-luis"></a>Envoyer un énoncé mal orthographié à LUIS
1. Dans un navigateur web, copiez la chaîne précédente et remplacez `region`, `appId`, `luisKey` et `bingKey` par vos propres valeurs. Vérifiez que vous utilisez la région du point de terminaison, si elle est différente de votre [région](luis-reference-regions.md) de publication.

2. Ajoutez un énoncé mal orthographié, comme « How far is the mountainn ? ». En anglais, `mountain`, avec un `n`, est l’orthographe correcte.

3. Appuyez sur Entrée pour envoyer la requête à LUIS.

4. LUIS répond avec un résultat JSON pour `How far is the mountain?`. Si l’API Vérification orthographique Bing v7 détecte une faute d’orthographe, le champ `query` de la réponse JSON de l’application LUIS contient la requête d’origine, et le champ `alteredQuery` contient la requête corrigée envoyée à LUIS.

```json
{
  "query": "How far is the mountainn?",
  "alteredQuery": "How far is the mountain?",
  "topScoringIntent": {
    "intent": "Concierge",
    "score": 0.183866
  },
  "entities": []
}
```

## <a name="ignore-spelling-mistakes"></a>Ignorer les fautes d’orthographe

Si vous ne souhaitez pas utiliser le service API Vérification orthographique Bing v7, vous devez ajouter l’orthographe correcte et incorrecte.

Pour ce faire, il existe deux solutions :

* Étiquetez les exemples d'énoncés présentant toutes les orthographes différentes pour permettre à LUIS d'apprendre l’orthographe correcte et les fautes de frappe. Cette option requiert davantage d’efforts en matière d’étiquetage que l’utilisation d’un vérificateur d’orthographe.
* Créez une liste d’expressions avec toutes les variantes de mot. Avec cette solution, vous n’êtes pas tenu d’étiqueter les variantes de mot dans les exemples d’énoncés.

## <a name="publishing-page"></a>Page de publication
La page de [publication](luis-how-to-publish-app.md) comporte une case à cocher permettant d’**activer la vérification orthographique Bing**. Cela est pratique pour créer la clé et de comprendre comment l’URL de point de terminaison change. Vous devez toujours utiliser les paramètres de point de terminaison appropriés afin de disposer de l’orthographe corrigée pour chaque énoncé.

> [!div class="nextstepaction"]
> [En savoir plus sur les exemples d’énoncés](./luis-how-to-add-entities.md)