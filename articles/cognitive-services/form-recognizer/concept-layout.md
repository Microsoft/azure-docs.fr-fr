---
title: Dispositions - Form Recognizer
titleSuffix: Azure Cognitive Services
description: Découvrez les concepts liés à l’analyse des dispositions avec l’API Form Recognizer - utilisation et limites.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 11/18/2020
ms.author: pafarley
ms.openlocfilehash: 60ef490bbba4dca4f01b82967ad6ea8636e7bd4e
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353695"
---
# <a name="form-recognizer-layout-service"></a>Service Layout de Form Recognizer

Azure Form Recognizer peut extraire du texte, des tables, des marques de sélection et des informations de structure dans des documents à l’aide de son service Layout. L’API Layout permet aux clients de prendre des documents dans une multitude de formats et de retourner des données structurées et une représentation de ces documents. Elle combine nos puissantes fonctionnalités de [reconnaissance optique de caractères (OCR)](../computer-vision/concept-recognizing-text.md) avec des modèles de Deep Learning qui comprennent les documents dans le but d’en extraire du texte, des tables, des marques de sélection et la structure. 

## <a name="what-does-the-layout-service-do"></a>Comment fonctionne le service Layout ?

L’API Layout extrait du texte, des tables, des marques de sélection et des informations de structure dans les documents avec une précision exceptionnelle, puis les retourne dans une réponse JSON structurée et organisée. Les documents peuvent être dans divers formats et de diverses qualités. Il peut s’agir notamment d’images capturées par téléphone, de documents numérisés et de fichiers PDF numériques. L’API Layout va extraire la sortie structurée à partir de tous ces documents.

![Exemple de disposition](./media/layout-tool-example.JPG)

## <a name="try-it-out"></a>Faire un essai

Pour tester le service Layout de Form Recognizer, accédez à l’outil d’exemple d’interface utilisateur en ligne :

> [!div class="nextstepaction"]
> [Exemple d’interface utilisateur](https://fott-preview.azurewebsites.net/)

Vous aurez besoin d’un abonnement Azure ([créez-en un gratuitement](https://azure.microsoft.com/free/cognitive-services)), ainsi que d’un point de terminaison et d’une clé de [ressource Form Recognizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer) pour tester l’API Layout de Form Recognizer. 

![Capture d’écran de l’exemple d’interface utilisateur. Le texte, les tables et les marques de sélection d’un document sont analysés.](./media/analyze-layout.png)

### <a name="input-requirements"></a>Critères des entrées 

[!INCLUDE [input requirements](./includes/input-requirements-receipts.md)]

## <a name="the-analyze-layout-operation"></a>Opération d’analyse d’une disposition

L’opération d’[analyse d’une disposition](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeLayoutAsync) prend un document (fichier image, TIFF ou PDF) en entrée et en extrait le texte, les tables, les marques de sélection et la structure. L’appel retourne un champ d’en-tête de réponse appelé `Operation-Location`. La valeur `Operation-Location` est une URL qui contient l’ID de résultat à utiliser à l’étape suivante.

|En-tête de réponse| URL de résultat |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.1-preview.2/prebuilt/layout/analyzeResults/44a436324-fc4b-4387-aa06-090cfbf0064f` |

## <a name="the-get-analyze-layout-result-operation"></a>Opération d’obtention du résultat de l’analyse de la disposition

La seconde étape consiste à appeler l’opération d’[obtention du résultat de l’analyse de la disposition](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/GetAnalyzeLayoutResult). Cette opération prend en entrée l’ID de résultat créé par l’opération d’analyse de la disposition. Elle retourne une réponse JSON qui contient un champ **État** avec les possibles valeurs suivantes. 

|Champ| Type | Valeurs possibles |
|:-----|:----:|:----|
|status | string | `notStarted` : L’opération d’analyse n’a pas commencé.<br /><br />`running` : L’opération d’analyse est en cours.<br /><br />`failed` : L’opération d’analyse a échoué.<br /><br />`succeeded` : L’opération d’analyse a réussi.|

Vous appelez cette opération de façon itérative jusqu’à ce qu’elle retourne avec la valeur `succeeded`. Utilisez un intervalle de 3 à 5 secondes pour éviter de dépasser le taux de demandes par seconde (RPS).

Quand le champ **status** a la valeur `succeeded`, la réponse JSON inclut les résultats de l’extraction de la disposition, à savoir le texte, les tables et les marques de sélection extraits. Les données extraites contiennent les lignes de texte et les mots extraits, le cadre englobant, l’indication manuscrite ressemblant à du texte, les tables et les marques de sélection avec une indication de sélection/non-sélection. 

### <a name="sample-json-output"></a>Exemple de sortir JSON

La réponse à l’opération d’obtention du résultat de l’analyse de la disposition va correspondre à la représentation structurée du document avec toutes les informations extraites. Reportez-vous à cet [exemple de fichier de document](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/curl/form-recognizer/sample-layout.pdf) et cet [exemple de sortie de disposition](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/curl/form-recognizer/sample-layout-output.json) structurée.

La sortie JSON comporte deux parties : 
* Le nœud `"readResults"` contient tout le texte reconnu et toutes les marques de sélection. Le texte est organisé par page, puis par ligne, puis par mots individuels. 
* Le nœud `"pageResults"` contient les tables et les cellules extraites avec leurs cadres englobants, la confiance et une référence aux lignes et aux mots dans « readResults ».

## <a name="example-output"></a>Exemple de sortie

### <a name="text"></a>Texte

Layout extrait le texte des documents (PDF, TIFF) et des images (jpg, png, bmp) et reconnaît des angles de texte, couleurs, angles, photos de documents, télécopies, impressions, entrées manuscrites (en anglais uniquement) et modes mixtes différents. Le texte est extrait avec des informations sur les lignes, les mots, les cadres englobants, les scores de confiance et le style (manuscrit ou autre). Toutes les informations textuelles sont incluses dans la section `"readResults"` de la sortie JSON. 

### <a name="tables"></a>Tables

Layout extrait les tables des documents (PDF, TIFF) et des images (jpg, png, bmp). Les documents peuvent être scannés, photographiés ou numériques. Les tables peuvent être des tables complexes avec des cellules ou colonnes fusionnées, avec ou sans bordures et avec des angles irréguliers. Les tables extraites incluent le nombre de colonnes et de lignes, l’étendue de ligne et l’étendue de colonne. Chaque cellule est extraite avec son cadre englobant et sa référence au texte extrait dans la section `"readResults"`. Les informations de table se trouvent dans la section `"pageResults"` de la sortie JSON. 

![Exemple de table](./media/tables-example.jpg)

### <a name="selection-marks"></a>Marques de sélection

Layout extrait aussi les marques de sélection des documents. Les marques de sélection extraites incluent le cadre englobant, la confiance et l’état (sélection/non sélectionné). Les informations sur les marques de sélection sont extraites dans la section `"readResults"` de la sortie JSON. 

## <a name="next-steps"></a>Étapes suivantes

- Testez l’extraction de votre propre disposition à l’aide de l’[exemple d’interface utilisateur de Form Recognizer](https://fott-preview.azurewebsites.net/).
- Suivez un guide de [démarrage rapide de la bibliothèque de client Form Recognizer](quickstarts/client-library.md) pour commencer à extraire des dispositions dans le langage de votre choix.
- Ou, suivez le guide de démarrage rapide sur l’[extraction de données de disposition](./QuickStarts/python-layout.md) pour implémenter une telle extraction à l’aide de Python et de l’API REST.

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que Form Recognizer ?](./overview.md)
* [Documentation de référence sur l’API REST](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeLayoutAsync)