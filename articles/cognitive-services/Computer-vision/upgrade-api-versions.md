---
title: Effectuer une mise à niveau de l’API Vision par ordinateur vers Read v3.0
titleSuffix: Azure Cognitive Services
description: Découvrez comment effectuer une mise à niveau vers l’API de lecture Vision par ordinateur v3.0 à partir de la version v2.0/v2.1.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 08/11/2020
ms.author: pafarley
ROBOTS: NOINDEX
ms.openlocfilehash: cfc9745fc4684a7b0d8f7da7e63149a6fe50f6d2
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92331836"
---
# <a name="upgrade-from-read-v2x-to-read-v3x"></a>Effectuer une mise à niveau de Read v2.x vers Read v3.x

Ce guide vous montre comment mettre à niveau votre code d’API cloud ou conteneur existant de Read v2.x vers Read v3.0 et v3.1 (préversion).

## <a name="determine-your-api-path"></a>Déterminer le chemin de votre API
Utilisez le tableau suivant pour déterminer la **chaîne de version** dans le chemin d’API en fonction de la version de Read 3.x vers laquelle vous effectuez la migration.

|Type de produit| Version | Chaîne de version dans le chemin d’API 3.x |
|:-----|:----|:----|
|Service | Read 3.0 ou 3.1 | **v3.0** ou **v3.1** respectivement |
|Service | Read 3.2 (préversion) | **v3.2-preview.1** |
|Conteneur | Read 3.0 (préversion) ou Read 3.1 (préversion) | **v3.0** ou **v3.1-preview.2** respectivement |


Ensuite, utilisez les sections suivantes pour affiner vos opérations et remplacer la **chaîne de version** dans votre chemin d’API par la valeur de la table. Par exemple, pour les versions cloud et conteneur de **Read v3.2 (préversion)** , remplacez le chemin d’API par **https://{endpoint}/vision/v3.2-preview.1/read/analyze[?language]** .

## <a name="servicecontainer"></a>Service/conteneur

### `Batch Read File`

|Read 2.x |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/ **v2.0/read/core/asyncBatchAnalyze**     |https://{endpoint}/vision/< **chaîne de version** >/read/analyze[?language]|
    
Un nouveau paramètre facultatif de _langue_ est disponible. Si vous ne connaissez pas la langue de votre document ou s’il est multilingue, ne l’incluez pas. 

### `Get Read Results`

|Read 2.x |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/ **v2.0/read/operations** /{operationId}     |https://{endpoint}/vision/< **chaîne de version** >/read/analyzeResults/{operationId}|

### <a name="get-read-operation-result-status-flag"></a>Indicateur d’état `Get Read Operation Result`

Lorsque l’appel de `Get Read Operation Result` aboutit, il renvoie un champ de chaîne d’état dans le corps du json.
 
|Read 2.x |Read 3.x  |
|----------|-----------|
|`"NotStarted"` |    `"notStarted"`|
|`"Running"` | `"running"`|
|`"Failed"` | `"failed"`|
|`"Succeeded"` | `"succeeded"`|
    
### <a name="api-response-json"></a>Réponse d’API (JSON) 

Notez les modifications suivantes apportées au json :
* Dans la version v2.x, `Get Read Operation Result` renvoie le json de reconnaissance OCR lorsque l’état est `Succeeded"`. Dans la version v3.0, ce champ est `succeeded`.
* Pour obtenir la racine du tableau de pages, remplacez la hiérarchie json `recognitionResults` par `analyzeResult`/`readResults`. La hiérarchie json des mots et lignes par page reste inchangée et dès lors, aucune modification de code n’est requise.
* L’angle de page `clockwiseOrientation` a été renommé par `angle` et la plage a été modifiée de 0-360 degrés à -180 à 180 degrés. Selon votre code, vous pouvez ou non être tenu d’apporter des modifications car la plupart des fonctions mathématiques peuvent gérer l’une ou l’autre plage.

