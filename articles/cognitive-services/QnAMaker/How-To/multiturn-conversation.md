---
title: Conversations multitours dans QnA Maker
description: Servez-vous des invites et du contexte pour gérer plusieurs tours (ou « multitour ») pour votre bot d’une question à l’autre. La conversation multitour est la possibilité d’avoir une conversation nourrie où le contexte de la question précédente influence la question et la réponse suivantes.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 04/13/2020
ms.openlocfilehash: cef2e2ca9c7ad4640014d9b5a9a7da42d308ef7c
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605142"
---
# <a name="use-follow-up-prompts-to-create-multiple-turns-of-a-conversation"></a>Utiliser des invites de suivi pour créer plusieurs tours de conversation

Servez-vous d’invites de suivi et du contexte pour gérer plusieurs tours (ou _multitour_) pour votre bot d’une question à l’autre.

Pour voir comment fonctionne la conversation multitour, regardez la vidéo de démonstration suivante :

[![Conversation multitour dans QnA Maker](../media/conversational-context/youtube-video.png)](https://aka.ms/multiturnexample)

## <a name="what-is-a-multi-turn-conversation"></a>Qu’est-ce qu’une conversations multitour ?

Il n’est pas possible de répondre à certaines questions en un seul tour. Quand vous concevez votre application cliente (bot conversationnel), un utilisateur peut poser une question qui a besoin d’être filtrée ou affinée pour déterminer la bonne réponse. Pour que ce flux puisse se dérouler par le biais des questions, vous devez présenter des *invites de suivi* à l’utilisateur.

Quand un utilisateur pose une question, QnA Maker retourne la réponse _et_ les éventuelles invites de suivi. Cette réponse vous permet de présenter les questions de suivi sous forme de choix.

> [!CAUTION]
> Les invites multitours ne sont pas extraites des documents de FAQ. Si vous avez besoin d’une extraction multitour, supprimez les points d’interrogation qui désignent les paires de questions/réponses en tant que FAQ.

## <a name="example-multi-turn-conversation-with-chat-bot"></a>Exemple de conversation multitour avec un bot conversationnel

Avec la conversation multitour, un bot conversationnel gère une conversation avec un utilisateur pour déterminer la réponse finale, comme le montre l’image suivante :

![Boîte de dialogue multitour à invites qui guident un utilisateur au cours d’une conversation](../media/conversational-context/conversation-in-bot.png)

Dans l’image précédente, un utilisateur a lancé une conversation en entrant **My account**. La base de connaissances contient trois paires de questions/réponses liées. Pour affiner la réponse, l’utilisateur sélectionne un des trois choix de la base de connaissances. La question (#1) comporte trois invites de suivi, qui sont présentées dans le bot conversationnel comme trois options (#2).

Quand l’utilisateur sélectionne une option (#3), la liste suivante d’options d’affinage (#4) est présentée. Cette séquence se poursuit (#5) jusqu’à ce que l’utilisateur détermine la réponse finale correcte (#6).

### <a name="use-multi-turn-in-a-bot"></a>Utiliser la conversation multitour dans un bot

Après avoir publié votre base de connaissances, vous pouvez sélectionner le bouton **Créer un bot** pour déployer votre bot QnA Maker sur Azure Bot Service. Les invites s’affichent dans les clients de conversation que vous avez activés pour votre bot.

## <a name="create-a-multi-turn-conversation-from-a-documents-structure"></a>Créer une conversation multitour à partir de la structure d’un document

Quand vous créez une base de connaissances, la section **Populate your KB** présente une case à cocher **Enable multi-turn extraction from URLs, .pdf or .docx files**.

![Case à cocher permettant d’activer l’extraction multitour](../media/conversational-context/enable-multi-turn.png)

Lorsque vous sélectionnez cette option, QnA Maker extrait la hiérarchie présente dans la structure du document. La hiérarchie est convertie en invites de suivi et la racine de la hiérarchie faite office de QnA parente. Dans certains documents où la racine de la hiérarchie n’a pas de contenu qui pourrait servir de réponse, vous pouvez fournir un texte de réponse par défaut  à utiliser comme texte de réponse de substitution pour extraire de telles hiérarchies.

La structure multitour ne peut être déduite qu’à partir d’URL, de fichiers PDF ou de fichiers DOCX. Pour obtenir un exemple de structure, examinez une image du [fichier PDF du manuel utilisateur Microsoft Surface](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/product-manual.pdf).

:::image type="content" source="../media/conversational-context/import-file-with-conversational-structure.png" alt-text="Capture d'écran représentant un exemple de la structure dans un manuel d'utilisation." lightbox="../media/conversational-context/import-file-with-conversational-structure.png":::

### <a name="building-your-own-multi-turn-document"></a>Création de votre propre document multitour

Si vous créez un document multitour, gardez à l’esprit les instructions suivantes :

* Utilisez des titres et sous-titres pour montrer la hiérarchie. Par exemple, vous pouvez H1 pour dénoter la QnA parente et H2 pour dénoter la QnA à considérer comme invite. Utilisez une petite taille de titre pour dénoter la hiérarchie suivante. N’utilisez pas de style, de couleur ou d’autre mécanisme pour montrer la structure de votre document. QnA Maker n’extrait pas les invites multitours.

* Le premier caractère de l’en-tête doit être en majuscule.

* Ne pas terminer un titre par un point d’interrogation (`?`).

* Vous pouvez utiliser l’[exemple de document](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/multi-turn.docx) en tant qu’exemple pour créer votre propre document multitour.

### <a name="adding-files-to-a-multi-turn-kb"></a>Ajout de fichiers à une base de connaissances multitour

Quand vous ajoutez un document hiérarchique, QnA Maker détermine des invites de suivi à partir de la structure pour créer un flux de conversation.

1. Dans QnA Maker, sélectionnez une base de connaissances existante créée avec la case **Enable multi-turn extraction from URLs, .pdf or .docx files** cochée.
1. Accédez à la page **Settings** (Paramètres), puis sélectionnez le fichier ou l’URL à ajouter.
1. Sélectionnez **Save and train** pour enregistrer et entraîner la base de connaissances.

> [!Caution]
> L’utilisation d’un fichier de base de connaissances multitour TSV ou XLS exporté comme source de données pour une base de connaissances, nouvelle ou vide, n’est pas prise en charge. Vous devez importer (**Import**) ce type de fichier à partir de la page **Settings** du portail QnA Maker pour ajouter des invites multitours exportées à une base de connaissances.

## <a name="create-knowledge-base-with-multi-turn-prompts-with-the-create-api"></a>Créer la base de connaissances avec des invites multitours à l’aide de l’API Create

Vous pouvez créer une base de connaissances avec des invites multitours à l’aide de l’[API Create de QnA Maker](/rest/api/cognitiveservices/qnamaker/knowledgebase/create). Les invites sont ajoutées dans le tableau `prompts` de la propriété `context`.

## <a name="show-questions-and-answers-with-context"></a>Présenter les questions et les réponses avec du contexte

Affichez uniquement les paires de questions/réponses assorties de conversations contextuelles.

Sélectionnez **Options d’affichage**, puis **Afficher le contexte**. La liste affiche les paires de questions/réponses qui contiennent des invites de suivi.

![Filtrer les paires questions/réponses par conversations contextuelles](../media/conversational-context/filter-question-and-answers-by-context.png)

Le contexte multitour s’affiche dans la première colonne.

:::image type="content" source="../media/conversational-context/surface-manual-pdf-follow-up-prompt.png" alt-text="Capture d'écran dans laquelle la section Context est mise en surbrillance." lightbox="../media/conversational-context/surface-manual-pdf-follow-up-prompt.png":::

Dans l’image précédente, **#1** désigne le texte en gras dans la colonne, qui représente la question actuelle. La question parente est le premier élément de la ligne. Les questions situées en dessous sont les paires de questions/réponses liées. Vous pouvez sélectionner ces éléments pour accéder immédiatement aux autres éléments contextuels.

## <a name="add-an-existing-question-and-answer-pair-as-a-follow-up-prompt"></a>Ajouter une paire de questions/réponses existante comme invite de suivi

La question d’origine, **My account**, comporte des invites de suivi, comme **Accounts and signing in**.

![Réponses et invites de suivi pour « Accounts and signing in »](../media/conversational-context/detected-and-linked-follow-up-prompts.png)

Ajoutez une invite de suivi à une paire de questions/réponses existante qui n’est pas actuellement liée. Comme la question n’est liée à aucune paire de questions/réponses, le paramètre d’affichage actuel doit être modifié.

1. Pour lier une paire de questions/réponses existante comme invite de suivi, sélectionnez la ligne correspondant à la paire de questions/réponses. Pour le manuel Surface, recherchez **Sign out** pour réduire la liste.
1. Dans la ligne **Signout**, dans la colonne **Answer**, sélectionnez **Add follow-up prompt**.
1. Dans les champs, de la fenêtre indépendante **Invite de suivi**, entrez les valeurs suivantes :

    |Champ|Valeur|
    |--|--|
    |Texte affiché|Entrez **Turn off the device**. Il s’agit du texte personnalisé à afficher dans l’invite de suivi.|
    |Context-only| Cochez cette case. Une réponse est retournée uniquement si la question spécifie le contexte.|
    |Link to answer|Entrez **Use the sign-in screen** pour rechercher la paire de questions/réponses existante.|

1.  Une seule correspondance est retournée. Sélectionnez cette réponse comme suivi, puis choisissez **Save**.

    ![Page « Follow-up prompt (PREVIEW) »](../media/conversational-context/search-follow-up-prompt-for-existing-answer.png)

1. Une fois que vous avez ajouté l’invite de suivi, sélectionnez **Save and train** dans le volet de navigation supérieur.

### <a name="edit-the-display-text"></a>Modifier le texte d’affichage

Quand une invite de suivi est créée et qu’une paire de questions/réponses existante est entrée en tant que lien vers la réponse (**Link to Answer**), vous pouvez entrer un nouveau texte d’affichage (**Display text**). Ce texte ne remplace pas la question existante et n’ajoute pas une nouvelle question de substitution. Elle est différente de ces valeurs.

1. Pour modifier le texte d’affichage, recherchez la question dans le champ **Context** et sélectionnez-la.
1. Dans la ligne correspondant à cette question, sélectionnez l’invite de suivi dans la colonne de réponse (answer).
1. Sélectionnez le texte d’affichage à modifier, puis sélectionnez **Edit**.

    ![Commande Edit pour le texte d’affichage](../media/conversational-context/edit-existing-display-text.png)

1. Dans la fenêtre indépendante **Follow-up prompt**, modifiez le texte d’affichage existant.
1. Une fois le texte d’affichage modifié, sélectionnez **Save**.
1. Dans la barre de navigation du haut, choisissez **Save and train**.

## <a name="add-a-new-question-and-answer-pair-as-a-follow-up-prompt"></a>Ajouter une nouvelle paire de questions/réponses comme invite de suivi

Quand vous ajoutez une nouvelle paire de questions/réponses à la base de connaissances, chaque paire doit être liée à une question existante en tant qu’invite de suivi.

1. Dans la barre d’outils de la base de connaissances, recherchez la paire de questions/réponses existante pour **Accounts and signing in** et sélectionnez-la.

1. Dans la colonne **Answer** de cette question, sélectionnez **Add follow-up prompt**.
1. Sous **Follow-up prompt (PREVIEW)** , créez une nouvelle invite de suivi en entrant les valeurs suivantes :

    |Champ|Valeur|
    |--|--|
    |Texte affiché|*Create a Windows Account*. Correspond au texte personnalisé à afficher dans l’invite de suivi.|
    |Context-only|Cochez cette case. Cette réponse est retournée uniquement si la question précise le contexte.|
    |Link to answer|Entrez le texte suivant en guise de réponse :<br>*[Create](https://account.microsoft.com/) a Windows account with a new or existing email account*.<br>Quand vous enregistrerez et entraînerez la base de données, ce texte sera converti. |
    |||

    ![Créer une nouvelle question et réponse d’invite](../media/conversational-context/create-child-prompt-from-parent.png)

1. Sélectionnez **Create new**, puis **Save**.

    Cette action a pour effet de créer une nouvelle paire de questions/réponses et de lier la question sélectionnée en tant qu’invite de suivi. La colonne **Context**, pour ces deux questions, indique une relation d’invite de suivi.

1. Sélectionnez **View options**, puis [**Show context (PREVIEW)**](#show-questions-and-answers-with-context).

    La nouvelle question montre comment elle est liée.

    ![Créer une nouvelle invite de suivi](../media/conversational-context/new-qna-follow-up-prompt.png)

    La question parente affiche une nouvelle question parmi ses différents choix.

    :::image type="content" source="../media/conversational-context/child-prompt-created.png" alt-text="Capture d'écran représentant la colonne Context, qui indique une relation d'invite de suivi pour les deux questions." lightbox="../media/conversational-context/child-prompt-created.png":::

1. Une fois que vous avez ajouté l’invite de suivi, sélectionnez **Save and train** dans la barre de navigation supérieure.

<a name="enable-multi-turn-during-testing-of-follow-up-prompts"></a>

## <a name="view-multi-turn-during-testing-of-follow-up-prompts"></a>Afficher la conversation multitour pendant le test des invites de suivi

Quand vous testez la question à l’aide d’invites de suivi dans le volet **Test**, la réponse comprend les invites de suivi.

![La réponse comprend les invites de suivi](../media/conversational-context/test-pane-with-question-having-follow-up-prompts.png)

## <a name="a-json-request-to-return-an-initial-answer-and-follow-up-prompts"></a>Demande JSON de retour d’une réponse initiale et d’invites de suivi

Utilisez l’objet `context` vide pour demander la réponse à la question de l’utilisateur et inclure des invites de suivi.

```JSON
{
  "question": "accounts and signing in",
  "top": 10,
  "userId": "Default",
  "isTest": false,
  "context": {}
}
```

## <a name="a-json-response-to-return-an-initial-answer-and-follow-up-prompts"></a>Réponse JSON de retour d’une réponse initiale et d’invites de suivi

Dans la section précédente, une réponse et les invites de suivi éventuelles à été demandée à **Accounts and signing in**. La réponse comprend les informations d’invite, qui se trouvent dans *answers[0].context*, et le texte à présenter à l’utilisateur.

```JSON
{
    "answers": [
        {
            "questions": [
                "Accounts and signing in"
            ],
            "answer": "**Accounts and signing in**\n\nWhen you set up your Surface, an account is set up for you. You can create additional accounts later for family and friends, so each person using your Surface can set it up just the way he or she likes. For more info, see All about accounts on Surface.com. \n\nThere are several ways to sign in to your Surface Pro 4: ",
            "score": 100.0,
            "id": 15,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": [
                    {
                        "displayOrder": 0,
                        "qnaId": 16,
                        "qna": null,
                        "displayText": "Use the sign-in screen"
                    }
                ]
            }
        },
        {
            "questions": [
                "Sign out"
            ],
            "answer": "**Sign out**\n\nHere's how to sign out: \n\n  Go to Start, and right-click your name. Then select Sign out. ",
            "score": 38.01,
            "id": 18,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": [
                    {
                        "displayOrder": 0,
                        "qnaId": 16,
                        "qna": null,
                        "displayText": "Turn off the device"
                    }
                ]
            }
        },
        {
            "questions": [
                "Use the sign-in screen"
            ],
            "answer": "**Use the sign-in screen**\n\n1.  \n\nTurn on or wake your Surface by pressing the power button. \n\n2.  \n\nSwipe up on the screen or tap a key on the keyboard. \n\n3.  \n\nIf you see your account name and account picture, enter your password and select the right arrow or press Enter on your keyboard. \n\n4.  \n\nIf you see a different account name, select your own account from the list at the left. Then enter your password and select the right arrow or press Enter on your keyboard. ",
            "score": 27.53,
            "id": 16,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": []
            }
        }
    ]
}
```

Le tableau `prompts` fournit le texte dans la propriété `displayText` et la valeur `qnaId`. Vous pouvez présenter ces réponses comme les choix suivants affichés dans le flux de conversation, puis renvoyer le `qnaId` sélectionné à QnA Maker dans la demande suivante.

<!--

The `promptsToDelete` array provides the ...

-->

## <a name="a-json-request-to-return-a-non-initial-answer-and-follow-up-prompts"></a>Demande JSON de retour d’une réponse non initiale et d’invites de suivi

Remplissez l’objet `context` pour inclure le contexte précédent.

Dans la demande JSON suivante, la question actuelle est *Use Windows Hello to sign in* et la question précédente était *accounts and signing in*.

```JSON
{
  "question": "Use Windows Hello to sign in",
  "top": 10,
  "userId": "Default",
  "isTest": false,
  "qnaId": 17,
  "context": {
    "previousQnAId": 15,
    "previousUserQuery": "accounts and signing in"
  }
}
```

##  <a name="a-json-response-to-return-a-non-initial-answer-and-follow-up-prompts"></a>Réponse JSON de retour d’une réponse non initiale et d’invites de suivi

La réponse JSON _GenerateAnswer_ de QnA Maker comprend les invites de suivi dans la propriété `context` du premier élément de l’objet `answers` :

```JSON
{
    "answers": [
        {
            "questions": [
                "Use Windows Hello to sign in"
            ],
            "answer": "**Use Windows Hello to sign in**\n\nSince Surface Pro 4 has an infrared (IR) camera, you can set up Windows Hello to sign in just by looking at the screen. \n\nIf you have the Surface Pro 4 Type Cover with Fingerprint ID (sold separately), you can set up your Surface sign you in with a touch. \n\nFor more info, see What is Windows Hello? on Windows.com. ",
            "score": 100.0,
            "id": 17,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": []
            }
        },
        {
            "questions": [
                "Meet Surface Pro 4"
            ],
            "answer": "**Meet Surface Pro 4**\n\nGet acquainted with the features built in to your Surface Pro 4. \n\nHere's a quick overview of Surface Pro 4 features: \n\n\n\n\n\n\n\nPower button \n\n\n\n\n\nPress the power button to turn your Surface Pro 4 on. You can also use the power button to put it to sleep and wake it when you're ready to start working again. \n\n\n\n\n\n\n\nTouchscreen \n\n\n\n\n\nUse the 12.3" display, with its 3:2 aspect ratio and 2736 x 1824 resolution, to watch HD movies, browse the web, and use your favorite apps. \n\nThe new Surface G5 touch processor provides up to twice the touch accuracy of Surface Pro 3 and lets you use your fingers to select items, zoom in, and move things around. For more info, see Surface touchscreen on Surface.com. \n\n\n\n\n\n\n\nSurface Pen \n\n\n\n\n\nEnjoy a natural writing experience with a pen that feels like an actual pen. Use Surface Pen to launch Cortana in Windows or open OneNote and quickly jot down notes or take screenshots. \n\nSee Using Surface Pen (Surface Pro 4 version) on Surface.com for more info. \n\n\n\n\n\n\n\nKickstand \n\n\n\n\n\nFlip out the kickstand and work or play comfortably at your desk, on the couch, or while giving a hands-free presentation. \n\n\n\n\n\n\n\nWi-Fi and Bluetooth&reg; \n\n\n\n\n\nSurface Pro 4 supports standard Wi-Fi protocols (802.11a/b/g/n/ac) and Bluetooth 4.0. Connect to a wireless network and use Bluetooth devices like mice, printers, and headsets. \n\nFor more info, see Add a Bluetooth device and Connect Surface to a wireless network on Surface.com. \n\n\n\n\n\n\n\nCameras \n\n\n\n\n\nSurface Pro 4 has two cameras for taking photos and recording video: an 8-megapixel rear-facing camera with autofocus and a 5-megapixel, high-resolution, front-facing camera. Both cameras record video in 1080p, with a 16:9 aspect ratio. Privacy lights are located on the right side of both cameras. \n\nSurface Pro 4 also has an infrared (IR) face-detection camera so you can sign in to Windows without typing a password. For more info, see Windows Hello on Surface.com. \n\nFor more camera info, see Take photos and videos with Surface and Using autofocus on Surface 3, Surface Pro 4, and Surface Book on Surface.com. \n\n\n\n\n\n\n\nMicrophones \n\n\n\n\n\nSurface Pro 4 has both a front and a back microphone. Use the front microphone for calls and recordings. Its noise-canceling feature is optimized for use with Skype and Cortana. \n\n\n\n\n\n\n\nStereo speakers \n\n\n\n\n\nStereo front speakers provide an immersive music and movie playback experience. To learn more, see Surface sound, volume, and audio accessories on Surface.com. \n\n\n\n\n",
            "score": 21.92,
            "id": 3,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": [
                    {
                        "displayOrder": 0,
                        "qnaId": 4,
                        "qna": null,
                        "displayText": "Ports and connectors"
                    }
                ]
            }
        },
        {
            "questions": [
                "Use the sign-in screen"
            ],
            "answer": "**Use the sign-in screen**\n\n1.  \n\nTurn on or wake your Surface by pressing the power button. \n\n2.  \n\nSwipe up on the screen or tap a key on the keyboard. \n\n3.  \n\nIf you see your account name and account picture, enter your password and select the right arrow or press Enter on your keyboard. \n\n4.  \n\nIf you see a different account name, select your own account from the list at the left. Then enter your password and select the right arrow or press Enter on your keyboard. ",
            "score": 19.04,
            "id": 16,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": []
            }
        }
    ]
}
```

## <a name="query-the-knowledge-base-with-the-qna-maker-id"></a>Interroger la base de connaissances avec l’ID QnA Maker

Si vous créez une application personnalisée à l’aide de la fonctionnalité multitour. Dans la réponse de la question initiale, les éventuelles invites de suivi et leurs `qnaId` associés sont retournés. Maintenant que vous disposez de l’ID, vous pouvez le transmettre dans le corps de la demande de l’invite de suivi. Si le corps de la demande contient le `qnaId` et l’objet de contexte (qui contient les propriétés QnA Maker précédentes), GenerateAnswer retourne la question exacte par ID, au lieu d’utiliser l’algorithme de classement pour trouver la réponse à partir du texte de la question.

## <a name="display-order-is-supported-in-the-update-api"></a>L’ordre d’affichage est pris en charge dans l’API Update

Le [texte d’affichage et l’ordre d’affichage](/rest/api/cognitiveservices/qnamaker/knowledgebase/update#promptdto), retournés dans la réponse JSON, sont pris en charge par l’[API Update](/rest/api/cognitiveservices/qnamaker/knowledgebase/update) et peuvent être modifiés.

## <a name="add-or-delete-multi-turn-prompts-with-the-update-api"></a>Ajouter ou supprimer des invites multitours à l’aide de l’API Update

Vous pouvez ajouter ou supprimer des invites multitours à l’aide de l’[API Update de QnA Maker](/rest/api/cognitiveservices/qnamaker/knowledgebase/update).  Les invites sont ajoutées dans le tableau `promptsToAdd` et le tableau `promptsToDelete` de la propriété `context`.

## <a name="export-knowledge-base-for-version-control"></a>Exporter la base de connaissances pour la gestion de versions

QnA Maker prend en charge la gestion de versions en incluant des étapes de conversation multitour dans le fichier exporté.

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus en détail les conversations contextuelles à travers cet [exemple de boîte de dialogue](https://github.com/microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/adaptive-dialog/07.qnamaker/QnAMaker.csproj) ou apprenez-en davantage sur la [conception de bots conceptuels pour les conversations multitours](/azure/bot-service/bot-builder-conversations?view=azure-bot-service-4.0).

> [!div class="nextstepaction"]
> [Migrer une base de connaissances](../Tutorials/migrate-knowledge-base.md)