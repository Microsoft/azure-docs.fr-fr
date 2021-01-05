---
title: 'Démarrage rapide : Bibliothèque de client Form Recognizer pour Python'
description: Utilisez la bibliothèque de client Form Recognizer pour Python afin de créer une application de traitement des formulaires qui extrait des paires clé/valeur et des données de table de vos documents personnalisés.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 10/26/2020
ms.author: pafarley
ms.openlocfilehash: 12b49bf81328cc405337b682241aef2cd965c3de
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96356543"
---
> [!IMPORTANT]
> * Le code indiqué dans cet article utilise des méthodes synchrones et un stockage d’informations d’identification non sécurisé pour des raisons de simplicité. Consultez la documentation de référence ci-dessous. 

[Documentation de référence](/python/api/azure-ai-formrecognizer) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/azure/ai/formrecognizer) | [Package (PyPi)](https://pypi.org/project/azure-ai-formrecognizer/) | [Exemples](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples)

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* [Python 3.x](https://www.python.org/)
* Un blob Stockage Azure qui contient un jeu de données d’apprentissage. Consultez [Créer un jeu de données d’entraînement pour un modèle personnalisé](../../build-training-data-set.md) pour obtenir des conseils et des options pour constituer votre jeu de données d’entraînement. Dans le cadre de ce guide de démarrage rapide, vous pouvez utiliser les fichiers disponibles dans le dossier **Train** de l’[exemple de jeu de données](https://go.microsoft.com/fwlink/?linkid=2090451) (téléchargez et extrayez *sample_data.zip*).
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="créez une ressource Form Recognizer"  target="_blank">créer une ressource Form Recognizer<span class="docon docon-navigate-external x-hidden-focus"></span></a> sur le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Form Recognizer. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration

### <a name="install-the-client-library"></a>Installer la bibliothèque de client

Après avoir installé Python, vous pouvez installer la dernière version de la bibliothèque cliente Form Recognizer avec :

#### <a name="version-30"></a>[version 3.0](#tab/ga)

```console
pip install azure-ai-formrecognizer
```

#### <a name="version-31-preview"></a>[version 3.1 (préversion)](#tab/preview)

```console
pip install azure-ai-formrecognizer --pre
```

---

### <a name="create-a-new-python-application"></a>Créer une application Python

Créez une application Python dans votre éditeur ou IDE favori. Importez ensuite les bibliothèques suivantes.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_imports)]

> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/FormRecognizerQuickstart.py), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.


Créez des variables pour le point de terminaison et la clé Azure de votre ressource. 

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_creds)]


## <a name="object-model"></a>Modèle objet 

Avec Form Recognizer, vous pouvez créer deux types de client différents. Le premier, `form_recognizer_client`, sert à interroger le service pour reconnaître les champs et le contenu de formulaires. Le deuxième, `form_training_client`, sert à créer et gérer des modèles personnalisés que vous pouvez utiliser pour améliorer la reconnaissance. 

### <a name="formrecognizerclient"></a>FormRecognizerClient
`form_recognizer_client` propose des opérations pour :

 * Reconnaître les champs et le contenu de formulaires à l’aide de modèles personnalisés entraînés pour reconnaître vos formulaires personnalisés. 
 * Reconnaître le contenu des formulaires, notamment les tableaux, les lignes et les mots, sans avoir à entraîner un modèle. 
 * Reconnaître les champs communs des tickets de caisse, à l’aide d’un modèle de ticket de caisse préentraîné dans le service Form Recognizer.

### <a name="formtrainingclient"></a>FormTrainingClient
`form_training_client` propose des opérations pour :

