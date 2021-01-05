---
title: 'Tutoriel : Intégrer Power BI au service cognitif Analyse de texte'
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser l’API Analyse de texte pour extraire les expressions clés d’un texte stocké dans Power BI.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: d987797c2c25f685a3c9250afeb17cec3ad3cb2e
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369543"
---
# <a name="tutorial-integrate-power-bi-with-the-text-analytics-cognitive-service"></a>Tutoriel : Intégrer Power BI au service cognitif Analyse de texte

Microsoft Power BI Desktop est une application gratuite qui vous permet de vous connecter à vos données, de les transformer et de les visualiser. Le service Analyse de texte, qui fait partie de l’offre Microsoft Azure Cognitive Services, fournit des fonctionnalités de traitement en langage naturel. Dans un texte brut non structuré, il est capable d’extraire les phrases les plus importantes, d’analyser les sentiments et d’identifier les entités bien connues, telles que les marques. Utilisés conjointement, ces outils peuvent vous aider à identifier rapidement ce dont parlent vos clients, ainsi que leurs sentiments à ce sujet.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Utiliser Power BI Desktop pour importer et transformer des données
> * Créer une fonction personnalisée dans Power BI Desktop
> * Intégrer Power BI Desktop à l’API d’extraction de phrases clés Analyse de texte
> * Utiliser l’API d’extraction de phrases clés Analyse de texte pour extraire les phrases plus importantes dans les commentaires clients
> * Créer un nuage de mots-clés à partir des commentaires clients

## <a name="prerequisites"></a>Prérequis
<a name="Prerequisites"></a>

