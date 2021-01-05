---
title: Format TSV pour les tests par lot - QnA Maker
titleSuffix: Azure Cognitive Services
description: Comprendre le format TSV pour les tests par lot
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 10/24/2019
ms.openlocfilehash: 0ed948b4bf69fc672a59a7825279a12868d10521
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87132124"
---
# <a name="batch-testing-tsv-format"></a>Format TSV pour les tests par lot

Les tests par lot sont disponibles à partir du [code source](https://github.com/Azure-Samples/cognitive-services-qnamaker-csharp/tree/master/documentation-samples/batchtesting) ou en tant que [fichier exécutable compressé téléchargeable](https://aka.ms/qna_btzip). Le format de la commande permettant d'exécuter le test par lot est le suivant :

```console
batchtesting.exe input.tsv https://YOUR-HOST.azurewebsites.net ENDPOINT-KEY out.tsv
```

|Paramètre|Valeur attendue|
|--|--|
|1|nom du fichier tsv formaté avec les [champs d'entrée TSV](#tsv-input-fields)|
|2|URI du point de terminaison, avec YOUR-HOST de la page de publication du portail QnA Maker.|
|3|ENDPOINT-KEY, disponible sur la page de publication du portail QnA Maker.|
|4|nom du fichier TSV créé par le test par lot pour les résultats.|

Utilisez les informations suivantes afin de comprendre et d'implémenter le format TSV pour les tests par lot. 

## <a name="tsv-input-fields"></a>Champs d'entrée TSV

|Champs du fichier d'entrée TSV|Notes|
|--|--|
|KBID|Votre ID de base de connaissances figurant sur la page de publication.|
|Question|Question qu'un utilisateur entrerait.|
|Balises de métadonnées|facultatif|
|Meilleur paramètre|facultatif| 
|ID de la réponse attendue|facultatif|

![Format d'entrée du fichier TSV pour le test par lot.](media/batch-test/input-tsv-format-batch-test.png)

## <a name="tsv-output-fields"></a>Champs de sortie TSV 

|Paramètres du fichier de sortie TSV|Notes|
|--|--|
|KBID|Votre ID de base de connaissances figurant sur la page de publication.|
|Question|Question telle qu'elle a été saisie à partir du fichier d'entrée.|
|Réponse|Meilleure réponse de votre base de connaissances.|
|ID de la réponse|ID de la réponse|
|Score|Score de prédiction pour la réponse. |
|Balises de métadonnées|associées à la réponse renvoyée|
|ID de la réponse attendue|facultatif (uniquement lorsque l'ID de la réponse attendue est fourni)|
|Étiquette de jugement|facultatif, les valeurs peuvent être : correct ou incorrect (uniquement lorsque la réponse attendue est fournie)|