* Entraîner les modèles personnalisés à reconnaître tous les champs et les valeurs rencontrés dans vos formulaires personnalisés. Pour obtenir des informations plus détaillées sur la création d’un jeu de données d’entraînement, consultez la [documentation du service sur l’entraînement d’un modèle sans étiquettes](#train-a-model-without-labels).
* Entraîner les modèles personnalisés à reconnaître des champs et des valeurs spécifiques que vous spécifiez en étiquetant vos formulaires personnalisés. Pour obtenir des informations plus détaillées sur l’application d’étiquettes à un jeu de données d’entraînement, consultez la [documentation du service sur l’entraînement d’un modèle avec étiquettes](#train-a-model-with-labels).
* Gérer les modèles créés dans votre compte.
* Copier un modèle personnalisé d’une ressource Form Recognizer vers une autre.

> [!NOTE]
> Les modèles peuvent aussi être entraînés à partir d’une interface graphique utilisateur comme l’[outil d’étiquetage Form Recognizer](../../quickstarts/label-tool.md).

## <a name="code-examples"></a>Exemples de code

Ces extraits de code montrent comment effectuer les tâches suivantes avec la bibliothèque de client Form Recognizer pour Python :

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

Ici, vous allez authentifier deux objets clients à l’aide des variables d’abonnement que vous avez définies ci-dessus. Vous allez utiliser un objet **AzureKeyCredential** pour que, le cas échéant, vous puissiez mettre à jour la clé d’API sans créer de nouveaux objets clients.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_auth)]


## <a name="get-assets-for-testing"></a>Obtenir des ressources à des fins de test

Vous devrez ajouter des références aux URL pour vos données d’entraînement et de test.
* Pour récupérer l’URL SAS des données d’entraînement de votre modèle personnalisé, ouvrez l’Explorateur Stockage Microsoft Azure, cliquez avec le bouton droit sur votre conteneur, puis sélectionnez **Obtenir une signature d’accès partagé**. Assurez-vous que les autorisations de **Lecture** et **Écriture** sont cochées, puis cliquez sur **Créer**. Copiez alors la valeur dans la section **URL**. Il doit avoir le format : `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
* Utilisez les exemples de formulaires et d’images de tickets de caisse fournis avec les exemples ci-dessous (également disponibles sur [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms)) ou utilisez les étapes ci-dessus pour récupérer l’URL SAS d’un document individuel dans le stockage d’objets blob. 

> [!NOTE]
> Les extraits de code présentés dans ce guide utilisent des formulaires distants accessibles par URL. Si vous voulez traiter des documents de formulaire locaux, consultez les méthodes correspondantes dans la [documentation de référence](/python/api/azure-ai-formrecognizer) et les [exemples](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples).

## <a name="recognize-form-content"></a>Reconnaître le contenu d’un formulaire

Vous pouvez utiliser Form Recognizer pour reconnaître les tables, les lignes et les mots dans les documents, sans avoir besoin d’entraîner un modèle.

Pour reconnaître le contenu d’un fichier à une URL donnée, utilisez la méthode `begin_recognize_content_from_url`. La valeur retournée est une collection d’objets `FormPage` : un pour chaque page du document envoyé. Le code suivant itère au sein de ces objets et imprime les paires clé/valeur extraites et les données de table.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_getcontent)]

> [!TIP]
> Vous pouvez également obtenir le contenu à partir d’images locales. Consultez les méthodes [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient?view=azure-python), telles que `begin_recognize_content`. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) pour obtenir des scénarios impliquant des images locales.

### <a name="output"></a>Output

```console
Table found on page 1:
Cell text: Invoice Number
Location: [Point(x=0.5075, y=2.8088), Point(x=1.9061, y=2.8088), Point(x=1.9061, y=3.3219), Point(x=0.5075, y=3.3219)]
Confidence score: 1.0

Cell text: Invoice Date
Location: [Point(x=1.9061, y=2.8088), Point(x=3.3074, y=2.8088), Point(x=3.3074, y=3.3219), Point(x=1.9061, y=3.3219)]
Confidence score: 1.0

Cell text: Invoice Due Date
Location: [Point(x=3.3074, y=2.8088), Point(x=4.7074, y=2.8088), Point(x=4.7074, y=3.3219), Point(x=3.3074, y=3.3219)]
Confidence score: 1.0

Cell text: Charges
Location: [Point(x=4.7074, y=2.8088), Point(x=5.386, y=2.8088), Point(x=5.386, y=3.3219), Point(x=4.7074, y=3.3219)]
Confidence score: 1.0
...

