---
title: 'Démarrage rapide : Bibliothèque de client Form Recognizer pour JavaScript'
description: Utilisez la bibliothèque de client Form Recognizer pour JavaScript afin de créer une application de traitement des formulaires qui extrait des paires clé/valeur et des données de table de vos documents personnalisés.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 10/26/2020
ms.author: pafarley
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 9b6fc05cc2e97e8e0d74f7d19912afedb37c0fa5
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97366531"
---
> [!IMPORTANT]
> * Le code indiqué dans cet article utilise des méthodes synchrones et un stockage d’informations d’identification non sécurisé pour des raisons de simplicité. Consultez la documentation de référence ci-dessous. 

[Documentation de référence](../../index.yml) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/formrecognizer/ai-form-recognizer/) | [Package (npm)](https://www.npmjs.com/package/@azure/ai-form-recognizer) | [Exemples](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples)

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* Version actuelle de [Node.js](https://nodejs.org/)
* Un blob Stockage Azure qui contient un jeu de données d’apprentissage. Consultez [Créer un jeu de données d’entraînement pour un modèle personnalisé](../../build-training-data-set.md) pour obtenir des conseils et des options pour constituer votre jeu de données d’entraînement. Dans le cadre de ce guide de démarrage rapide, vous pouvez utiliser les fichiers disponibles dans le dossier **Train** de l’[exemple de jeu de données](https://go.microsoft.com/fwlink/?linkid=2090451) (téléchargez et extrayez *sample_data.zip*).
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="créez une ressource Form Recognizer"  target="_blank">créer une ressource Form Recognizer<span class="docon docon-navigate-external x-hidden-focus"></span></a> sur le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Form Recognizer. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-nodejs-application"></a>Création d’une application Node.js

Dans une fenêtre de console (telle que cmd, PowerShell ou bash), créez un répertoire pour votre application et accédez-y. 

```console
mkdir myapp && cd myapp
```

Exécutez la commande `npm init` pour créer une application de nœud avec un fichier `package.json`. 

```console
npm init
```

### <a name="install-the-client-library"></a>Installer la bibliothèque de client

Installez le package NPM `ai-form-recognizer` :

```console
npm install @azure/ai-form-recognizer
```

Le fichier `package.json` de votre application sera mis à jour avec les dépendances.

Créez un fichier sous le nom `index.js`, ouvrez-le, puis importez les bibliothèques suivantes :

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_imports)]


> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/FormRecognizer/FormRecognizerQuickstart.js), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.

Créez des variables pour le point de terminaison et la clé Azure de votre ressource. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_creds)]

> [!IMPORTANT]
> Accédez au portail Azure. Si la ressource [nom du produit] que vous avez créée dans la section **Prérequis** a été déployée, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. La clé et le point de terminaison se trouvent dans la page **Clé et point de terminaison** de la ressource, sous **Gestion des ressources**. 
>
> N’oubliez pas de supprimer la clé de votre code une fois que vous avez terminé, et ne la postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Pour plus d’informations, consultez l’article sur la [sécurité](../../../cognitive-services-security.md) de Cognitive Services.

## <a name="object-model"></a>Modèle objet 

Avec Form Recognizer, vous pouvez créer deux types de client différents. Le premier, `FormRecognizerClient`, sert à interroger le service pour reconnaître les champs et le contenu de formulaires. Le deuxième, `FormTrainingClient`, sert à créer et gérer des modèles personnalisés que vous pouvez utiliser pour améliorer la reconnaissance. 

### <a name="formrecognizerclient"></a>FormRecognizerClient
`FormRecognizerClient` propose des opérations pour :

 * Reconnaître les champs et le contenu de formulaires à l’aide de modèles personnalisés entraînés pour reconnaître vos formulaires personnalisés. Ces valeurs sont retournées dans une collection d’objets `RecognizedForm`.
 * Reconnaître le contenu des formulaires, notamment les tableaux, les lignes et les mots, sans avoir à entraîner un modèle. Le contenu des formulaires est retourné dans une collection d’objets `FormPage`.
 * Reconnaître les champs communs des tickets de caisse, à l’aide d’un modèle de ticket de caisse préentraîné dans le service Form Recognizer. Ces champs et métadonnées sont retournés dans une collection de `RecognizedReceipt`.