- Microsoft Power BI Desktop. [Téléchargez-le sans frais](https://powerbi.microsoft.com/get-started/).
- Un compte Microsoft Azure [Créez un compte gratuit](https://azure.microsoft.com/free/cognitive-services/) ou [connectez-vous](https://portal.azure.com/).
- Un compte d’API Cognitive Services avec l’API Analyse de texte. Si vous n’en avez pas, vous pouvez vous [inscrire](../../cognitive-services-apis-create-account.md) afin de bénéficier du niveau gratuit qui offre 5 000 transactions/mois (voir [Tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/)), ce qui suffit à suivre ce tutoriel.
- La [clé d’accès Analyse de texte](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) qui a été générée pendant le processus d’inscription.
- Des commentaires de clients. Vous pouvez utiliser [nos exemples de données](https://aka.ms/cogsvc/ta) ou vos propres données. Ce tutoriel suppose que vous utilisez nos exemples de données.

## <a name="load-customer-data"></a>Charger des données client
<a name="LoadingData"></a>

Pour commencer, ouvrez Power BI Desktop et chargez le fichier CSV `FabrikamComments.csv` que vous avez téléchargé dans la section [Prérequis](#Prerequisites). Ce fichier représente une journée d’activité hypothétique dans le forum de support d’une petite entreprise fictive.

> [!NOTE]
> Power BI peut utiliser les données issues d’une multitude de sources, telles que Facebook ou une base de données SQL. Pour plus d’informations, consultez les pages concernant [l’intégration de Facebook à Power BI](https://powerbi.microsoft.com/integrations/facebook/) et [l’intégration de SQL Server à Power BI](https://powerbi.microsoft.com/integrations/sql-server/).

Dans la fenêtre principale de Power BI Desktop, sélectionnez le ruban **Accueil**. Dans le groupe **Données externes** du ruban, ouvrez le menu déroulant **Obtenir des données** , puis sélectionnez **Texte/CSV**.

![[Bouton Obtenir les données]](../media/tutorials/power-bi/get-data-button.png)

La boîte de dialogue Ouvrir s’affiche. Accédez à votre dossier Téléchargements, ou au dossier dans lequel vous avez téléchargé le fichier `FabrikamComments.csv`. Cliquez sur `FabrikamComments.csv`, puis sur le bouton **Ouvrir**. La boîte de dialogue d’importation CSV s’affiche.

![[Boîte de dialogue d’importation CSV]](../media/tutorials/power-bi/csv-import.png)

La boîte de dialogue d’importation CSV vous permet de vérifier que Power BI Desktop a correctement détecté le jeu de caractères, le délimiteur, les lignes d’en-tête et les types de colonnes. Ces informations sont toutes correctes. Vous pouvez donc cliquer sur **Charger**.

Pour visualiser les données chargées, cliquez sur le bouton **Vue de données** situé sur le bord gauche de l’espace de travail Power BI. Cette opération affiche une table contenant des données, comme dans Microsoft Excel.

![[Vue initiale des données importées]](../media/tutorials/power-bi/initial-data-view.png)

## <a name="prepare-the-data"></a>Préparer les données
<a name="PreparingData"></a>

Vous pouvez avoir besoin de transformer vos données dans Power BI Desktop pour que celles-ci puissent être traitées par l’API d’extraction des phrases clés du service Analyse de texte.

Les exemples de données contiennent une colonne `subject` et une colonne `comment`. Avec la fonction Fusionner les colonnes de Power BI Desktop, vous pouvez extraire des phrases clés à partir des données de ces deux colonnes, plutôt que de la colonne `comment` uniquement.

Dans Power BI Desktop, sélectionnez le ruban **Accueil**. Dans le groupe **Données externes** , cliquez sur **Modifier les requêtes**.

![[Groupe Données externes du ruban Accueil]](../media/tutorials/power-bi/edit-queries.png)

Sélectionnez `FabrikamComments` dans la liste **Requêtes** située sur le côté gauche de la fenêtre, si cette requête n’est pas encore sélectionnée.

À présent, sélectionnez à la fois la colonne `subject` et la colonne `comment` de la table. Vous devrez peut-être faire défiler l’écran horizontalement pour voir ces colonnes. Commencez par cliquer sur l’en-tête de la colonne `subject`, puis maintenez la touche CTRL enfoncée tout en cliquant sur l’en-tête de la colonne `comment`.

![[Sélection des champs à fusionner]](../media/tutorials/power-bi/select-columns.png)

Sélectionnez le ruban **Transformer**. Dans le groupe **Colonne Texte** du ruban, cliquez sur **Fusionner les colonnes**. La boîte de dialogue Fusionner les colonnes s’affiche.

![[Fusion de champs à l’aide de la boîte de dialogue Fusionner les colonnes]](../media/tutorials/power-bi/merge-columns.png)

Dans la boîte de dialogue Fusionner les colonnes, choisissez `Tab` comme séparateur, puis cliquez sur **OK**.

Vous pouvez également envisager de filtrer les messages vides en utilisant le filtre Supprimer les éléments vides ou en supprimant les caractères non imprimables à l’aide de la transformation Nettoyer. Si vos données comportent une colonne comparable à la colonne `spamscore` de l’exemple de fichier, vous pouvez ignorer les commentaires « spam » à l’aide d’un filtre Nombre.

## <a name="understand-the-api"></a>Comprendre l’API
<a name="UnderstandingAPI"></a>

[L’API d’extraction de phrases clés](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-V2-1/operations/56f30ceeeda5650db055a3c6) du service Analyse de texte peut traiter jusqu’à mille documents texte par requête HTTP. Power BI traite généralement les enregistrements un par un. Par conséquent, dans ce tutoriel, chacun de vos appels à l’API ne contient qu’un seul document. Pour chaque document à traiter, l’API d’extraction de phrases clés nécessite les champs ci-dessous.

| Champ | Description |
| - | - |
| `id`  | Identificateur unique de ce document dans la requête. La réponse contient également ce champ. De cette façon, si vous traitez plusieurs documents, vous pouvez facilement associer les phrases clés extraites au document dont elles sont issues. Dans ce tutoriel, étant donné que vous ne traitez qu’un seul document par requête, vous pouvez coder en dur la valeur de `id` afin que celle-ci soit la même pour chaque requête.|
| `text`  | Texte à traiter. La valeur de ce champ provient de la colonne `Merged`que vous avez créée dans la [section précédente](#PreparingData), qui contient la combinaison de la ligne d’objet et du texte de commentaire. L’API d’extraction de phrases clés exige que ces données ne dépassent pas 5 120 caractères.|
| `language` | Code représentant le langage naturel dans lequel le document est écrit. Tous les messages des exemples de données sont en anglais. Vous pouvez donc coder en dur la valeur `en` pour ce champ.|

## <a name="create-a-custom-function"></a>Créer une fonction personnalisée
<a name="CreateCustomFunction"></a>

Vous êtes désormais prêt à créer la fonction personnalisée qui doit intégrer Power BI et l’API Analyse de texte. La fonction reçoit le texte à traiter sous la forme d’un paramètre. Elle effectue des conversions de données vers et à partir du format JSON, et adresse la requête HTTP à l’API d’extraction des phrases clés. La fonction analyse alors la réponse de l’API et retourne une chaîne contenant une liste de phrases clés extraites, séparées par des virgules.

> [!NOTE]
> Les fonctions Power BI Desktop personnalisées sont écrites en [langage de formule M Power Query](/powerquery-m/power-query-m-reference), abrégé sous l’appellation « M ». M est un langage de programmation fonctionnel basé sur [F#](/dotnet/fsharp/). Toutefois, vous n’avez pas besoin d’être programmeur pour terminer ce didacticiel ; le code requis est inclus ci-après.

Dans Power BI Desktop, vérifiez que vous vous trouvez toujours dans la fenêtre Éditeur de requête. Si ce n’est pas le cas, sélectionnez le ruban **Accueil** , puis, dans le groupe **Données externes** , cliquez sur **Modifier les requêtes**.

Sur le ruban **Accueil** , dans le groupe **Nouvelle requête** , ouvrez le menu déroulant **Nouvelle source** , puis sélectionnez **Requête vide**. 

Une nouvelle requête, initialement nommée `Query1`, apparaît dans la liste Requêtes. Double-cliquez sur cette entrée et renommez-la `KeyPhrases`.

À présent, sur le ruban **Accueil** , dans le groupe **Requête** , cliquez sur **Éditeur avancé** pour ouvrir la fenêtre correspondante. Supprimez le code figurant déjà dans cette fenêtre, puis collez le code suivant. 

> [!NOTE]
> Remplacez l’exemple de point de terminaison ci-dessous (contenant `<your-custom-subdomain>`) par le point de terminaison généré pour votre ressource Analyse de texte. Pour trouver ce point de terminaison, connectez-vous au [portail Azure](https://azure.microsoft.com/features/azure-portal/), sélectionnez votre abonnement Analyse de texte, puis sélectionnez `Quick start`.


```fsharp
// Returns key phrases from the text in a comma-separated list
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics" & "/v3.0/keyPhrases",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    keyphrases  = Text.Lower(Text.Combine(jsonresp[documents]{0}[keyPhrases], ", "))
in  keyphrases
```

Remplacez `YOUR_API_KEY_HERE` par votre clé d’accès Analyse de texte. De même, pour trouver cette clé, connectez-vous au [portail Azure](https://azure.microsoft.com/features/azure-portal/), sélectionnez votre abonnement Analyse de texte, puis sélectionnez la page de présentation. Veillez à conserver les guillemets situés avant et après la clé. Puis cliquez sur **Terminé**.

## <a name="use-the-custom-function"></a>Utiliser la fonction personnalisée
<a name="UseCustomFunction"></a>

À présent, vous pouvez utiliser la fonction personnalisée pour obtenir les phrases clés figurant dans chacun des commentaires clients et pour les stocker dans une nouvelle colonne de la table. 

Dans Power BI Desktop, dans la fenêtre Éditeur de requête, revenez à la requête `FabrikamComments`. Sélectionnez le ruban **Ajouter une colonne**. Dans le groupe **Général** , cliquez sur **Appeler une fonction personnalisée**.

![[Bouton Appeler une fonction personnalisée]](../media/tutorials/power-bi/invoke-custom-function-button.png)<br><br>

La boîte de dialogue Appeler une fonction personnalisée s’affiche. Dans **Nouveau nom de colonne** , entrez `keyphrases`. Dans **Requête de fonction** , sélectionnez la fonction personnalisée que vous avez créée (`KeyPhrases`).

Un nouveau champ s’affiche dans la boîte de dialogue : **Texte (facultatif)** . Dans ce champ, vous devez entrer la colonne que vous souhaitez utiliser dans le but de fournir des valeurs pour le paramètre `text` de l’API d’extraction de phrases clés. N’oubliez pas que vous avez déjà codé en dur les valeurs des paramètres `language` et `id`. Dans le menu déroulant, sélectionnez `Merged` (la colonne que vous avez créée [précédemment](#PreparingData) en fusionnant les champs d’objet et de message).

![[Appel d’une fonction personnalisée]](../media/tutorials/power-bi/invoke-custom-function.png)

Pour finir, cliquez sur **OK**.

Si tout est prêt, Power BI appelle votre fonction personnalisée une fois pour chaque ligne de la table. Il envoie les requêtes à l’API d’extraction de phrases clés, et ajoute une colonne à la table pour stocker les résultats. Toutefois, avant l’exécution de ce processus, vous pouvez avoir besoin de spécifier les paramètres d’authentification et de confidentialité.

## <a name="authentication-and-privacy"></a>Authentification et confidentialité
<a name="Authentication"></a>

Une fois que vous avez fermé la boîte de dialogue Appeler une fonction personnalisée, il est possible qu’une bannière s’affiche pour vous inviter à spécifier le mode de connexion à l’API d’extraction de phrases clés.

![[Bannière d’informations d’identification]](../media/tutorials/power-bi/credentials-banner.png)

Cliquez sur **Modifier les informations d’identification** , vérifiez que `Anonymous` est sélectionné dans la boîte de dialogue, puis cliquez sur **Se connecter**. 

> [!NOTE]
> Vous devez sélectionner `Anonymous`, car le service Analyse de texte vous authentifie à l’aide de votre clé d’accès. Par conséquent, Power BI n’a pas besoin de fournir d’informations d’identification pour la requête HTTP proprement dite.

> [!div class="mx-imgBorder"]
> ![[Définition du mode d’accès anonyme]](../media/tutorials/power-bi/access-web-content.png)

Si la bannière Modifier les informations d’identification reste visible même après avoir choisi l’accès anonyme, il se peut que vous ayez oublié de coller votre clé d’accès Analyse de texte dans le code de la [fonction personnalisée](#CreateCustomFunction) `KeyPhrases`.

Ensuite, une bannière peut vous inviter à fournir des informations sur la confidentialité de vos sources de données. 

![[Bannière de confidentialité]](../media/tutorials/power-bi/privacy-banner.png)

Cliquez sur **Continuer** , puis choisissez `Public` pour chacune des sources de données de la boîte de dialogue. Ensuite, cliquez sur **Enregistrer**.

![[Définition de la confidentialité des sources de données]](../media/tutorials/power-bi/privacy-dialog.png)

## <a name="create-the-word-cloud"></a>Créer le nuage de mots-clés
<a name="WordCloud"></a>

Une fois que vous avez géré les bannières qui s’affichent, cliquez sur **Fermer et appliquer** dans le ruban Accueil pour fermer l’Éditeur de requête.

Power BI Desktop consacre quelques minutes à l’exécution des requêtes HTTP nécessaires. Pour chaque ligne de la table, la nouvelle colonne `keyphrases` contient les expressions clés détectées dans le texte par l’API Expressions clés. 

Vous allez utiliser cette colonne pour générer un nuage de mots-clés. Pour commencer, cliquez sur le bouton **Rapport** dans la fenêtre principale de Power BI Desktop, à gauche de l’espace de travail.

> [!NOTE]
> Pourquoi utiliser les expressions clés extraites pour générer un nuage de mots, plutôt que le texte complet de chaque commentaire ? Les expressions clés nous précisent les mots *importants* des commentaires de nos clients, et non simplement les *mots les plus courants*. En outre, le dimensionnement des mots dans le nuage résultant n’est pas faussé par l’usage fréquent d’un mot dans un nombre de commentaires relativement faible.

Si vous n’avez pas encore installé l’objet visuel personnalisé Word Cloud, installez-le. Dans le panneau Visualisations à droite de l’espace de travail, cliquez sur les points de suspension ( **...** ) et choisissez **Import From Store** (Importer à partir du Store). Recherchez le terme « cloud », puis cliquez sur le bouton **Ajouter** en regard de l’objet visuel Word Cloud. Power BI installe le visuel Word Cloud et vous avertit lorsque l’installation est terminée.

![[Ajout d’un objet visuel personnalisé]](../media/tutorials/power-bi/add-custom-visuals.png)<br><br>

Commencez par cliquer sur l’icône Word Cloud dans le panneau Visualisations.

![[Icône Word Cloud dans le panneau Visualisations]](../media/tutorials/power-bi/visualizations-panel.png)

Un nouveau rapport s’affiche dans l’espace de travail. Faites glisser le champ `keyphrases` du panneau Champs vers le champ Catégorie du volet Visualisations. Le nuage de mots apparaît à l’intérieur du rapport.

Passez maintenant à la page Format du panneau Visualisations. Dans la catégorie Mots vides, activez l’option **Mots vides par défaut** pour éliminer du nuage les mots courants courts tels que « de ». 

![[Activation des mots vides par défaut]](../media/tutorials/power-bi/default-stop-words.png)

Un peu plus bas dans ce panneau, désactivez les options **Faire pivoter le texte** et **Titre**.

![[Activation du mode Focus]](../media/tutorials/power-bi/word-cloud-focus-mode.png)

Cliquez sur l’outil Mode Focus dans le rapport pour améliorer l’aspect de votre nuage de mots. L’outil développe le nuage de mots afin qu’il remplisse la totalité de l’espace de travail, comme illustré ci-après.

![[Exemple de nuage de mots]](../media/tutorials/power-bi/word-cloud.png)

## <a name="more-text-analytics-services"></a>Autres services Analyse de texte
<a name="MoreServices"></a>

Le service Analyse de texte, appartenant à la gamme Cognitive Services proposée par Microsoft Azure, offre également des fonctionnalités d’analyse des sentiments et de détection de la langue. La détection de la langue se révèle particulièrement utile si les commentaires de vos clients ne sont pas tous en anglais.

Ces deux autres API sont très semblables à l’API d’extraction de phrases clés. Cela signifie que vous pouvez les intégrer à Power BI Desktop à l’aide de fonctions personnalisées qui sont presque identiques à celle que vous avez créée dans ce tutoriel. Il vous suffit de créer une requête vide et de coller le code approprié ci-après dans l’Éditeur avancé, comme vous l’avez fait précédemment. (N’oubliez pas votre clé d’accès !) Ensuite, comme auparavant, utilisez la fonction pour ajouter une nouvelle colonne à la table.

La fonction Analyse des sentiments ci-après renvoie un score évaluant le caractère positif du sentiment exprimé dans le texte.

```fsharp
// Returns the sentiment score of the text, from 0.0 (least favorable) to 1.0 (most favorable)
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com" & "/text/analytics/v3.0/sentiment",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    sentiment   = jsonresp[documents]{0}[confidenceScores]
in  sentiment
```

Voici deux versions d’une fonction Détection de langue. La première retourne le code de langue ISO (par exemple, `en` pour l’anglais), alors que la seconde retourne le nom « convivial » de la langue (par exemple, `English`). Vous pouvez constater que seule la dernière ligne du corps diffère entre les deux versions.

```fsharp
// Returns the two-letter language code (for example, 'en' for English) of the text
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com" & "/text/analytics/v3.0/languages",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    language    = jsonresp[documents]{0}[detectedLanguages]{0}[iso6391Name]
in  language
```
```fsharp
// Returns the name (for example, 'English') of the language in which the text is written
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com" & "/text/analytics/v3.0/languages",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    language    = jsonresp[documents]{0}[detectedLanguages]{0}[name]
in  language
```

Pour finir, voici une variante de la fonction Expressions clés déjà présentée qui renvoie les expressions sous la forme d’un objet de liste, plutôt qu’au moyen d’une chaîne unique d’expressions séparées par des virgules. 

> [!NOTE]
> Le renvoi d’une chaîne unique contribuait à simplifier notre exemple de nuage de mots. En revanche, une liste vous offre un surcroît de souplesse lorsque vous travaillez sur les expressions renvoyées dans Power BI. Vous pouvez manipuler les objets de liste dans Power BI Desktop à l’aide du groupe Colonne structurée dans le ruban Transformation de l’Éditeur de requête.

```fsharp
// Returns key phrases from the text as a list object
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com" & "/text/analytics/v3.0/keyPhrases",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    keyphrases  = jsonresp[documents]{0}[keyPhrases]
in  keyphrases
```

## <a name="next-steps"></a>Étapes suivantes
<a name="NextSteps"></a>

Apprenez-en davantage sur le service Analyse de texte, sur le langage de formule M Power Query ou sur Power BI.

> [!div class="nextstepaction"]
> [Informations de référence sur l’API Analyse de texte](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0)

> [!div class="nextstepaction"]
> [Informations de référence sur le langage M Power Query](/powerquery-m/power-query-m-reference)

> [!div class="nextstepaction"]
> [Documentation Power BI](https://powerbi.microsoft.com/documentation/powerbi-landing-page/)