```

## <a name="recognize-receipts"></a>Reconnaître les tickets de caisse

Cette section montre comment reconnaître et extraire les champs communs de tickets de caisse émis aux États-Unis à l’aide d’un modèle de ticket préentraîné. Pour reconnaître les tickets de caisse à partir d’une URL, utilisez la méthode `begin_recognize_receipts_from_url`. 

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_receipts)]

> [!TIP]
> Vous pouvez également reconnaître les images de tickets de caisse locales. Consultez les méthodes [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient?view=azure-python), telles que `begin_recognize_receipts`. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) pour obtenir des scénarios impliquant des images locales.

### <a name="output"></a>Output

```console
ReceiptType: Itemized has confidence 0.659
MerchantName: Contoso Contoso has confidence 0.516
MerchantAddress: 123 Main Street Redmond, WA 98052 has confidence 0.986
MerchantPhoneNumber: None has confidence 0.99
TransactionDate: 2019-06-10 has confidence 0.985
TransactionTime: 13:59:00 has confidence 0.968
Receipt Items:
...Item #1
......Name: 8GB RAM (Black) has confidence 0.916
......TotalPrice: 999.0 has confidence 0.559
...Item #2
......Quantity: None has confidence 0.858
......Name: SurfacePen has confidence 0.858
......TotalPrice: 99.99 has confidence 0.386
Subtotal: 1098.99 has confidence 0.964
Tax: 104.4 has confidence 0.713
Total: 1203.39 has confidence 0.774
```

#### <a name="version-30"></a>[version 3.0](#tab/ga)

#### <a name="version-31-preview"></a>[version 3.1 (préversion)](#tab/preview)

## <a name="recognize-business-cards"></a>Reconnaître les cartes de visite

Cette section montre comment reconnaître et extraire les champs courants des cartes de visite en anglais en utilisant un modèle préentraîné. Pour reconnaître des cartes de visite à partir d’une URL, utilisez la méthode `begin_recognize_business_cards_from_url`. 

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart-preview.py?name=snippet_bc)]

> [!TIP]
> Vous pouvez également reconnaître les images de cartes de visite locales. Consultez les méthodes [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient?view=azure-python), telles que `begin_recognize_business_cards`. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) pour obtenir des scénarios impliquant des images locales.

## <a name="recognize-invoices"></a>Reconnaître les factures

Cette section montre comment reconnaître et extraire les champs communs de factures de vente en utilisant un modèle préentraîné. Pour reconnaître des factures à partir d’une URL, utilisez la méthode `begin_recognize_invoices_from_url`. 

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart-preview.py?name=snippet_invoice)]

> [!TIP]
> Vous pouvez également reconnaître les images de factures locales. Consultez les méthodes [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient?view=azure-python), telles que `begin_recognize_invoices`. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) pour obtenir des scénarios impliquant des images locales.

---

## <a name="train-a-custom-model"></a>Entraîner un modèle personnalisé

Cette section montre comment entraîner un modèle avec vos propres données. Un modèle entraîné peut restituer des données structurées qui incluent les relations clé/valeur du document de formulaire d’origine. Une fois que le modèle est entraîné, vous pouvez le tester, le réentraîner et l’utiliser ensuite pour extraire de manière fiable des données d’autres formulaires, en fonction de vos besoins.

> [!NOTE]
> Vous pouvez aussi entraîner des modèles à l’aide d’une interface graphique utilisateur telle que l’[outil d’étiquetage des exemples Form Recognizer](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Entraîner un modèle sans étiquettes

Entraînez les modèles personnalisés à reconnaître tous les champs et valeurs rencontrés dans vos formulaires personnalisés sans étiqueter manuellement les documents d’entraînement.

Le code suivant utilise le client d’entraînement avec la fonction `begin_training` pour entraîner un modèle sur un ensemble de documents donné. L’objet `CustomFormModel` retourné contient des informations sur les types de formulaires que le modèle peut reconnaître et les champs qu’il peut extraire de chaque type de formulaire. Le bloc de code suivant imprime ces informations dans la console.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_train)]


### <a name="output"></a>Output

Ceci est la sortie d’un modèle entraîné avec les données d’entraînement disponibles dans le [SDK Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training).

```console
Model ID: 628739de-779c-473d-8214-d35c72d3d4f7
Status: ready
Training started on: 2020-08-20 23:16:51+00:00
Training completed on: 2020-08-20 23:16:59+00:00