### <a name="formtrainingclient"></a>FormTrainingClient
`FormTrainingClient` propose des opérations pour :

* Entraîner les modèles personnalisés à reconnaître tous les champs et les valeurs rencontrés dans vos formulaires personnalisés. Un `CustomFormModel` est retourné indiquant les types de formulaire que le modèle va reconnaître et les champs qu’il va extraire pour chaque type de formulaire. Pour obtenir des informations plus détaillées sur la création d’un jeu de données d’entraînement, consultez la [documentation du service sur l’entraînement d’un modèle sans étiquettes](#train-a-model-without-labels).
* Entraîner les modèles personnalisés à reconnaître des champs et des valeurs spécifiques que vous spécifiez en étiquetant vos formulaires personnalisés. Un `CustomFormModel` est retourné indiquant les champs que le modèle va extraire ainsi que la précision estimée pour chaque champ. Pour obtenir des informations plus détaillées sur l’application d’étiquettes à un jeu de données d’entraînement, consultez la [documentation du service sur l’entraînement d’un modèle avec étiquettes](#train-a-model-with-labels).
* Gérer les modèles créés dans votre compte.
* Copier un modèle personnalisé d’une ressource Form Recognizer vers une autre.

> [!NOTE]
> Les modèles peuvent aussi être entraînés à partir d’une interface graphique utilisateur comme l’[outil d’étiquetage Form Recognizer](../../quickstarts/label-tool.md).


## <a name="code-examples"></a>Exemples de code

Ces extraits de code montrent comment effectuer les tâches suivantes avec la bibliothèque de client Form Recognizer pour JavaScript :

* [Authentifier le client](#authenticate-the-client)
* [Reconnaître le contenu d’un formulaire](#recognize-form-content)
* [Reconnaître les tickets de caisse](#recognize-receipts)
* [Entraîner un modèle personnalisé](#train-a-custom-model)
* [Analyser les formulaires avec un modèle personnalisé](#analyze-forms-with-a-custom-model)
* [Gérer vos modèles personnalisés](#manage-your-custom-models)

## <a name="authenticate-the-client"></a>Authentifier le client



Authentifiez un objet client à l’aide des variables d’abonnement que vous avez définies. Vous allez utiliser un objet `AzureKeyCredential` pour que, le cas échéant, vous puissiez mettre à jour la clé d’API sans créer de nouveaux objets clients. Vous allez aussi créer un objet client d’entraînement.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_auth)]


## <a name="get-assets-for-testing"></a>Obtenir des ressources à des fins de test

Vous devrez aussi ajouter des références aux URL pour vos données d’entraînement et de test.
* [!INCLUDE [get SAS URL](../../includes/sas-instructions.md)]
* Utilisez les exemples de formulaires et d’images de tickets de caisse fournis avec les exemples ci-dessous (également disponibles sur [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/test-assets)) ou utilisez les étapes ci-dessus pour récupérer l’URL SAS d’un document individuel dans le stockage d’objets blob. 


## <a name="recognize-form-content"></a>Reconnaître le contenu d’un formulaire

Vous pouvez utiliser Form Recognizer pour reconnaître les tables, les lignes et les mots dans les documents, sans avoir besoin d’entraîner un modèle. Pour reconnaître le contenu d’un fichier à un URI donné, utilisez la méthode `beginRecognizeContentFromUrl`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_getcontent)]


> [!TIP]
> Vous pouvez également obtenir le contenu à partir d’un fichier local. Consultez les méthodes [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient?view=azure-node-latest), telles que **beginRecognizeContent**. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) pour obtenir des scénarios impliquant des images locales.

### <a name="output"></a>Output

```console
Page 1: width 8.5 and height 11 with unit inch
cell [0,0] has text Invoice Number
cell [0,1] has text Invoice Date
cell [0,2] has text Invoice Due Date
cell [0,3] has text Charges
cell [0,5] has text VAT ID
cell [1,0] has text 34278587
cell [1,1] has text 6/18/2017
cell [1,2] has text 6/24/2017
cell [1,3] has text $56,651.49
cell [1,5] has text PT
```

## <a name="recognize-receipts"></a>Reconnaître les tickets de caisse

Cette section montre comment reconnaître et extraire les champs communs de tickets de caisse émis aux États-Unis à l’aide d’un modèle de ticket préentraîné.

Pour reconnaître les tickets à partir d’un URI, utilisez la méthode `beginRecognizeReceiptsFromUrl`. Le code suivant traite un ticket à l’URI donné et imprime les champs principaux et les valeurs dans la console.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_receipts)]

> [!TIP]
> Vous pouvez également reconnaître les images de tickets de caisse locales. Consultez les méthodes [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient?view=azure-node-latest), telles que **beginRecognizeReceipts**. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) pour obtenir des scénarios impliquant des images locales.