L’API v3.0 introduit également les améliorations suivantes que vous pouvez éventuellement exploiter :
* `createdDateTime` et `lastUpdatedDateTime` sont ajoutés pour vous permettre de suivre la durée du traitement. Pour plus d’informations, consultez la documentation. 
* `version` indique la version de l’API utilisée pour générer les résultats
* Ajout de `confidence` par mot . Cette valeur est étalonnée afin qu’une valeur de 0,95 signifie qu’il existe 95 % de chances que la reconnaissance soit correcte. Le score de confiance peut être utilisé pour sélectionner le texte à envoyer à des fins de révision humaine. 
    
    
Dans la version 2.X, le format de sortie est le suivant : 
    
```json
    {
        {
                "status": "Succeeded",
                "recognitionResults": [
                    {
                    "page": 1,
                    "language": "en",
                    "clockwiseOrientation": 349.59,
                    "width": 2661,
                    "height": 1901,
                    "unit": "pixel",
                    "lines": [
                        {
                        "boundingBox": [
                            67,
                            646,
                            2582,
                            713,
                            2580,
                            876,
                            67,
                            821
                        ],
                        "text": "The quick brown fox jumps",
                        "words": [
                            {
                            "boundingBox": [
                                143,
                                650,
                                435,
                                661,
                                436,
                                823,
                                144,
                                824
                            ],
                            "text": "The",
                            },
            // The rest of result is omitted for brevity 
            
}
```
    
Dans la version v3.0, il a été ajusté :
    
```json
    {
        {
            "status": "succeeded",
            "createdDateTime": "2020-05-28T05:13:21Z",
            "lastUpdatedDateTime": "2020-05-28T05:13:22Z",
            "analyzeResult": {
            "version": "3.0.0",
            "readResults": [
                {
                "page": 1,
                "language": "en",
                "angle": 0.8551,
                "width": 2661,
                "height": 1901,
                "unit": "pixel",
                "lines": [
                    {
                    "boundingBox": [
                        67,
                        646,
                        2582,
                        713,
                        2580,
                        876,
                        67,
                        821
                    ],
                    "text": "The quick brown fox jumps",
                    "words": [
                        {
                        "boundingBox": [
                            143,
                            650,
                            435,
                            661,
                            436,
                            823,
                            144,
                            824
                        ],
                        "text": "The",
                        "confidence": 0.958
                        },
        // The rest of result is omitted for brevity 
        
    }
```

## <a name="service-only"></a>Service uniquement

### `Recognize Text`
`Recognize Text` est une opération de la *préversion* qui est *dépréciée dans toutes les versions de l’API Vision par ordinateur* . Vous devez migrer de `Recognize Text` vers `Read` (v3.0) ou `Batch Read File` (v2.0, v2.1). La version v3.0 de `Read`, qui comprend des modèles plus récents et plus performants pour la reconnaissance de texte et des fonctionnalités supplémentaires, est recommandée. Pour effectuer une mise à niveau de `Recognize Text` vers `Read` :

|Reconnaître le texte 2.x |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/ **v2.0/recognizeText[?mode]**|https://{endpoint}/vision/< **chaîne de version** >/read/analyze[?language]|
    
Le paramètre _Mode_ n'est pas pris en charge dans `Read`. Les textes manuscrits et imprimés sont automatiquement pris en charge.
    
Un nouveau paramètre facultatif _Langue_ est disponible dans la version v3.0. Si vous ne connaissez pas la langue de votre document ou s’il est multilingue, ne l’incluez pas. 

### `Get Recognize Text Operation Result`