Recognized fields:
The submodel with form type 'form-0' has recognized the following fields: Additional Notes:, Address:, Company Name:, Company Phone:, Dated As:, Details, Email:, Hero Limited, Name:, Phone:, Purchase Order, Purchase Order #:, Quantity, SUBTOTAL, Seattle, WA 93849 Phone:, Shipped From, Shipped To, TAX, TOTAL, Total, Unit Price, Vendor Name:, Website:
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_4.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_5.jpg
Document status: succeeded
Document page count: 1
Document errors: []
```

### <a name="train-a-model-with-labels"></a>Entraîner un modèle avec des étiquettes

Vous pouvez aussi entraîner les modèles personnalisés en étiquetant manuellement les documents d’entraînement. L’entraînement avec étiquettes offre de meilleures performances dans certains scénarios. Le `CustomFormModel` retourné indique les champs que le modèle peut extraire ainsi qu’une estimation de sa justesse dans chaque champ. Le bloc de code suivant imprime ces informations dans la console.

> [!IMPORTANT]
> Pour effectuer un entraînement avec des étiquettes, vous devez disposer de fichiers d’informations spéciaux sur les étiquettes (`\<filename\>.pdf.labels.json`) dans votre conteneur de stockage d’objets blob, en même temps que les documents d’entraînement. L’[outil d’étiquetage des exemples Form Recognizer](../../quickstarts/label-tool.md) propose une interface utilisateur qui facilite la création de ces fichiers d’étiquettes. Une fois ceux-ci à disposition, vous pouvez appeler la méthode `begin_training` avec le paramètre *use_training_labels* défini sur `true`.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_trainlabels)]

### <a name="output"></a>Output

Ceci est la sortie d’un modèle entraîné avec les données d’entraînement disponibles dans le [SDK Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training).

```console
Model ID: ae636292-0b14-4e26-81a7-a0bfcbaf7c91

Status: ready
Training started on: 2020-08-20 23:20:56+00:00
Training completed on: 2020-08-20 23:20:57+00:00

Recognized fields:
The submodel with form type 'form-ae636292-0b14-4e26-81a7-a0bfcbaf7c91' has recognized the following fields: CompanyAddress, CompanyName, CompanyPhoneNumber, DatedAs, Email, Merchant, PhoneNumber, PurchaseOrderNumber, Quantity, Signature, Subtotal, Tax, Total, VendorName, Website
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_4.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_5.jpg
Document status: succeeded
Document page count: 1
Document errors: []
```

## <a name="analyze-forms-with-a-custom-model"></a>Analyser les formulaires avec un modèle personnalisé

Cette section montre comment extraire des informations clé/valeur et d’autres contenus de vos types de formulaires personnalisés à l’aide de modèles que vous avez entraînés avec vos propres formulaires.

> [!IMPORTANT]
> Pour implémenter ce scénario, vous devez avoir déjà entraîné un modèle pour pouvoir passer son ID dans la méthode ci-dessous. Consultez la section [Entraîner un modèle](#train-a-model-without-labels).

Vous allez utiliser la méthode `begin_recognize_custom_forms_from_url`. La valeur retournée est une collection d’objets `RecognizedForm` : un pour chaque page du document envoyé. Le code suivant imprime les résultats de l’analyse dans la console. Il imprime chaque champ reconnu et la valeur correspondante ainsi qu’un score de confiance.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_analyze)]

> [!TIP]
> Vous pouvez également analyser des images locales. Consultez les méthodes [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient?view=azure-python), telles que `begin_recognize_custom_forms`. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) pour obtenir des scénarios impliquant des images locales.


### <a name="output"></a>Output

En utilisant le modèle de l’exemple précédent, voici la sortie qui est fournie.

```console
Form type: form-ae636292-0b14-4e26-81a7-a0bfcbaf7c91
Field 'Merchant' has label 'Merchant' with value 'Invoice For:' and a confidence score of 0.116
Field 'CompanyAddress' has label 'CompanyAddress' with value '1 Redmond way Suite 6000 Redmond, WA' and a confidence score of 0.258
Field 'Website' has label 'Website' with value '99243' and a confidence score of 0.114
Field 'VendorName' has label 'VendorName' with value 'Charges' and a confidence score of 0.145
Field 'CompanyPhoneNumber' has label 'CompanyPhoneNumber' with value '$56,651.49' and a confidence score of 0.249
Field 'CompanyName' has label 'CompanyName' with value 'PT' and a confidence score of 0.245
Field 'DatedAs' has label 'DatedAs' with value 'None' and a confidence score of None
Field 'Email' has label 'Email' with value 'None' and a confidence score of None
Field 'PhoneNumber' has label 'PhoneNumber' with value 'None' and a confidence score of None
Field 'PurchaseOrderNumber' has label 'PurchaseOrderNumber' with value 'None' and a confidence score of None
Field 'Quantity' has label 'Quantity' with value 'None' and a confidence score of None
Field 'Signature' has label 'Signature' with value 'None' and a confidence score of None
Field 'Subtotal' has label 'Subtotal' with value 'None' and a confidence score of None
Field 'Tax' has label 'Tax' with value 'None' and a confidence score of None
Field 'Total' has label 'Total' with value 'None' and a confidence score of None
```

## <a name="manage-your-custom-models"></a>Gérer vos modèles personnalisés

Cette section explique comment gérer les modèles personnalisés stockés dans votre compte. 

### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Vérifier le nombre de modèles dans le compte de ressource FormRecognizer

Le bloc de code suivant vérifie le nombre de modèles que vous avez enregistrés dans votre compte Form Recognizer et le compare à la limite du compte.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_count)]


### <a name="output"></a>Output

```console
Our account has 5 custom models, and we can have at most 5000 custom models
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Lister les modèles actuellement stockés dans le compte de ressource

