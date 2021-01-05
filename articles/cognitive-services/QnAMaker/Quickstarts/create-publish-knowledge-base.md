---
title: 'Démarrage rapide : Créer, entraîner et publier une base de connaissances - QnA Maker'
description: Vous pouvez créer une base de connaissances QnA Maker à partir de votre propre contenu, tel que des Questions fréquentes (FAQ) ou des manuels. Cet article inclut un exemple de création d’une base de connaissances QnA Maker à partir d’une simple page web de questions fréquentes (FAQ), pour répondre à des questions sur QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: quickstart
ms.date: 11/09/2020
ms.openlocfilehash: 1fe1ad14dc1cc8f5ff5171ef517d23363969be4d
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387783"
---
# <a name="quickstart-create-train-and-publish-your-qna-maker-knowledge-base"></a>Démarrage rapide : Créer, entraîner et publier votre base de connaissances QnA Maker

Vous pouvez créer une base de connaissances QnA Maker à partir de votre propre contenu, tel que des Questions fréquentes (FAQ) ou des manuels. Cet article inclut un exemple de création d’une base de connaissances QnA Maker à partir d’une simple page web de questions fréquentes (FAQ), pour répondre à des questions sur QnA Maker.

## <a name="prerequisites"></a>Prérequis

> [!div class="checklist"]
> * Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.
> * Une [ressource](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) QnA Maker créée dans le portail Azure. Rappelez-vous l’ID Azure Active Directory, l’abonnement et le nom de ressource QnA que vous avez sélectionnés au moment de créer la ressource.

## <a name="create-your-first-qna-maker-knowledge-base"></a>Créer votre première base de connaissances QnA Maker

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

