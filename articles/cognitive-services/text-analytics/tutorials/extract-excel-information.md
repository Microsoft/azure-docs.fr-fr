---
title: Extraire des informations dans Excel avec Analyse de texte et Power Automate
titleSuffix: Azure Cognitive Services
description: Découvrez comment extraire du texte Excel sans avoir à écrire du code, avec Analyse de texte et Power Automate.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 02/27/2019
ms.author: aahi
ms.openlocfilehash: 197d28b2ac3d94b6639a6611b2919bdeb2b182e2
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93359899"
---
# <a name="extract-information-in-excel-using-text-analytics-and-power-automate"></a>Extraire des informations dans Excel avec Analyse de texte et Power Automate 

Dans ce tutoriel, vous allez créer un flux Power Automate pour extraire du texte dans une feuille de calcul Excel sans avoir à écrire du code. 

Ce flux prend une feuille de calcul des problèmes signalés à propos d’un immeuble d’habitation et les classifie en deux catégories : plomberie et autres. Il extrait également les noms et numéros de téléphone des locataires qui les ont envoyés. Enfin, le flux ajoute ces informations à la feuille Excel. 

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Utiliser Power Automate pour créer un flux
> * Charger des données Excel depuis OneDrive Entreprise
> * Extraire le texte d’Excel et l’envoyer à l’API Analyse de texte 
> * Utiliser les informations de l’API pour mettre à jour une feuille Excel.

## <a name="prerequisites"></a>Prérequis