Le bloc de code suivant liste les modèles actifs dans votre compte et en imprime les détails dans la console. Il enregistre également une référence au premier modèle.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_list)]


### <a name="output"></a>Output

Ceci est un exemple de sortie pour le compte de test.

```console
We have models with the following ids:
453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
628739de-779c-473d-8214-d35c72d3d4f7
ae636292-0b14-4e26-81a7-a0bfcbaf7c91
b4b5df77-8538-4ffb-a996-f67158ecd305
c6309148-6b64-4fef-aea0-d39521452699
```

### <a name="get-a-specific-model-using-the-models-id"></a>Obtenir un modèle spécifique à l’aide de l’ID du modèle

Le bloc de code suivant utilise l’ID de modèle enregistré dans la section précédente et l’utilise pour récupérer des détails sur le modèle.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_getmodel)]


### <a name="output"></a>Output

Ceci est l’exemple de sortie pour le modèle personnalisé créé dans l’exemple précédent.

```console
Model ID: ae636292-0b14-4e26-81a7-a0bfcbaf7c91
Status: ready
Training started on: 2020-08-20 23:20:56+00:00
Training completed on: 2020-08-20 23:20:57+00:00
```

### <a name="delete-a-model-from-the-resource-account"></a>Supprimer un modèle du compte de ressource

Vous pouvez aussi supprimer un modèle de votre compte en référençant son ID. Ce code supprime le modèle utilisé dans la section précédente.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_delete)]


## <a name="run-the-application"></a>Exécution de l'application

Exécutez l’application avec la commande `python` de votre fichier de démarrage rapide.

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Dépannage

### <a name="general"></a>Général

La bibliothèque de client Form Recognizer lève les exceptions définies dans [Azure Core](https://aka.ms/azsdk-python-azure-core).

### <a name="logging"></a>Journalisation

Cette bibliothèque utilise la [bibliothèque de journalisation standard](https://docs.python.org/3/library/logging.html) pour la journalisation. Les informations de base sur les sessions HTTP (URL, en-têtes, etc.) sont journalisées au niveau INFO.

La journalisation détaillée de niveau DEBUG, comprenant le corps de la demande et/ou de la réponse et les en-têtes non rédigés, peut être activée sur un client à l’aide de l’argument de mot clé `logging_enable` :

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerLogging.py?name=snippet_logging)]


De la même façon, `logging_enable` peut activer la journalisation détaillée pour une seule opération, même quand elle n’est pas activée pour le client :

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerLogging.py?name=snippet_example)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez utilisé la bibliothèque de client Python Form Recognizer pour entraîner des modèles et analyser des formulaires de différentes manières. Découvrez ensuite les astuces pour créer un jeu de données d’apprentissage plus performant et produire des modèles plus précis.

> [!div class="nextstepaction"]
> [Créer un jeu de données d’apprentissage](../../build-training-data-set.md)

* [Qu’est-ce que Form Recognizer ?](../../overview.md)
* Vous trouverez l’exemple de code de ce guide sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/FormRecognizerQuickstart.py).