1. Connectez-vous au portail [QnAMaker.ai](https://QnAMaker.ai) avec vos informations d’identification Azure.

2. Dans le portail QnA Maker, sélectionnez **Créer une base de connaissances**.

3. Dans la page **Créer**, ignorez l’**étape 1** si vous disposez déjà de votre ressource QnA Maker.

    Si vous n’avez pas encore créé la ressource, sélectionnez **Créer un service QnA**. Vous êtes dirigé vers le [portail Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) pour configurer un service QnA Maker dans votre abonnement. Rappelez-vous l’ID Azure Active Directory, l’abonnement et le nom de ressource QnA que vous avez sélectionnés au moment de créer la ressource.

    Lorsque vous avez terminé de créer la ressource dans le portail Azure, revenez au portail QnA Maker, actualisez la page du navigateur et passez à l’**étape 2**.

4. À l’**étape 2**, sélectionnez votre annuaire Active Directory, votre abonnement, votre service (ressource) et la langue de toutes les bases de connaissances créées dans le service.

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/qnaservice-selection.png" alt-text="Capture d’écran de la sélection d’une base de connaissances du service QnA Maker":::

5. À l’**étape 3**, nommez votre base de connaissances **My Sample QnA KB**.

6. À l’**étape 4**, configurez les paramètres à l’aide du tableau suivant :

    |Paramètre|Valeur|
    |--|--|
    |**Enable multi-turn extraction from URLs, .pdf or .docx files** (Activer l’extraction multitour à partir d’URL, de fichier .pdf ou .docx)|Activé|
    |**Texte par défaut multitour**| Sélectionnez une option|
    |**+ Ajouter une URL**|`https://www.microsoft.com/en-us/software-download/faq`|
    |**Échange de conversation (chit-chat)**|Sélectionnez **Professional**.|

7. À l’**étape 5**, sélectionnez **Create your KB** (Créer votre base de connaissances).

    Le processus d’extraction prend quelques minutes pour lire le document et identifier les questions et réponses.

    Une fois que QnA Maker a créé la base de connaissances, la page **Knowledge base** (Base de connaissances) s’ouvre. Vous pouvez modifier le contenu de la base de connaissances dans cette page.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

1. Connectez-vous au portail [QnAMaker.ai](https://QnAMaker.ai) avec vos informations d’identification Azure.

2. Dans le portail QnA Maker, sélectionnez **Créer une base de connaissances**.

3. Dans la page **Créer**, ignorez l’**étape 1** si vous disposez déjà de votre ressource QnA Maker.

    Si vous n’avez pas encore créé la ressource, sélectionnez **Créer un service QnA**. Vous êtes dirigé vers le [portail Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) pour configurer un service QnA Maker dans votre abonnement. Rappelez-vous l’ID Azure Active Directory, l’abonnement et le nom de ressource QnA que vous avez sélectionnés au moment de créer la ressource.

    Lorsque vous avez terminé de créer la ressource dans le portail Azure, revenez au portail QnA Maker, actualisez la page du navigateur et passez à l’**étape 2**.

4. À l’**étape 2**, sélectionnez votre annuaire Active Directory, votre abonnement, votre service (ressource) et la langue de toutes les bases de connaissances créées dans le service.

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/connect-your-knowledge-base.png" alt-text="Capture d’écran de la sélection d’une base de connaissances du service QnA Maker (préversion managée)":::

5. À l’**étape 2**, si vous créez la première base de connaissances pour votre service, vous avez la possibilité de rendre le paramètre de langue spécifique à chaque base de connaissances. Une fois le paramètre de langue défini pour la première base de connaissances, vous ne pouvez plus modifier les paramètres du service.

6. À l’ **étape 3**, nommez votre base de connaissances  **My Sample QnA KB**. 

7. À l’**étape 4**, configurez les paramètres à l’aide du tableau suivant :

    |Paramètre|Valeur|
    |--|--|
    |**Enable multi-turn extraction from URLs, .pdf or .docx files** (Activer l’extraction multitour à partir d’URL, de fichier .pdf ou .docx)|Activé|
    |**Texte par défaut multitour**| Sélectionnez une option|
    |**+ Ajouter un fichier**| Télécharger le manuel Surface Laptop à partir de : « https://download.microsoft.com/download/7/B/1/7B10C82E-F520-4080-8516-5CF0D803EEE0/surface-book-user-guide-EN.pdf  » 
    |**Échange de conversation (chit-chat)**|Sélectionnez **Professional**.|

8. À l’**étape 5**, sélectionnez **Create your KB** (Créer votre base de connaissances).

    Le processus d’extraction prend quelques minutes pour lire le document et identifier les questions et réponses.

    Une fois que QnA Maker a créé la base de connaissances, la page **Knowledge base** (Base de connaissances) s’ouvre. Vous pouvez modifier le contenu de la base de connaissances dans cette page.

---

## <a name="add-a-new-question-and-answer-set"></a>Ajouter une nouvelle série de questions et réponses

1. Sur le portail QnA Maker, dans la page **Edit** (Modifier), sélectionnez **+ Add QnA pair** (Ajouter une paire QnA) dans la barre d’outils contextuelle.
1. Ajoutez la question suivante :

    `How many Azure services are used by a knowledge base?`

1. Ajoutez la réponse mise en forme avec la syntaxe _Markdown_ :

    ` * Azure QnA Maker service\n* Azure Cognitive Search\n* Azure web app\n* Azure app plan`

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/add-question-and-answer.png" alt-text="Ajoutez la question sous forme de texte et la réponse mise en forme avec la syntaxe Markdown.":::

    Le symbole Markdown, `*`, est utilisé pour les éléments à puce. Le symbole `\n` est utilisée pour une nouvelle ligne.

    La page **Modifier** affiche la syntaxe Markdown. Quand vous utiliserez le panneau **Test** ultérieurement, la syntaxe Markdown s’affichera correctement.

## <a name="save-and-train"></a>Enregistrer et entraîner

Dans le coin supérieur droit, sélectionnez **Save and train** (Enregistrer et entraîner) pour enregistrer vos modifications et entraîner QnA Maker. Les modifications ne sont conservées que si elles sont enregistrées.

## <a name="test-the-knowledge-base"></a>Tester la base de connaissances

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

1. Dans le portail QnA Maker, en haut à droite, sélectionnez **Test** pour vérifier que les modifications apportées ont pris effet.
2. Entrez un exemple de requête d’utilisateur dans la zone de texte.

    `I want to know the difference between 32 bit and 64 bit Windows`

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/query-dialogue.png" alt-text="Entrez un exemple de requête d’utilisateur dans la zone de texte.":::

3. Sélectionnez **Inspect** pour examiner la réponse plus en détail. La fenêtre de test est utilisée pour tester les changements que vous apportez à la base de connaissances avant de la publier.

4. Sélectionnez à nouveau **Test** (Tester) pour fermer le panneau **Test**.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

1. Dans le portail QnA Maker, en haut à droite, sélectionnez **Test** pour vérifier que les modifications apportées ont pris effet.
2. Entrez un exemple de requête d’utilisateur dans la zone de texte.

    `whats the size of the touchscreen`

3. Si vous activez la fonctionnalité MRC pour votre base de connaissances, en cochant la case **Display short answer** (Afficher une réponse brève), vous verrez également une réponse précise, si elle est disponible, ainsi que le passage de la réponse dans le volet de test. 

    ![Volet de test activé (version managée)](../media/conversational-context/test-pane-with-managed.png)
    

4. Sélectionnez Inspect pour examiner la réponse plus en détail. La fenêtre de test est utilisée pour tester les changements que vous apportez à la base de connaissances avant de la publier. 
5. Sélectionnez à nouveau **Test** (Tester) pour fermer le panneau **Test**.
---

## <a name="publish-the-knowledge-base"></a>Publier la base de connaissances

Quand vous publiez une base de connaissances, son contenu passe de l’index `test` à un index `prod` dans Recherche Azure.

![Capture d’écran du déplacement du contenu de votre base de connaissances](../media/qnamaker-how-to-publish-kb/publish-prod-test.png)

1. Dans le portail QnA Maker, sélectionnez **Publish** (Publier). Ensuite, pour confirmer, sélectionnez **Publish** dans la page.

    Le service QnA Maker est désormais correctement publié. Vous pouvez utiliser le point de terminaison dans votre code d’application ou de bot.

    ![Capture d’écran de la réussite de la publication](../media/qnamaker-create-publish-knowledge-base/publish-knowledge-base-to-endpoint.png)

## <a name="create-a-bot"></a>Créer un bot

Après la publication, vous pouvez créer un bot à partir de la page **Publish** (Publier) :

* Vous pouvez créer plusieurs bots rapidement, tous pointant vers la même base de connaissances pour différentes régions ou différents plans tarifaires pour les bots individuels.
* Si vous ne voulez qu’un seul bot pour la base de connaissances, utilisez le lien **Afficher tous vos bots sur le portail Azure** pour voir la liste de vos bots actuel.

Quand vous apportez des modifications à la base de connaissances et que vous republiez, aucune autre action n’est nécessaire avec le bot. Il est déjà configuré pour fonctionner avec la base de connaissances et il fonctionne avec toutes les modifications futures de la base de connaissances. Chaque fois que vous publiez une base de connaissances, tous les bots qui y sont connectés sont automatiquement mis à jour.

1. Dans le portail QnA Maker, dans la page **Publish**, sélectionnez **Create bot**. Ce bouton apparaît seulement une fois que vous avez publié la base de connaissances.

    ![Capture d’écran de la création d’un bot](../media/qnamaker-create-publish-knowledge-base/create-bot-from-published-knowledge-base-page.png)

1. Un nouvel onglet de navigateur s’ouvre pour le portail Azure, avec la page de création d’Azure Bot Service. Configurez le service bot Azure. Le bot et QnA Maker peuvent partager le plan du service Web App, mais ils ne peuvent pas partager l’application web. Cela signifie que le **nom de l’application** pour le bot doit être différent du nom de l’application pour le service QnA Maker.

    * **À faire**
        * Changer le descripteur du bot, s’il n’est pas unique.
        * Sélectionner le langage du SDK. Une fois le bot créé, vous pouvez télécharger le code dans votre environnement de développement local et poursuivre le processus de développement.
    * **À ne pas faire**
        * Modifiez les paramètres suivants sur le portail Azure au moment de créer le bot. Ils sont préremplis pour votre base de connaissances existante :
           * Clé d’authentification QnA
           * Plan et emplacement App Service


1. Une fois le bot créé, ouvrez la ressource **Service Bot**.
1. Sous **Gestion du bot**, sélectionnez **Tester dans le Web Chat**.
1. À l’invite de chat **Tapez votre message**, entrez :

    `Azure services?`

    Le chatbot répond avec une réponse provenant de votre base de connaissances.

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/test-web-chat.png" alt-text="Entrez une requête d’utilisateur dans le web chat de test.":::

## <a name="what-did-you-accomplish"></a>Qu’avez-vous fait ?

Vous avez créé une base de connaissances, ajouté une URL publique à la base de connaissances, ajouté votre propre paire de Q/R, entraîné, testé puis publié la base de connaissances.

Après avoir publié la base de connaissances, vous avez créé un bot et testé celui-ci.

Tout cela a été fait en quelques minutes sans avoir à écrire de code ni à supprimer le contenu.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne passez pas au démarrage rapide suivant, supprimez les ressources de QnA Maker et de Bot Framework dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajouter des questions avec des métadonnées](add-question-metadata-portal.md)

Pour plus d'informations :

* [Format Markdown dans les réponses](../reference-markdown-format.md)
* [Sources de données](../index.yml) QnA Maker
