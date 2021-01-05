---
title: 'Démarrage rapide : Bibliothèque de client Form Recognizer pour .NET'
description: Utilisez la bibliothèque de client Form Recognizer pour .NET afin de créer une application de traitement des formulaires qui extrait des paires clé/valeur et des données de table de vos documents personnalisés.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 10/06/2020
ms.author: pafarley
ms.openlocfilehash: d7577668d87ecaf2d769136d64990f95fc212fe6
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96356526"
---
> [!IMPORTANT]
> Le code indiqué dans cet article utilise des méthodes synchrones et un stockage d’informations d’identification non sécurisé pour des raisons de simplicité.

[Documentation de référence](/dotnet/api/overview/azure/ai.formrecognizer-readme) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/src) | [Package (NuGet)](https://www.nuget.org/packages/Azure.AI.FormRecognizer) | [Exemples](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md)

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* L’[IDE Visual Studio](https://visualstudio.microsoft.com/vs/) ou la version actuelle de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Un blob Stockage Azure qui contient un jeu de données d’apprentissage. Consultez [Créer un jeu de données d’entraînement pour un modèle personnalisé](../../build-training-data-set.md) pour obtenir des conseils et des options pour constituer votre jeu de données d’entraînement. Dans le cadre de ce guide de démarrage rapide, vous pouvez utiliser les fichiers disponibles dans le dossier **Train** de l’[exemple de jeu de données](https://go.microsoft.com/fwlink/?linkid=2090451) (téléchargez et extrayez *sample_data.zip*).
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="créez une ressource Form Recognizer"  target="_blank">créer une ressource Form Recognizer<span class="docon docon-navigate-external x-hidden-focus"></span></a> sur le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Form Recognizer. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration

Dans une fenêtre de console (par exemple cmd, PowerShell ou Bash), utilisez la commande `dotnet new` pour créer une application console avec le nom `formrecognizer-quickstart`. Cette commande crée un projet C# simple nommé « Hello World » avec un seul fichier source : *program.cs*. 

```console
dotnet new console -n formrecognizer-quickstart
```

Déplacez vos répertoires vers le dossier d’application nouvellement créé. Vous pouvez générer l’application avec :

```console
dotnet build
```

La sortie de génération ne doit contenir aucun avertissement ni erreur. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Installer la bibliothèque de client 

Dans le répertoire de l’application, installez la bibliothèque de client Form Recognizer pour .NET avec la commande suivante :

#### <a name="version-30"></a>[version 3.0](#tab/ga)

```console
dotnet add package Azure.AI.FormRecognizer --version 3.0.0
```

#### <a name="version-31-preview"></a>[version 3.1 (préversion)](#tab/preview)

```console
dotnet add package Azure.AI.FormRecognizer --version 3.1.0-beta.1
```
---

> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.

À partir du répertoire de projet, ouvrez le fichier *Program.cs* dans votre éditeur ou votre IDE favori. Ajoutez les directives `using` suivantes :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_using)]

Dans la classe **Program** de l’application, créez des variables pour la clé et le point de terminaison de votre ressource.

> [!IMPORTANT]
> Accédez au portail Azure. Si la ressource Form Recognizer que vous avez créée dans la section **Prérequis** a été déployée, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. La clé et le point de terminaison se trouvent dans la page **Clé et point de terminaison** de la ressource, sous **Gestion des ressources**. 
>
> N’oubliez pas de supprimer la clé de votre code une fois que vous avez terminé, et ne la postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Pour plus d’informations, consultez l’article sur la [sécurité](../../../cognitive-services-security.md) de Cognitive Services.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_creds)]

Dans la méthode **Main** de l’application, ajoutez un appel aux tâches asynchrones utilisées dans ce guide de démarrage rapide. Vous les implémenterez ultérieurement.

#### <a name="version-30"></a>[version 3.0](#tab/ga)
[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_main)]
#### <a name="version-31-preview"></a>[version 3.1 (préversion)](#tab/preview)
[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_main)]