### <a name="output"></a>Output

```console
status: notStarted
status: running
status: succeeded
First receipt:
  Receipt Type: 'Itemized', with confidence of 0.659
  Merchant Name: 'Contoso Contoso', with confidence of 0.516
  Transaction Date: 'Sun Jun 09 2019 17:00:00 GMT-0700 (Pacific Daylight Time)', with confidence of 0.985
    Item Name: '8GB RAM (Black)', with confidence of 0.916
    Item Name: 'SurfacePen', with confidence of 0.858
  Total: '1203.39', with confidence of 0.774
```

## <a name="train-a-custom-model"></a>Entraîner un modèle personnalisé

Cette section montre comment entraîner un modèle avec vos propres données. Un modèle entraîné peut restituer des données structurées qui incluent les relations clé/valeur du document de formulaire d’origine. Une fois que le modèle est entraîné, vous pouvez le tester, le réentraîner et l’utiliser ensuite pour extraire de manière fiable des données d’autres formulaires, en fonction de vos besoins.

> [!NOTE]
> Vous pouvez aussi entraîner des modèles à l’aide d’une interface graphique utilisateur telle que l’[outil d’étiquetage des exemples Form Recognizer](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Entraîner un modèle sans étiquettes

Entraînez les modèles personnalisés à reconnaître tous les champs et valeurs rencontrés dans vos formulaires personnalisés sans étiqueter manuellement les documents d’entraînement.

La fonction suivante entraîne un modèle sur un ensemble donné de documents et imprime l’état du modèle dans la console. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_train)]


### <a name="output"></a>Output

C’est la sortie d’un modèle entraîné avec les données d’entraînement disponibles dans le [SDK JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer). Cet exemple de sortie a été tronqué dans un souci de lisibilité.

```console
training status: creating
training status: ready
Model ID: 9d893595-1690-4cf2-a4b1-fbac0fb11909
Status: ready
Training started on: Thu Aug 20 2020 20:27:26 GMT-0700 (Pacific Daylight Time)
Training completed on: Thu Aug 20 2020 20:27:37 GMT-0700 (Pacific Daylight Time)
We have recognized the following fields
The model found field 'field-0'
The model found field 'field-1'
The model found field 'field-2'
The model found field 'field-3'
The model found field 'field-4'
The model found field 'field-5'
The model found field 'field-6'
The model found field 'field-7'
...
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: 
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: 
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: 
...
```

### <a name="train-a-model-with-labels"></a>Entraîner un modèle avec des étiquettes

