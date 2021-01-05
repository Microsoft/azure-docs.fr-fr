---
title: Envoyer des données manuscrites à l’API Ink Recognizer
titleSuffix: Azure Cognitive Services
description: Apprenez à appeler l’API Ink Recognizer pour différentes applications.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: aahi
ms.openlocfilehash: 9ecf3ad43b46d491d5f15270790251a0bc4e882e
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94365361"
---
# <a name="send-ink-data-to-the-ink-recognizer-api"></a>Envoyer des données manuscrites à l’API Ink Recognizer 

[!INCLUDE [ink-recognizer-deprecation](../includes/deprecation-note.md)]

Les entrées manuscrites numériques font référence aux technologies qui permettent de représenter sous forme numérique des entrées comme l’écriture manuscrite et les dessins. Ces représentations sont généralement obtenues à l’aide d’un numériseur qui capture les mouvements des appareils d’entrée, comme les stylets. Alors que les appareils permettent toujours des expériences d’écriture manuscrite numériques riches, l’intelligence artificielle et le machine learning permettent la reconnaissance de formes et de texte écrits dans n’importe quel contexte. L’API Ink Recognizer vous permet d’envoyer des traits d’encre pour obtenir des informations détaillées les concernant. 

## <a name="the-ink-recognizer-api-vs-ocr-services"></a>API Ink Recognizer et services OCR

L’API Ink Recognizer n’utilise pas la reconnaissance optique de caractères (OCR). Les services OCR traitent les données de pixel des images pour assurer la reconnaissance de l’écriture manuscrite et du texte. Ce procédé est parfois appelé reconnaissance hors connexion. En revanche, l’API Ink Recognizer a besoin de données de trait d’encre numérique capturées lors de l’utilisation de l’appareil d’entrée. Le traitement des données d’entrée manuscrite numérique de cette façon peut produire des résultats plus précis par rapport aux services OCR. 

## <a name="sending-ink-data"></a>Envoi de données d’entrée manuscrite

L’API Ink Recognizer a besoin des coordonnées X et Y qui représentent les traits d’encre créés par un appareil d’entrée, à partir du moment où il touche la surface de détection jusqu’à ce qu’il soit relevé. Les points de chaque trait doivent être une chaîne de valeurs séparées par des virgules au format JSON comme dans l’exemple ci-dessous. De plus, chaque trait d’encre doit avoir un ID unique dans chaque requête. Si l’ID est répété au sein de la même requête, l’API retourne une erreur. Pour obtenir des résultats de reconnaissance plus précis, utilisez au moins huit chiffres après la virgule décimale. L’origine (0,0) de la zone de dessin est supposée être le coin supérieur gauche de la zone d’écriture manuscrite.

> [!NOTE]
> L’exemple suivant n’est pas un format JSON valide. Vous trouverez une requête JSON Ink Recognizer complète sur [GitHub](https://go.microsoft.com/fwlink/?linkid=2089909).
 
```json
{
  "language": "en-US",
  "strokes": [
   {
    "id": 43,
    "points": 
        "5.1365, 12.3845,
        4.9534, 12.1301,
        4.8618, 12.1199,
        4.7906, 12.2217,
        4.7906, 12.5372,
        4.8211, 12.9849,
        4.9534, 13.6667,
        5.0958, 14.4503,
        5.3299, 15.2441,
        5.6555, 16.0480,
        ..."
   },
    ...
  ]
}
```

## <a name="ink-recognizer-response"></a>Réponse Ink Recognizer

L’API Ink Recognizer retourne une réponse d’analyse sur les objets qu’elle a reconnus dans le contenu de l’entrée manuscrite. La réponse contient les unités de reconnaissance qui décrivent les relations entre les différents traits d’encre. Par exemple, les traits qui créent des formes distinctes se trouvent dans des unités différentes. Chaque unité contient des informations détaillées sur ses traits d’encre, notamment l’objet reconnu, ses coordonnées et d’autres attributs de dessin.

## <a name="shapes-recognized-by-the-ink-recognizer-api"></a>Formes reconnues par l’API Ink Recognizer

L’API Ink Recognizer peut identifier les formes les plus couramment utilisées dans la prise de notes. L’image ci-dessous montre quelques exemples simples. Pour obtenir la liste complète des formes et des contenus d’entrée manuscrite reconnus par l’API, consultez l’[article sur les informations de référence de l’API](/rest/api/cognitiveservices/inkrecognizer/inkrecognizer). 

![Liste des formes reconnues par l’API Ink Recognizer](../media/shapes.png)

## <a name="recommended-calling-patterns"></a>Modèles d’appel recommandés

Vous pouvez appeler l’API REST Ink Recognizer dans différents modèles selon votre application. 

### <a name="user-initiated-api-calls"></a>Appels d’API lancés par l’utilisateur

Si vous créez une application qui accepte une entrée utilisateur (par exemple, une application de prise de notes ou d’annotation), vous pouvez avoir envie de lui donner la possibilité de contrôler à quel moment une entrée manuscrite est envoyée à l’API Ink Recognizer. Cette fonctionnalité s’avère particulièrement utile quand du texte et des formes sont à la fois présents dans la zone de dessin, alors que les utilisateurs veulent effectuer différentes actions sur eux. Envisagez d’ajouter des fonctionnalités de sélection (comme un outil lasso ou tout autre outil de sélection géométrique) qui permettent aux utilisateurs de choisir ce qui est envoyé à l’API.  

### <a name="app-initiated-api-calls"></a>Appels d’API lancés par l’application

Vous pouvez également demander à votre application d’appeler l’API Ink Recognizer après un délai d’expiration. En envoyant systématiquement les traits d’encre lors de leur tracé à l’API, vous pouvez stocker des résultats de reconnaissance au moment où ils sont créés tout en améliorant le temps de réponse de l’API. Par exemple, vous pouvez envoyer une ligne de texte manuscrit à l’API dès la détection que votre utilisateur l’a terminée. 

Avoir des résultats de reconnaissance à l’avance vous donne des informations sur les caractéristiques des traits d’encre dans leur façon d’être liés entre eux. Par exemple, des traits regroupés pour former le même mot, une ligne, une liste, un paragraphe ou une forme. Ces informations peuvent améliorer les fonctionnalités de sélection d’entrée manuscrite de votre application en étant capable de sélectionner des groupes de traits immédiatement, par exemple.

## <a name="integrate-the-ink-recognizer-api-with-windows-ink"></a>Intégrer l’API Ink Recognizer dans Windows Ink

[Windows Ink](/windows/uwp/design/input/pen-and-stylus-interactions) fournit des outils et des technologies qui permettent des expériences d’entrée manuscrite numérique sur divers appareils. Vous pouvez combiner la plateforme Windows Ink avec l’API Ink Recognizer pour créer des applications qui affichent et interprètent les traits d’encre numériques.

## <a name="next-steps"></a>Étapes suivantes

* [Qu’est-ce que l’API Ink Recognizer ?](../overview.md)
* [Informations de référence sur l’API REST Ink Recognizer](/rest/api/cognitiveservices/inkrecognizer/inkrecognizer)

* Commencer à envoyer des données de trait d’encre numérique en :
    * [C#](../quickstarts/csharp.md)
    * [Java](../quickstarts/java.md)
    * [JavaScript](../quickstarts/javascript.md)