- Un compte Microsoft Azure [Créez un compte gratuit](https://azure.microsoft.com/free/cognitive-services/) ou [connectez-vous](https://portal.azure.com/).
- Ressource Analyse de texte. Si vous n’en avez pas, vous pouvez [en créer une dans le portail Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) et utiliser le niveau gratuit pour suivre ce tutoriel.
- [Clé et point de terminaison](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) générés automatiquement pendant l’inscription.
- Feuille de calcul contenant les problèmes des locataires. Des exemples de données sont fournis sur GitHub
- Microsoft 365, avec OneDrive Entreprise.

## <a name="add-the-excel-file-to-onedrive-for-business"></a>Ajouter le fichier Excel à OneDrive Entreprise

Téléchargez l’exemple de fichier Excel à partir de [GitHub](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/TextAnalytics/sample-data/ReportedIssues.xlsx). Ce fichier doit être stocké dans votre compte OneDrive Entreprise.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/example-data.png" alt-text="Exemples provenant du fichier Excel.":::

Les problèmes sont signalés en texte brut. Nous allons utiliser la reconnaissance d’entité nommée de l’API Analyse de texte pour extraire le nom et le numéro de téléphone de la personne. Le flux recherche ensuite le mot « plomberie » dans la description pour catégoriser les problèmes. 

## <a name="create-a-new-power-automate-workflow"></a>Créer un workflow Power Automate

Accédez au [site Power Automate](https://preview.flow.microsoft.com/) et connectez-vous. Cliquez ensuite sur **Créer**, puis sur **Flux planifié**.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/flow-creation.png" alt-text="Écran de création du flux.":::


Dans la page **Créer un flux planifié**, initialisez votre flux avec les champs suivants :

|Champ |Valeur  |
|---------|---------|
|**Nom du flux**     | **Révision prévue** ou un autre nom.         |
|**Démarrage en cours**     |  Entrez les date et heure actuelles.       |
|**Intervalle de récurrence**     | **1 heure**        |

## <a name="add-variables-to-the-flow"></a>Ajouter des variables au flux

> [!NOTE]
> Si vous souhaitez voir une image du flux terminé, vous pouvez la télécharger à partir de [GitHub](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/TextAnalytics/flow-diagrams). 

Créez des variables représentant les informations qui seront ajoutées au fichier Excel. Cliquez sur **Nouvelle étape** et recherchez **Initialiser la variable**. Effectuez cette opération quatre fois, pour créer quatre variables.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/initialize-variables.png" alt-text="Initialiser les variables.":::

Ajoutez les informations suivantes aux variables que vous avez créées. Elles représentent les colonnes du fichier Excel. Si des variables sont réduites, vous pouvez cliquer dessus pour les développer.

| Action |Nom   | Type | Valeur |
|---------|---------|---|---|
| Initialiser la variable | var_person | String | Personne |
| Initialiser la variable 2 | var_phone | String | Numéro_téléphone |
| Initialiser la variable 3 | var_plumbing | String | plomberie |
| Initialiser la variable 4 | var_other | String | Autres | 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/flow-variables.png" alt-text="informations contenues dans les variables de flux":::

## <a name="read-the-excel-file"></a>Lire le fichier Excel

Cliquez sur **Nouvelle étape** et tapez **Excel**, puis sélectionnez **Lister les lignes présentes dans un tableau** dans la liste des actions.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/list-excel-rows.png" alt-text="ajouter des lignes Excel.":::

Ajoutez le fichier Excel au flux en renseignant les champs de cette action. Ce tutoriel nécessite que le fichier soit chargé sur OneDrive Entreprise.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/list-excel-rows-options.png" alt-text="remplir des lignes Excel":::

Cliquez sur **Nouvelle étape** et ajoutez une action **Appliquer à chacun**.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/add-apply-action.png" alt-text="ajouter une commande Appliquer.":::

Cliquez sur **Sélectionnez un résultat à partir des étapes précédentes**. Dans la zone de contenu dynamique qui s’affiche, sélectionnez **valeur**.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/select-output.png" alt-text="Sélectionner un résultat à partir du fichier Excel.":::

## <a name="send-a-request-to-the-text-analytics-api"></a>Envoyer une demande à l’API Analyse de texte

Si ce n’est déjà fait, vous devez créer une [ressource Analyse de texte](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) dans le portail Azure.

### <a name="create-a-text-analytics-connection"></a>Créer une connexion à la ressource Analyse de texte

Dans **Appliquer à chacun**, cliquez sur **Ajouter une action**. Accédez à la page relative **à la clé et au point de terminaison** de votre ressource Analyse de texte dans le portail Azure, et obtenez la clé et le point de terminaison pour votre ressource Analyse de texte.

Dans votre flux, entrez les informations suivantes pour créer une connexion à la ressource Analyse de texte.

> [!NOTE]
> Si vous avez déjà créé une connexion à la ressource Analyse de texte et voulez changer les détails de la connexion, cliquez sur les points de suspension dans le coin supérieur droit, puis sur **+ Ajouter une nouvelle connexion**.

| Champ           | Valeur                                                                                                             |
|-----------------|-------------------------------------------------------------------------------------------------------------------|
| Nom de connexion | Nom de la connexion à votre ressource Analyse de texte. Par exemple : `TAforPowerAutomate`. |
| Clé de compte     | Clé de votre ressource Analyse de texte.                                                                                   |
| URL du site        | Point de terminaison de votre ressource Analyse de texte.                                                       |

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/add-credentials.png" alt-text="Ajouter les informations d’identification Analyse de texte à votre flux.":::

## <a name="extract-the-excel-content"></a>Extraire le contenu Excel 

Une fois la connexion créée, recherchez **Analyse de texte** et sélectionnez **Entités**. Cela permet d’extraire des informations de la colonne de description du problème.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/extract-info.png" alt-text="Ajoutez des entités Analyse de texte.":::

Cliquez dans le champ **Texte** et sélectionnez **Description** dans la fenêtre de contenu dynamique qui s’affiche. Entrez `en` pour Langue. (Cliquez sur Afficher les options avancées si vous ne voyez pas l’option Langue)

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/description-from-dynamic-content.png" alt-text="Ajoutez des paramètres Analyse de texte.":::


## <a name="extract-the-person-name"></a>Extraire le nom de la personne

Nous allons ensuite trouver le type d’entité de la personne dans la sortie Analyse de texte. Dans **Appliquer à chacun**, cliquez sur **Ajouter une action**, puis créez une autre action **Appliquer à chacun**. Cliquez dans la zone de texte et sélectionnez **Entités** dans la fenêtre de contenu dynamique qui s’affiche.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/add-apply-action-2.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 2":::

Dans l’action **Appliquer à chacun 2** nouvellement créée, cliquez sur **Ajouter une action**, puis ajoutez un contrôle **Condition**.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/create-condition.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 3":::

Dans la fenêtre Condition, cliquez sur la première zone de texte. Dans la fenêtre de contenu dynamique, recherchez **Type des entités** et sélectionnez cette option.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/choose-entities-value.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 4":::

Vérifiez que la seconde zone a la valeur **est égal à**. Sélectionnez ensuite la troisième zone, puis recherchez `var_person` dans la fenêtre de contenu dynamique. 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/choose-variable-value.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 5":::

Dans la condition **Si Oui**, tapez Excel, puis sélectionnez **Mettre à jour une ligne**.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/yes-column-action.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 6":::

Entrez les informations Excel, puis mettez à jour les champs **Colonne clé**, **Valeur de la clé** et **PersonName**. Cela permet d’ajouter le nom détecté par l’API à la feuille Excel. 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/yes-column-action-options.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 7":::

## <a name="get-the-phone-number"></a>Obtenir le numéro de téléphone

Réduisez l’action **Appliquer à chacun 2** en cliquant sur le nom. Ajoutez ensuite une autre action **Appliquer à chacun**, comme précédemment. Elle sera nommée **Appliquer à chacun 3**. Sélectionnez la zone de texte et ajoutez **Entités** comme sortie pour cette action. 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/add-apply-action-3.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 8":::

Dans **Appliquer à chacun 3**, ajoutez un contrôle **Condition**. Il sera nommé **Condition 2**. Dans la première zone de texte, recherchez et ajoutez **Type des entités** à partir de la fenêtre de contenu dynamique. Vérifiez que la zone centrale a la valeur **est égal à**. Ensuite, dans la zone de texte de droite, entrez `var_phone`. 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/condition-2-options.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 9":::

Dans la condition **Si Oui**, ajoutez une action **Mettre à jour une ligne**. Entrez ensuite les informations comme ci-dessus pour la colonne des numéros de téléphone de la feuille Excel. Cela permet d’ajouter le numéro de téléphone détecté par l’API à la feuille Excel. 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/condition-2-yes-column.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 10":::


## <a name="get-the-plumbing-issues"></a>Obtenir les problèmes de plomberie

Réduisez **Appliquer à chacun 3** en cliquant sur le nom. Créez ensuite une autre action **Appliquer à chacun** dans l’action parente. Sélectionnez la zone de texte et ajoutez **Entités** comme sortie pour cette action à partir de la fenêtre de contenu dynamique. 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/add-apply-action-4.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 11":::


Le flux vérifie ensuite si la description du problème à partir de la ligne du tableau Excel contient le mot « plomberie ». Si c’est le cas, il ajoute « plomberie » dans la colonne IssueType. Sinon, nous allons entrer « autre ».

Dans l’action **Appliquer à chacun 4**, ajoutez un contrôle **Condition**. Il sera nommé **Condition 3**. Dans la première zone de texte, recherchez et ajoutez **Description** à partir du fichier Excel en utilisant la fenêtre de contenu dynamique. Vérifiez que la zone centrale indique **contient**. Ensuite, dans la zone de texte de droite, recherchez et sélectionnez `var_plumbing`. 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/condition-3-options.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 12":::


Dans la condition **Si Oui**, cliquez sur **Ajouter une action**, puis sélectionnez **Mettre à jour une ligne**. Entrez ensuite les informations comme précédemment. Dans la colonne IssueType, sélectionnez `var_plumbing`. Cela permet d’appliquer une étiquette « plomberie » à la ligne.

Dans la condition **Si Non**, cliquez sur **Ajouter une action**, puis sélectionnez **Mettre à jour une ligne**. Entrez ensuite les informations comme précédemment. Dans la colonne IssueType, sélectionnez `var_other`. Cela permet d’appliquer une étiquette « autres » à la ligne.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/plumbing-issue-condition.png" alt-text="Ajouter des informations d’identification Analyse de texte à votre flux. 13":::

## <a name="test-the-workflow"></a>Tester le flux de travail

Dans le coin supérieur droit de l’écran, cliquez sur **Enregistrer**, puis **Tester**. Sélectionnez **J’effectuerai l’action de déclenchement**. Cliquez sur **Enregistrer et tester**, **Exécuter le flux**, puis **Terminé**.

Le fichier Excel sera mis à jour dans votre compte OneDrive. Il ressemble à ce qui suit.

> [!div class="mx-imgBorder"] 
> :::image type="content" source="../media/tutorials/excel/updated-excel-sheet.png" alt-text="Feuille de calcul Excel mise à jour.":::

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Explorez d’autres solutions](../text-analytics-user-scenarios.md)
