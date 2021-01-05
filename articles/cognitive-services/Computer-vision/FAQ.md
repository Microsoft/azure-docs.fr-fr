---
title: Questions fréquentes (FAQ) sur Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Obtenez des réponses aux questions fréquentes concernant l’API Vision par ordinateur dans Azure Cognitive Services.
services: cognitive-services
author: KellyDF
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: kefre
ms.custom: seodec18
ms.openlocfilehash: e8ab205a43e5cb1e8e2b96dbd9600e1fceb29403
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68564605"
---
# <a name="computer-vision-api-frequently-asked-questions"></a>Questions fréquentes (FAQ) sur l’API Vision par ordinateur

> [!TIP]
> Si vous ne trouvez pas les réponses à vos questions dans cette FAQ, essayez d’interroger la communauté de l’API Vision par ordinateur sur [StackOverflow](https://stackoverflow.com/questions/tagged/project-oxford+or+microsoft-cognitive) ou de contacter le [support technique sur UserVoice](https://cognitive.uservoice.com/).

---

**Question** : *Puis-je former l’API Vision par ordinateur pour qu’elle utilise des balises personnalisées ?  J’aimerais par exemple ajouter des images d’espèces de chats pour « former » l’IA, puis recevoir une valeur d’espèce en fonction d’une requête d’IA.*

**Réponse** : Cette fonction n’est pas disponible actuellement. Toutefois, nos ingénieurs travaillent pour mettre en œuvre cette fonctionnalité sur l’API Vision par ordinateur.

---

**Question** : *L’API Vision par ordinateur peut-elle être utilisée localement sans connexion Internet ?*

**Réponse** : Actuellement, nous ne proposons pas de solution locale.

---

**Question** : *L’API Vision par ordinateur peut-elle être utilisée pour lire des plaques d’immatriculation ?*

**Réponse** : L’API Vision par ordinateur offre une fonctionnalité efficace de détection de texte avec l’OCR, mais celle-ci n’est pas optimisée pour les plaques d’immatriculation pour le moment. Nous essayons en permanence d’améliorer nos services et avons ajouté l’OCR pour la détection automatique des plaques d’immatriculation à notre liste de demandes de fonctionnalités.

---

**Question** : *Quels sont les types de surfaces d’écriture pris en charge pour la reconnaissance d’écriture manuscrite ?*

**Réponse** : La technologie fonctionne avec différents types de surfaces, y compris les tableaux blancs, un livre blanc et les pense-bêtes jaunes.

---

**Question** : *Quelle est la durée de l’opération de reconnaissance d’écriture manuscrite ?*

**Réponse** : La durée de cette opération varie en fonction de la longueur du texte. Pour les textes plus longs, elle peut prendre plusieurs secondes. Par conséquent, une fois l’opération de reconnaissance d’écriture manuscrite terminée, vous devez attendre avant de récupérer les résultats à l’aide de l’opération Get Handwritten Text Operation Result.

---

**Question** : *De quelle façon la technologie de reconnaissance d’écriture manuscrite traite-t-elle le texte inséré à l’aide d’un caret au milieu d’une ligne ?*

**Réponse** : Ce type de texte est renvoyé sous la forme d’une ligne distincte par l’opération de reconnaissance d’écriture manuscrite.

---

**Question** : *Comment la technologie de reconnaissance d’écriture manuscrite gère-t-elle les mots ou les lignes barrés ?*

**Réponse** : Si les mots sont barrés avec plusieurs lignes pour les masquer complètement, l’opération de reconnaissance d’écriture manuscrite ne les prend pas en compte. Toutefois, si les mots sont barrés à l’aide d’une seule ligne, ce passage est traité comme un bruit et les mots seront tout de même pris en compte par l’opération de reconnaissance d’écriture manuscrite.

---

**Question** : *Quelles sont les orientations de texte prises en charge pour la technologie de reconnaissance d’écriture manuscrite ?*

**Réponse** : Les textes orientés de 30 à 40 degrés maximum peuvent être pris en compte par l’opération de reconnaissance d’écriture manuscrite.

---