Vous pouvez aussi entraîner les modèles personnalisés en étiquetant manuellement les documents d’entraînement. L’entraînement avec étiquettes offre de meilleures performances dans certains scénarios. Pour effectuer un entraînement avec des étiquettes, vous devez disposer de fichiers d’informations spéciaux sur les étiquettes (`\<filename\>.pdf.labels.json`) dans votre conteneur de stockage d’objets blob, en même temps que les documents d’entraînement. L’[outil d’étiquetage des exemples Form Recognizer](../../quickstarts/label-tool.md) propose une interface utilisateur qui facilite la création de ces fichiers d’étiquettes. Une fois ceux-ci à disposition, vous pouvez appeler la méthode `beginTraining` avec le paramètre `uselabels` défini sur `true`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_trainlabels)]


### <a name="output"></a>Output 

C’est la sortie d’un modèle entraîné avec les données d’entraînement disponibles dans le [SDK JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples). Cet exemple de sortie a été tronqué dans un souci de lisibilité.

```console
training status: creating
training status: ready
Model ID: 789b1b37-4cc3-4e36-8665-9dde68618072
Status: ready
Training started on: Thu Aug 20 2020 20:30:37 GMT-0700 (Pacific Daylight Time)
Training completed on: Thu Aug 20 2020 20:30:43 GMT-0700 (Pacific Daylight Time)
We have recognized the following fields
The model found field 'CompanyAddress'
The model found field 'CompanyName'
The model found field 'CompanyPhoneNumber'
The model found field 'DatedAs'
...
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
...
```

## <a name="analyze-forms-with-a-custom-model"></a>Analyser les formulaires avec un modèle personnalisé

Cette section montre comment extraire des informations clé/valeur et d’autres contenus de vos types de formulaires personnalisés à l’aide de modèles que vous avez entraînés avec vos propres formulaires.

> [!IMPORTANT]
> Pour implémenter ce scénario, vous devez avoir déjà entraîné un modèle pour pouvoir passer son ID dans la méthode ci-dessous. Consultez la section [Entraîner un modèle](#train-a-model-without-labels).

Vous allez utiliser la méthode `beginRecognizeCustomFormsFromUrl`. La valeur retournée est une collection d’objets `RecognizedForm` : un pour chaque page du document envoyé.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_analyze)]

> [!TIP]
> Vous pouvez également analyser des fichiers locaux. Consultez les méthodes [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient?view=azure-node-latest), telles que **beginRecognizeCustomForms**. Ou consultez l’exemple de code sur [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) pour obtenir des scénarios impliquant des images locales.


### <a name="output"></a>Output

```console
status: notStarted
status: succeeded
Forms:
custom:form, page range: [object Object]
Pages:
Page number: 1
Tables
cell (0,0) Invoice Number
cell (0,1) Invoice Date
cell (0,2) Invoice Due Date
cell (0,3) Charges
cell (0,5) VAT ID
cell (1,0) 34278587
cell (1,1) 6/18/2017
cell (1,2) 6/24/2017
cell (1,3) $56,651.49
cell (1,5) PT
Fields:
Field Merchant has value 'Invoice For:' with a confidence score of 0.116
Field CompanyPhoneNumber has value '$56,651.49' with a confidence score of 0.249
Field VendorName has value 'Charges' with a confidence score of 0.145
Field CompanyAddress has value '1 Redmond way Suite 6000 Redmond, WA' with a confidence score of 0.258
Field CompanyName has value 'PT' with a confidence score of 0.245
Field Website has value '99243' with a confidence score of 0.114
Field DatedAs has value 'undefined' with a confidence score of undefined
Field Email has value 'undefined' with a confidence score of undefined
Field PhoneNumber has value 'undefined' with a confidence score of undefined
Field PurchaseOrderNumber has value 'undefined' with a confidence score of undefined
Field Quantity has value 'undefined' with a confidence score of undefined
Field Signature has value 'undefined' with a confidence score of undefined
Field Subtotal has value 'undefined' with a confidence score of undefined
Field Tax has value 'undefined' with a confidence score of undefined
Field Total has value 'undefined' with a confidence score of undefined
```

