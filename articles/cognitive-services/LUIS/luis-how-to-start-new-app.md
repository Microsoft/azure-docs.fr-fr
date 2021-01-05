---
title: Créer une nouvelle application - LUIS
titleSuffix: Azure Cognitive Services
description: Créez et gérez vos applications sur la page web de Language Understanding (LUIS).
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/18/2020
ms.openlocfilehash: 2dd06a7b4c8e6296cda747d17fd3d5be5db0af6b
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018886"
---
# <a name="create-a-new-luis-app-in-the-luis-portal"></a>Créer une application LUIS dans le portail LUIS
Il existe deux façons de créer une application LUIS. Vous pouvez créer une application LUIS dans le portail LUIS ou par le biais des [API](developer-reference-resource.md) de création LUIS.

## <a name="using-the-luis-portal"></a>Utilisation du portail LUIS

Vous pouvez créer une application dans le portail de plusieurs manières :

* Démarrez avec une application vide et créer des intentions, des énoncés et des entités.
* Démarrez avec une application vide et ajoutez un [domaine prédéfini](./howto-add-prebuilt-models.md).
* Importez une application LUIS à partir d’un fichier `.lu` ou `.json` qui contient déjà des intentions, des énoncés et des entités.

## <a name="using-the-authoring-apis"></a>Utilisation des API de création
Vous pouvez créer une application avec les API de création de deux manières :

* [Ajoutez une application](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2f) : Démarrez avec une application vide et créer des intentions, des énoncés et des entités.
* [Ajouter une application prédéfinie](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/59104e515aca2f0b48c76be5) : Démarrez avec un domaine prédéfini comprenant des intentions, des énoncés et des entités.


<a name="export-app"></a>
<a name="import-new-app"></a>
<a name="delete-app"></a>


[!INCLUDE [Sign in to LUIS](./includes/sign-in-process.md)]

## <a name="create-new-app-in-luis"></a>Créer une application dans LUIS

1. Dans la page **Mes applications**, sélectionnez votre **Abonnement** et **Ressource de création**, puis **+ Créer**. 

> [!div class="mx-imgBorder"]
> ![Liste des applications LUIS](./media/create-app-in-portal.png)

1. Dans la boîte de dialogue, entrez le nom de votre application, par exemple `Pizza Tutorial`.

    ![Boîte de dialogue Créer une application](./media/create-pizza-tutorial-app-in-portal.png)

1. Choisissez la culture de votre application, puis sélectionnez **Terminé**. La description et la ressource de prédiction sont facultatives à ce stade. Vous pouvez les définir à tout moment dans la section **Gérer** du portail.

    > [!NOTE]
    > La culture ne peut pas être modifiée une fois que l’application est créée.

    Une fois l’application créée, le portail LUIS présente la liste **Intentions** dans laquelle figure l’intention `None` déjà créée pour vous. Vous avez maintenant une application vide.

    > [!div class="mx-imgBorder"]
    > ![Liste des intentions avec l’intention None créée sans aucun exemple d’énoncé.](media/pizza-tutorial-new-app-empty-intent-list.png)

## <a name="other-actions-available-on-my-apps-page"></a>Autres actions disponibles sur la page Mes applications

La barre d’outils de contextuelle propose d’autres actions :

* Renommer l’application
* Importer à partir d’un fichier `.lu` ou `.json`
* Exporter l’application au format `.lu` (pour [LUDown](https://github.com/microsoft/botbuilder-tools/tree/master/packages/Ludown)), `.json` ou `.zip` (pour [conteneur LUIS](luis-container-howto.md))
* Importer les journaux de point de terminaison de conteneur pour réviser les énoncés de point de terminaison
* Exporter les journaux de point de terminaison, au format `.csv`, à des fins d’analyse hors connexion
* Supprimer l’application

## <a name="next-steps"></a>Étapes suivantes

Si la conception de votre application comprend la détection des intentions, [créez de nouvelles intentions](luis-how-to-add-intents.md) et ajoutez des exemples d’énoncés. Si la conception de votre application n’a trait qu’à l’extraction de données, ajoutez des exemples d’énoncés à l’intention None, puis [créez des entités](./luis-how-to-add-entities.md) et étiquetez les exemples d’énoncés avec ces entités.