---


## <a name="object-model"></a>Modèle objet 

Avec Form Recognizer, vous pouvez créer deux types de client différents. Le premier, `FormRecognizerClient`, sert à interroger le service pour reconnaître les champs et le contenu de formulaires. Le deuxième, `FormTrainingClient`, sert à créer et gérer des modèles personnalisés que vous pouvez utiliser pour améliorer la reconnaissance. 

### <a name="formrecognizerclient"></a>FormRecognizerClient

`FormRecognizerClient` propose des opérations pour :

 - Reconnaître les champs et le contenu de formulaires à l’aide de modèles personnalisés entraînés pour reconnaître vos formulaires personnalisés.  Ces valeurs sont retournées dans une collection d’objets `RecognizedForm`. Consultez l’exemple [Analyser les formulaires personnalisés](#analyze-forms-with-a-custom-model).
 - Reconnaître le contenu des formulaires, notamment les tableaux, les lignes et les mots, sans avoir à entraîner un modèle.  Le contenu des formulaires est retourné dans une collection d’objets `FormPage`. Consultez l’exemple [Reconnaître le contenu des formulaires](#recognize-form-content).
 - Reconnaître les champs communs des tickets de caisse émis aux États-Unis, à l’aide d’un modèle de ticket de caisse préentraîné dans le service Form Recognizer. Ces champs et métadonnées sont retournés dans une collection d’objets `RecognizedForm`. Consultez l’exemple [Reconnaître les tickets de caisse](#recognize-receipts).

### <a name="formtrainingclient"></a>FormTrainingClient

`FormTrainingClient` propose des opérations pour :

- Entraîner les modèles personnalisés à reconnaître tous les champs et les valeurs rencontrés dans vos formulaires personnalisés.  Un `CustomFormModel` est retourné indiquant les types de formulaire que le modèle va reconnaître et les champs qu’il va extraire pour chaque type de formulaire.
- Entraîner les modèles personnalisés à reconnaître des champs et des valeurs spécifiques que vous spécifiez en étiquetant vos formulaires personnalisés.  Un `CustomFormModel` est retourné indiquant les champs que le modèle va extraire ainsi que la précision estimée pour chaque champ.
- Gérer les modèles créés dans votre compte.
- Copier un modèle personnalisé d’une ressource Form Recognizer vers une autre.

Consultez les exemples pour [entraîner un modèle](#train-a-custom-model) et [gérer des modèles personnalisés](#manage-custom-models).

> [!NOTE]
> Les modèles peuvent aussi être entraînés à partir d’une interface graphique utilisateur comme l’[outil d’étiquetage Form Recognizer](../../quickstarts/label-tool.md).

## <a name="code-examples"></a>Exemples de code

Ces extraits de code montrent comment effectuer les tâches suivantes avec la bibliothèque de client Form Recognizer pour .NET :

#### <a name="version-30"></a>[version 3.0](#tab/ga)

* [Authentifier le client](#authenticate-the-client)
* [Reconnaître le contenu d’un formulaire](#recognize-form-content)
* [Reconnaître les tickets de caisse](#recognize-receipts)
* [Entraîner un modèle personnalisé](#train-a-custom-model)
* [Analyser les formulaires avec un modèle personnalisé](#analyze-forms-with-a-custom-model)
* [Gérer vos modèles personnalisés](#manage-your-custom-models)

#### <a name="version-31-preview"></a>[version 3.1 (préversion)](#tab/preview)

* [Authentifier le client](#authenticate-the-client)
* [Reconnaître le contenu d’un formulaire](#recognize-form-content)
* [Reconnaître les tickets de caisse](#recognize-receipts)
* [Reconnaître les cartes de visite](#recognize-business-cards)
* [Reconnaître les factures](#recognize-invoices)
* [Entraîner un modèle personnalisé](#train-a-custom-model)
* [Analyser les formulaires avec un modèle personnalisé](#analyze-forms-with-a-custom-model)
* [Gérer vos modèles personnalisés](#manage-your-custom-models)

---

## <a name="authenticate-the-client"></a>Authentifier le client

En dessous de **Main**, créez une méthode sous le nom `AuthenticateClient`. Vous vous en servirez dans d’autres tâches pour authentifier vos demandes auprès du service Form Recognizer. Cette méthode utilise l’objet `AzureKeyCredential` pour que, le cas échéant, vous puissiez mettre à jour la clé API sans créer de nouveaux objets clients.

> [!IMPORTANT]
> Obtenez votre clé et votre point de terminaison à partir du portail Azure. Si la ressource Form Recognizer que vous avez créée dans la section **Prérequis** a été déployée, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. La clé et le point de terminaison se trouvent dans la page **Clé et point de terminaison** de la ressource, sous **Gestion des ressources**. 
>
> N’oubliez pas de supprimer la clé de votre code une fois que vous avez terminé, et ne la postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Par exemple, [Azure Key Vault](../../../../key-vault/general/overview.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_auth)]

Répétez les étapes ci-dessus pour une nouvelle méthode qui authentifie un client d’entraînement.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_auth_training)]

## <a name="get-assets-for-testing"></a>Obtenir des ressources à des fins de test 

Vous devrez aussi ajouter des références aux URL pour vos données d’entraînement et de test. Ajoutez-les à la racine de votre **Program**.

* Pour récupérer l’URL SAS des données d’entraînement de votre modèle personnalisé, ouvrez l’Explorateur Stockage Microsoft Azure, cliquez avec le bouton droit sur votre conteneur, puis sélectionnez **Obtenir une signature d’accès partagé**. Assurez-vous que les autorisations de **Lecture** et **Écriture** sont cochées, puis cliquez sur **Créer**. Copiez alors la valeur dans la section **URL**. Il doit avoir le format : `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
* Ensuite, utilisez les étapes ci-dessus pour obtenir l’URL SAS d’un document individuel dans le stockage d’objets blob.
* Enfin, enregistrez l’URL du ou des exemples d’image inclus ci-dessous (également disponible sur [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms)). 

#### <a name="version-30"></a>[version 3.0](#tab/ga)
[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_urls)]
#### <a name="version-31-preview"></a>[version 3.1 (préversion)](#tab/preview)
[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_urls)]

---


## <a name="recognize-form-content"></a>Reconnaître le contenu d’un formulaire

Vous pouvez utiliser Form Recognizer pour reconnaître les tables, les lignes et les mots dans les documents, sans avoir besoin d’entraîner un modèle. La valeur retournée est une collection d’objets **FormPage** : un pour chaque page du document envoyé. 

Pour reconnaître le contenu d’un fichier à une URL donnée, utilisez la méthode `StartRecognizeContentFromUri`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_getcontent_call)]

> [!TIP]
> Vous pouvez également obtenir le contenu à partir d’un fichier local. Consultez les méthodes [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient?view=azure-dotnet), telles que **StartRecognizeContent**. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) pour obtenir des scénarios impliquant des images locales.

Le reste de cette tâche affiche les informations du contenu dans la console.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_getcontent_print)]

### <a name="output"></a>Output

```console
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    Line 4 has 3 words, and text: '1020 Enterprise Way'.
    Line 5 has 3 words, and text: '6000 Redmond, WA'.
    Line 6 has 3 words, and text: 'Sunnayvale, CA 87659'.
    Line 7 has 1 word, and text: '99243'.
    Line 8 has 2 words, and text: 'Invoice Number'.
    Line 9 has 2 words, and text: 'Invoice Date'.
    Line 10 has 3 words, and text: 'Invoice Due Date'.
    Line 11 has 1 word, and text: 'Charges'.
    Line 12 has 2 words, and text: 'VAT ID'.
    Line 13 has 1 word, and text: '34278587'.
    Line 14 has 1 word, and text: '6/18/2017'.
    Line 15 has 1 word, and text: '6/24/2017'.
    Line 16 has 1 word, and text: '$56,651.49'.
    Line 17 has 1 word, and text: 'PT'.
Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    Cell (0, 3) contains text: 'Charges'.
    Cell (0, 5) contains text: 'VAT ID'.
    Cell (1, 0) contains text: '34278587'.
    Cell (1, 1) contains text: '6/18/2017'.
    Cell (1, 2) contains text: '6/24/2017'.
    Cell (1, 3) contains text: '$56,651.49'.
    Cell (1, 5) contains text: 'PT'.
```

## <a name="recognize-receipts"></a>Reconnaître les tickets de caisse

Cette section montre comment reconnaître et extraire les champs communs de tickets de caisse émis aux États-Unis à l’aide d’un modèle de ticket préentraîné.

Pour reconnaître les tickets de caisse à partir d’une URL, utilisez la méthode `StartRecognizeReceiptsFromUri`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_receipt_call)]

> [!TIP]
> Vous pouvez également reconnaître les images de tickets de caisse locales. Consultez les méthodes [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient?view=azure-dotnet), telles que **StartRecognizeReceipts**. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) pour obtenir des scénarios impliquant des images locales.

La valeur retournée est une collection d’objets `RecognizedReceipt` : un pour chaque page du document envoyé. Le code suivant traite le ticket à l’URI donné et affiche les champs principaux et les valeurs dans la console.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_receipt_print)]

### <a name="output"></a>Output 

```console
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    Line 4 has 3 words, and text: '1020 Enterprise Way'.
    Line 5 has 3 words, and text: '6000 Redmond, WA'.
    Line 6 has 3 words, and text: 'Sunnayvale, CA 87659'.
    Line 7 has 1 word, and text: '99243'.
    Line 8 has 2 words, and text: 'Invoice Number'.
    Line 9 has 2 words, and text: 'Invoice Date'.
    Line 10 has 3 words, and text: 'Invoice Due Date'.
    Line 11 has 1 word, and text: 'Charges'.
    Line 12 has 2 words, and text: 'VAT ID'.
    Line 13 has 1 word, and text: '34278587'.
    Line 14 has 1 word, and text: '6/18/2017'.
    Line 15 has 1 word, and text: '6/24/2017'.
    Line 16 has 1 word, and text: '$56,651.49'.
    Line 17 has 1 word, and text: 'PT'.
Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    Cell (0, 3) contains text: 'Charges'.
    Cell (0, 5) contains text: 'VAT ID'.
    Cell (1, 0) contains text: '34278587'.
    Cell (1, 1) contains text: '6/18/2017'.
    Cell (1, 2) contains text: '6/24/2017'.
    Cell (1, 3) contains text: '$56,651.49'.
    Cell (1, 5) contains text: 'PT'.
Merchant Name: 'Contoso Contoso', with confidence 0.516
Transaction Date: '6/10/2019 12:00:00 AM', with confidence 0.985
Item:
    Name: '8GB RAM (Black)', with confidence 0.916
    Total Price: '999', with confidence 0.559
Item:
    Name: 'SurfacePen', with confidence 0.858
    Total Price: '99.99', with confidence 0.386
Total: '1203.39', with confidence '0.774'
```

#### <a name="version-30"></a>[version 3.0](#tab/ga)

#### <a name="version-31-preview"></a>[version 3.1 (préversion)](#tab/preview)

## <a name="recognize-business-cards"></a>Reconnaître les cartes de visite

Cette section montre comment reconnaître et extraire les champs courants des cartes de visite en anglais en utilisant un modèle préentraîné.

Pour reconnaître des cartes de visite à partir d’une URL, utilisez la méthode `StartRecognizeBusinessCardsFromUriAsync`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_bc_call)]

> [!TIP]
> Vous pouvez également reconnaître les images de tickets de caisse locales. Examinez les méthodes de [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient?view=azure-dotnet), comme **StartRecognizeBusinessCards**. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) pour obtenir des scénarios impliquant des images locales.

La valeur retournée est une collection d’objets `RecognizedForm`, un pour chaque carte du document. Le code suivant traite la carte de visite à l’URI donné, et affiche les champs principaux et leurs valeurs sur la console.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_bc_print)]

## <a name="recognize-invoices"></a>Reconnaître les factures

Cette section montre comment reconnaître et extraire les champs communs de factures de vente en utilisant un modèle préentraîné.

Pour reconnaître des factures à partir d’une URL, utilisez la méthode `StartRecognizeInvoicesFromUriAsync`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_invoice_call)]

> [!TIP]
> Vous pouvez également reconnaître les images de factures locales. Examinez les méthodes de [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient?view=azure-dotnet), comme **StartRecognizeInvoices**. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) pour obtenir des scénarios impliquant des images locales.

La valeur retournée est une collection d’objets `RecognizedForm`, un pour chaque facture du document soumis. Le code suivant traite la facture à l’URI donné et affiche les champs principaux et leurs valeurs sur la console.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_invoice_print)]

---


## <a name="train-a-custom-model"></a>Entraîner un modèle personnalisé

Cette section montre comment entraîner un modèle avec vos propres données. Un modèle entraîné peut restituer des données structurées qui incluent les relations clé/valeur du document de formulaire d’origine. Une fois que le modèle est entraîné, vous pouvez le tester, le réentraîner et l’utiliser ensuite pour extraire de manière fiable des données d’autres formulaires, en fonction de vos besoins.

> [!NOTE]
> Vous pouvez aussi entraîner des modèles à l’aide d’une interface graphique utilisateur telle que l’[outil d’étiquetage des exemples Form Recognizer](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Entraîner un modèle sans étiquettes

Entraînez les modèles personnalisés à reconnaître tous les champs et valeurs rencontrés dans vos formulaires personnalisés sans étiqueter manuellement les documents d’entraînement. La méthode suivante entraîne un modèle sur un ensemble donné de documents et imprime l’état du modèle dans la console. 


[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_train)]


L’objet `CustomFormModel` retourné contient des informations sur les types de formulaires que le modèle peut reconnaître et les champs qu’il peut extraire de chaque type de formulaire. Le bloc de code suivant imprime ces informations dans la console.


[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_train_response)]

Enfin, retournez l’ID de modèle entraîné pour l’utiliser dans les étapes ultérieures.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_train_return)]

### <a name="output"></a>Output

Cette réponse a été tronquée dans un souci de lisibilité.

```console
Merchant Name: 'Contoso Contoso', with confidence 0.516
Transaction Date: '6/10/2019 12:00:00 AM', with confidence 0.985
Item:
    Name: '8GB RAM (Black)', with confidence 0.916
    Total Price: '999', with confidence 0.559
Item:
    Name: 'SurfacePen', with confidence 0.858
    Total Price: '99.99', with confidence 0.386
Total: '1203.39', with confidence '0.774'
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    Line 4 has 3 words, and text: '1020 Enterprise Way'.
    ...
Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    Cell (0, 3) contains text: 'Charges'.
    ... 
Custom Model Info:
    Model Id: 95035721-f19d-40eb-8820-0c806b42798b
    Model Status: Ready
    Training model started on: 8/24/2020 6:36:44 PM +00:00
    Training model completed on: 8/24/2020 6:36:50 PM +00:00
Submodel Form Type: form-95035721-f19d-40eb-8820-0c806b42798b
    FieldName: CompanyAddress
    FieldName: CompanyName
    FieldName: CompanyPhoneNumber
    ... 
Custom Model Info:
    Model Id: e7a1181b-1fb7-40be-bfbe-1ee154183633
    Model Status: Ready
    Training model started on: 8/24/2020 6:36:44 PM +00:00
    Training model completed on: 8/24/2020 6:36:52 PM +00:00
Submodel Form Type: form-0
    FieldName: field-0, FieldLabel: Additional Notes:
    FieldName: field-1, FieldLabel: Address:
    FieldName: field-2, FieldLabel: Company Name:
    FieldName: field-3, FieldLabel: Company Phone:
    FieldName: field-4, FieldLabel: Dated As:
    FieldName: field-5, FieldLabel: Details
    FieldName: field-6, FieldLabel: Email:
    FieldName: field-7, FieldLabel: Hero Limited
    FieldName: field-8, FieldLabel: Name:
    FieldName: field-9, FieldLabel: Phone:
    ...
```

### <a name="train-a-model-with-labels"></a>Entraîner un modèle avec des étiquettes

Vous pouvez aussi entraîner les modèles personnalisés en étiquetant manuellement les documents d’entraînement. L’entraînement avec étiquettes offre de meilleures performances dans certains scénarios. Pour effectuer un entraînement avec des étiquettes, vous devez disposer de fichiers d’informations spéciaux sur les étiquettes (`\<filename\>.pdf.labels.json`) dans votre conteneur de stockage d’objets blob, en même temps que les documents d’entraînement. L’[outil d’étiquetage des exemples Form Recognizer](../../quickstarts/label-tool.md) propose une interface utilisateur qui facilite la création de ces fichiers d’étiquettes. Une fois ceux-ci à disposition, vous pouvez appeler la méthode `StartTrainingAsync` avec le paramètre `uselabels` défini sur `true`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_trainlabels)]

Le `CustomFormModel` retourné indique les champs que le modèle peut extraire ainsi qu’une estimation de sa justesse dans chaque champ. Le bloc de code suivant imprime ces informations dans la console.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_trainlabels_response)]


### <a name="output"></a>Output

Cette réponse a été tronquée dans un souci de lisibilité.

```console
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    Line 4 has 3 words, and text: '1020 Enterprise Way'.
    Line 5 has 3 words, and text: '6000 Redmond, WA'.
    ...
Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    ... 
Merchant Name: 'Contoso Contoso', with confidence 0.516
Transaction Date: '6/10/2019 12:00:00 AM', with confidence 0.985
Item:
    Name: '8GB RAM (Black)', with confidence 0.916
    Total Price: '999', with confidence 0.559
Item:
    Name: 'SurfacePen', with confidence 0.858
    Total Price: '99.99', with confidence 0.386
Total: '1203.39', with confidence '0.774'
Custom Model Info:
    Model Id: 63c013e3-1cab-43eb-84b0-f4b20cb9214c
    Model Status: Ready
    Training model started on: 8/24/2020 6:42:54 PM +00:00
    Training model completed on: 8/24/2020 6:43:01 PM +00:00
Submodel Form Type: form-63c013e3-1cab-43eb-84b0-f4b20cb9214c
    FieldName: CompanyAddress
    FieldName: CompanyName
    FieldName: CompanyPhoneNumber
    FieldName: DatedAs
    FieldName: Email
    FieldName: Merchant
    ... 
```

## <a name="analyze-forms-with-a-custom-model"></a>Analyser les formulaires avec un modèle personnalisé

Cette section montre comment extraire des informations clé/valeur et d’autres contenus de vos types de formulaires personnalisés à l’aide de modèles que vous avez entraînés avec vos propres formulaires.

> [!IMPORTANT]
> Pour implémenter ce scénario, vous devez avoir déjà entraîné un modèle pour pouvoir passer son ID dans la méthode ci-dessous.

Vous allez utiliser la méthode `StartRecognizeCustomFormsFromUri`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_analyze)]

> [!TIP]
> Vous pouvez également analyser un fichier local. Consultez les méthodes [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient?view=azure-dotnet), telles que **StartRecognizeCustomForms**. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) pour obtenir des scénarios impliquant des images locales.

La valeur retournée est une collection d’objets `RecognizedForm` : un pour chaque page du document envoyé. Le code suivant imprime les résultats de l’analyse dans la console. Il imprime chaque champ reconnu et la valeur correspondante ainsi qu’un score de confiance.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_analyze_response)]


### <a name="output"></a>Output

Cette réponse a été tronquée dans un souci de lisibilité.

```console
Custom Model Info:
    Model Id: 9b0108ee-65c8-450e-b527-bb309d054fc4
    Model Status: Ready
    Training model started on: 8/24/2020 7:00:31 PM +00:00
    Training model completed on: 8/24/2020 7:00:32 PM +00:00
Submodel Form Type: form-9b0108ee-65c8-450e-b527-bb309d054fc4
    FieldName: CompanyAddress
    FieldName: CompanyName
    FieldName: CompanyPhoneNumber
    ...
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    ...

Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    ...
Merchant Name: 'Contoso Contoso', with confidence 0.516
Transaction Date: '6/10/2019 12:00:00 AM', with confidence 0.985
Item:
    Name: '8GB RAM (Black)', with confidence 0.916
    Total Price: '999', with confidence 0.559
Item:
    Name: 'SurfacePen', with confidence 0.858
    Total Price: '99.99', with confidence 0.386
Total: '1203.39', with confidence '0.774'
Custom Model Info:
    Model Id: dc115156-ce0e-4202-bbe4-7426e7bee756
    Model Status: Ready
    Training model started on: 8/24/2020 7:00:31 PM +00:00
    Training model completed on: 8/24/2020 7:00:41 PM +00:00
Submodel Form Type: form-0
    FieldName: field-0, FieldLabel: Additional Notes:
    FieldName: field-1, FieldLabel: Address:
    FieldName: field-2, FieldLabel: Company Name:
    FieldName: field-3, FieldLabel: Company Phone:
    FieldName: field-4, FieldLabel: Dated As:
    ... 
Form of type: custom:form
Field 'Azure.AI.FormRecognizer.Models.FieldValue:
    Value: '$56,651.49
    Confidence: '0.249
Field 'Azure.AI.FormRecognizer.Models.FieldValue:
    Value: 'PT
    Confidence: '0.245
Field 'Azure.AI.FormRecognizer.Models.FieldValue:
    Value: '99243
    Confidence: '0.114
   ...
```

## <a name="manage-custom-models"></a>Gérer des modèles personnalisés

Cette section explique comment gérer les modèles personnalisés stockés dans votre compte. Vous effectuerez plusieurs opérations dans la méthode suivante :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage)]


### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Vérifier le nombre de modèles dans le compte de ressource FormRecognizer

Le bloc de code suivant vérifie le nombre de modèles que vous avez enregistrés dans votre compte Form Recognizer et le compare à la limite du compte.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage_model_count)]

### <a name="output"></a>Output 

```console
Account has 20 models.
It can have at most 5000 models.
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Lister les modèles actuellement stockés dans le compte de ressource

Le bloc de code suivant liste les modèles actifs dans votre compte et imprime leurs détails dans la console.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage_model_list)]


### <a name="output"></a>Output 

Cette réponse a été tronquée dans un souci de lisibilité.

```console
Custom Model Info:
    Model Id: 05932d5a-a2f8-4030-a2ef-4e5ed7112515
    Model Status: Creating
    Training model started on: 8/24/2020 7:35:02 PM +00:00
    Training model completed on: 8/24/2020 7:35:02 PM +00:00
Custom Model Info:
    Model Id: 150828c4-2eb2-487e-a728-60d5d504bd16
    Model Status: Ready
    Training model started on: 8/24/2020 7:33:25 PM +00:00
    Training model completed on: 8/24/2020 7:33:27 PM +00:00
Custom Model Info:
    Model Id: 3303e9de-6cec-4dfb-9e68-36510a6ecbb2
    Model Status: Ready
    Training model started on: 8/24/2020 7:29:27 PM +00:00
    Training model completed on: 8/24/2020 7:29:36 PM +00:00
```

### <a name="get-a-specific-model-using-the-models-id"></a>Obtenir un modèle spécifique à l’aide de l’ID du modèle

Le bloc de code suivant entraîne un nouveau modèle (comme dans la section [Entraîner un modèle](#train-a-model-without-labels)), puis récupère une deuxième référence à celui-ci en utilisant son ID.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage_model_get)]

### <a name="output"></a>Output 

Cette réponse a été tronquée dans un souci de lisibilité.

```console
Custom Model Info:
    Model Id: 150828c4-2eb2-487e-a728-60d5d504bd16
    Model Status: Ready
    Training model started on: 8/24/2020 7:33:25 PM +00:00
    Training model completed on: 8/24/2020 7:33:27 PM +00:00
Submodel Form Type: form-150828c4-2eb2-487e-a728-60d5d504bd16
    FieldName: CompanyAddress
    FieldName: CompanyName
    FieldName: CompanyPhoneNumber
    FieldName: DatedAs
    FieldName: Email
    FieldName: Merchant
    FieldName: PhoneNumber
    FieldName: PurchaseOrderNumber
    FieldName: Quantity
    FieldName: Signature
    FieldName: Subtotal
    FieldName: Tax
    FieldName: Total
    FieldName: VendorName
    FieldName: Website
...
```

### <a name="delete-a-model-from-the-resource-account"></a>Supprimer un modèle du compte de ressource

Vous pouvez aussi supprimer un modèle de votre compte en référençant son ID. Cette étape ferme également la méthode.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage_model_delete)]


## <a name="run-the-application"></a>Exécuter l’application

Exécutez l’application à partir de votre répertoire d’application avec la commande `dotnet run`.

```dotnet
dotnet run
```


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Dépannage

Quand vous interagissez avec la bibliothèque de client Cognitive Services Form Recognizer en utilisant le kit SDK .NET, les erreurs retournées par le service génèrent une exception `RequestFailedException`. Elles comportent le même code d’état HTTP qui aurait été retourné par une demande d’API REST.

Par exemple, si vous envoyez une image de ticket avec un URI non valide, une erreur `400` est retournée, ce qui indique « Demande incorrecte ».

```csharp
try
{
    RecognizedReceiptCollection receipts = await client.StartRecognizeReceiptsFromUri(new Uri(receiptUri)).WaitForCompletionAsync();
}
catch (RequestFailedException e)
{
    Console.WriteLine(e.ToString());
}
```

Vous remarquerez que des informations supplémentaires, comme l’ID de demande client de l’opération, sont journalisées.

```
Message:
    Azure.RequestFailedException: Service request failed.
    Status: 400 (Bad Request)

Content:
    {"error":{"code":"FailedToDownloadImage","innerError":
    {"requestId":"8ca04feb-86db-4552-857c-fde903251518"},
    "message":"Failed to download image from input URL."}}

Headers:
    Transfer-Encoding: chunked
    x-envoy-upstream-service-time: REDACTED
    apim-request-id: REDACTED
    Strict-Transport-Security: REDACTED
    X-Content-Type-Options: REDACTED
    Date: Mon, 20 Apr 2020 22:48:35 GMT
    Content-Type: application/json; charset=utf-8
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez utilisé la bibliothèque de client .NET Form Recognizer pour entraîner des modèles et analyser des formulaires de différentes manières. Découvrez ensuite les astuces pour créer un jeu de données d’apprentissage plus performant et produire des modèles plus précis.

> [!div class="nextstepaction"]
> [Créer un jeu de données d’apprentissage](../../build-training-data-set.md)

* [Qu’est-ce que Form Recognizer ?](../../overview.md)
* Vous trouverez l’exemple de code de ce guide (et bien plus encore) sur [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md).