## <a name="manage-your-custom-models"></a>Gérer vos modèles personnalisés

Cette section explique comment gérer les modèles personnalisés stockés dans votre compte. Le code suivant effectue toutes les tâches de gestion de modèles dans une même fonction, à titre d’exemple. 

### <a name="get-number-of-models"></a>Obtenir le nombre de modèles

Le bloc de code suivant obtient le nombre de modèles présents dans votre compte.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_count)]


### <a name="get-list-of-models-in-account"></a>Obtenir la liste des modèles du compte

Le bloc de code suivant fournit la liste complète des modèles disponibles dans votre compte, notamment des informations sur la date de création du modèle et son état actuel.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_list)]


### <a name="output"></a>Output

```console
model 0:
{
  modelId: '453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e',
  status: 'invalid',
  trainingStartedOn: 2020-08-20T22:28:52.000Z,
  trainingCompletedOn: 2020-08-20T22:28:53.000Z
}
model 1:
{
  modelId: '628739de-779c-473d-8214-d35c72d3d4f7',
  status: 'ready',
  trainingStartedOn: 2020-08-20T23:16:51.000Z,
  trainingCompletedOn: 2020-08-20T23:16:59.000Z
}
model 2:
{
  modelId: '789b1b37-4cc3-4e36-8665-9dde68618072',
  status: 'ready',
  trainingStartedOn: 2020-08-21T03:30:37.000Z,
  trainingCompletedOn: 2020-08-21T03:30:43.000Z
}
model 3:
{
  modelId: '9d893595-1690-4cf2-a4b1-fbac0fb11909',
  status: 'ready',
  trainingStartedOn: 2020-08-21T03:27:26.000Z,
  trainingCompletedOn: 2020-08-21T03:27:37.000Z
}
```

### <a name="get-list-of-model-ids-by-page"></a>Obtenir la liste des ID de modèle par page

Ce bloc de code fournit une liste paginée des modèles et des ID de modèle.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_listpages)]


### <a name="output"></a>Output

```console
model 1: 453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
model 2: 628739de-779c-473d-8214-d35c72d3d4f7
model 3: 789b1b37-4cc3-4e36-8665-9dde68618072
```

### <a name="get-model-by-id"></a>Obtient le modèle par ID

La fonction suivante accepte un ID de modèle et obtient l’objet de modèle correspondant. Cette fonction n’est pas appelée par défaut.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_getmodel)]


### <a name="delete-a-model-from-the-resource-account"></a>Supprimer un modèle du compte de ressource

Vous pouvez aussi supprimer un modèle de votre compte en référençant son ID. Cette fonction supprime le modèle portant l’ID spécifié. Cette fonction n’est pas appelée par défaut.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_delete)]


### <a name="output"></a>Output

```console
Model with id 789b1b37-4cc3-4e36-8665-9dde68618072 has been deleted
```

## <a name="run-the-application"></a>Exécution de l'application

Exécutez l’application avec la commande `node` de votre fichier de démarrage rapide.

```console
node index.js
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Dépannage

### <a name="enable-logs"></a>Activer les journaux d’activité

Vous pouvez définir la variable d’environnement suivante pour afficher les journaux de débogage quand vous utilisez cette bibliothèque.

```
export DEBUG=azure*
```

Pour obtenir des instructions plus détaillées sur l’activation des journaux, consultez les [documents sur le package @azure/logger](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/core/logger).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez utilisé la bibliothèque de client JavaScript Form Recognizer pour entraîner des modèles et analyser des formulaires de différentes manières. Découvrez ensuite les astuces pour créer un jeu de données d’apprentissage plus performant et produire des modèles plus précis.

> [!div class="nextstepaction"]
> [Créer un jeu de données d’apprentissage](../../build-training-data-set.md)

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que Form Recognizer ?](../../overview.md)
* Vous trouverez l’exemple de code de ce guide sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/FormRecognizer/FormRecognizerQuickstart.js).