|Reconnaître le texte 2.x |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/ **v2.0/textOperations/** {operationId}|https://{endpoint}/vision/< **chaîne de version** >/read/analyzeResults/{operationId}|

### <a name="get-recognize-text-operation-result-status-flags"></a>Indicateurs d’état `Get Recognize Text Operation Result`
Lorsque l’appel de `Get Recognize Text Operation Result` aboutit, il renvoie un champ de chaîne d’état dans le corps du json. 
 
|Reconnaître le texte 2.x |Read 3.x  |
|----------|-----------|
|`"NotStarted"` |    `"notStarted"`|
|`"Running"` | `"running"`|
|`"Failed"` | `"failed"`|
|`"Succeeded"` | `"succeeded"`|

### <a name="api-response-json"></a>Réponse d’API (JSON)

Notez les modifications suivantes apportées au json :    
* Dans la version v2.x, `Get Read Operation Result` renvoie le json de reconnaissance OCR lorsque l’état est `Succeeded`. Dans v3.x, ce champ est `succeeded`.
* Pour obtenir la racine du tableau de pages, remplacez la hiérarchie json `recognitionResult` par `analyzeResult`/`readResults`. La hiérarchie json des mots et lignes par page reste inchangée et dès lors, aucune modification de code n’est requise.

L’API v3.0 présente également les améliorations suivantes. Pour plus d’informations, consultez les informations de référence sur l’API :
* `createdDateTime` et `lastUpdatedDateTime` sont ajoutés pour vous permettre de suivre la durée du traitement. Pour plus d’informations, consultez la documentation. 
* `version` indique la version de l’API utilisée pour générer les résultats
* Ajout de `confidence` par mot . Cette valeur est étalonnée afin qu’une valeur de 0,95 signifie qu’il existe 95 % de chances que la reconnaissance soit correcte. Le score de confiance peut être utilisé pour sélectionner le texte à envoyer à des fins de révision humaine. 
* `angle` orientation générale du texte dans le sens des aiguilles d’une montre, mesurée en degrés entre (-180, 180].
* `width` et `"height"` vous donnent les dimensions de votre document, et `"unit"` fournit l’unité de ces dimensions (pixels ou pouces, en fonction du type de document).
* `page` documents multipages pris en charge
* `language` langue d’entrée du document (à partir du paramètre facultatif _Langue_ ).


Dans la version 2.X, le format de sortie est le suivant : 
    
```json
    {
        {
                "status": "Succeeded",
                "recognitionResult": [
                    {
                    "lines": [
                        {
                        "boundingBox": [
                            67,
                            646,
                            2582,
                            713,
                            2580,
                            876,
                            67,
                            821
                        ],
                        "text": "The quick brown fox jumps",
                        "words": [
                            {
                            "boundingBox": [
                                143,
                                650,
                                435,
                                661,
                                436,
                                823,
                                144,
                                824
                            ],
                            "text": "The",
                            },
            // The rest of result is omitted for brevity 
            
    }
```
    
Dans v3.x, il a été ajusté :
    
```json
    {
        {
            "status": "succeeded",
            "createdDateTime": "2020-05-28T05:13:21Z",
            "lastUpdatedDateTime": "2020-05-28T05:13:22Z",
            "analyzeResult": {
            "version": "3.0.0",
            "readResults": [
                {
                "page": 1,
                "angle": 0.8551,
                "width": 2661,
                "height": 1901,
                "unit": "pixel",
                "lines": [
                    {
                    "boundingBox": [
                        67,
                        646,
                        2582,
                        713,
                        2580,
                        876,
                        67,
                        821
                    ],
                    "text": "The quick brown fox jumps",
                    "words": [
                        {
                        "boundingBox": [
                            143,
                            650,
                            435,
                            661,
                            436,
                            823,
                            144,
                            824
                        ],
                        "text": "The",
                        "confidence": 0.958
                        },
        // The rest of result is omitted for brevity 
        
    }
```

## <a name="container-only"></a>Conteneur uniquement

### `Synchronous Read`

|Read 2.0 |Read 3.x  |
|----------|-----------|
|https://{endpoint}/vision/ **v2.0/read/core/Analyze**     |https://{endpoint}/vision/< **chaîne de version** >/read/syncAnalyze[?language]|
